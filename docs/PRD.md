# Gallery Zero: PRD and Build Plan

**Course:** Cloud Computing (SE-490) Complex Engineering Activity **Document type:** Product Requirements Document + Delivery Plan + AI-Assisted Build Strategy **Status:** Draft v1

> **Note on project choice.** You did not pick between the four ideas yet, so this PRD uses **Gallery Zero**, the Awwwards-style virtual exhibition builder. It is the safest pick for a motion-heavy frontend with a simple backend. If you prefer Sonic Atlas, sections 3 to 5 change (pins instead of exhibits, audio instead of images); sections 6 to 14 stay almost identical.

---

## 1. Product Overview

### 1.1 Vision

Gallery Zero lets artists, photographers, and designers turn their work into a cinematic, scroll-driven online exhibition with its own shareable URL. Visitors walk through the exhibition like a gallery: smooth scroll, parallax, reveal animations, and a custom cursor, instead of a flat grid.

### 1.2 Problem

Portfolio platforms are either template-flat (Behance, Instagram) or require front-end skills to look distinctive (hand-built Awwwards-style sites). Creators without developer skills cannot publish an immersive showcase.

### 1.3 Goals

1. Deliver a visually exceptional public frontend that works as a portfolio piece.
2. Provide a real, complete full-stack product: auth, CRUD, uploads, search, analytics.
3. Satisfy every CEA requirement (VPC, containers, registry, CI/CD, ALB, ASG, security, NFR justification).
4. Implement at least one enhancement (this plan implements three: S3, Terraform, CloudWatch).

### 1.4 Non-Goals (to protect scope and credits)

- No payments or e-commerce.
- No real-time collaboration or comments.
- No native mobile app (responsive web only).
- No ML recommendation features.

### 1.5 Success Metrics

| Metric | Target |
| --- | --- |
| Lighthouse Performance (public exhibit page, mobile) | 80 or higher |
| Lighthouse Accessibility | 90 or higher |
| Pipeline: push to live | under 10 minutes, zero manual steps |
| API p95 latency (list/read endpoints) | under 300 ms at 50 concurrent users |
| Layout correctness | no horizontal scroll or broken layout at 375, 768, 1280, 1920 px |

---

## 2. Users and Personas

| Persona | Description | Key needs |
| --- | --- | --- |
| **Creator** | Artist/photographer with 10 to 60 works | Upload, arrange, publish, share link, see views |
| **Visitor** | Anyone with a link, no account | Fast, beautiful viewing experience on any device |
| **Admin** (optional) | Course demo / moderation | Feature exhibitions, remove content |

---

## 3. Functional Requirements

Priority: **P0** = must ship for CEA demo, **P1** = should ship, **P2** = nice to have.

### 3.1 Authentication and Accounts

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-A1 | Register with email and password; passwords hashed with Argon2 or bcrypt | P0 |
| FR-A2 | Login returns short-lived JWT access token plus refresh token (httpOnly cookie) | P0 |
| FR-A3 | Profile: display name, bio, avatar, social link | P1 |
| FR-A4 | Rate limiting on login and register endpoints | P1 |

### 3.2 Exhibitions

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-E1 | Create, read, update, delete an exhibition (title, subtitle, slug, theme, cover) | P0 |
| FR-E2 | Each exhibition has a unique public URL: `/x/{slug}` | P0 |
| FR-E3 | Draft and published states; only published are public | P0 |
| FR-E4 | Theme presets (3 minimum): Monolith (dark, minimal), Paper (light, editorial), Neon (high contrast) | P1 |
| FR-E5 | Tags and category per exhibition | P1 |

### 3.3 Artworks (Items)

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-W1 | Upload image (JPEG, PNG, WebP, max 10 MB) direct to S3 via presigned URL | P0 |
| FR-W2 | Backend generates thumbnail and medium sizes (WebP) after upload | P1 |
| FR-W3 | Artwork metadata: title, year, medium, description, alt text | P0 |
| FR-W4 | Reorder artworks within an exhibition (drag and drop; persisted as `position`) | P0 |
| FR-W5 | Layout hint per artwork: full-bleed, split, pair, caption-only | P1 |

### 3.4 Discovery and Viewing

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-D1 | Home page shows featured and latest published exhibitions | P0 |
| FR-D2 | Browse page with filter by tag/category and text search | P1 |
| FR-D3 | Public exhibition viewer with scroll-driven motion (see section 6) | P0 |
| FR-D4 | View counter (unique per day per visitor hash, no PII stored) | P1 |
| FR-D5 | Open Graph image and meta tags per exhibition for link previews | P1 |

### 3.5 Operations

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-O1 | `GET /health` (liveness) and `GET /ready` (checks DB) for ALB health checks | P0 |
| FR-O2 | Structured JSON logging with request ID | P0 |
| FR-O3 | `GET /metrics` Prometheus format (optional if CloudWatch only) | P2 |

---

## 4. Non-Functional Requirements (mapped to CEA item 10)

| NFR | Design decision that addresses it | Known limitation |
| --- | --- | --- |
| **Scalability** | Stateless API and web containers; sessions via JWT, files in S3, state in RDS; ALB distributes load; ASG adds instances | RDS is a single writer; scaling DB writes needs read replicas or Aurora |
| **Availability** | Two AZs; ALB in public subnets across AZs; ASG min 2 instances spread across AZs; RDS Multi-AZ failover | NAT gateway in one AZ is a SPOF unless one per AZ (cost trade-off, documented) |
| **Elasticity** | ASG target-tracking policy on average CPU (target 60%); scale in on low demand | Cold start time of new instance (about 2 to 3 min) |
| **Security** | Tiered security groups, private subnets for API and DB, Secrets Manager, OIDC for CI, least-privilege IAM, HTTPS at ALB | No WAF in base design; listed as improvement |
| **Maintainability / Deployability** | Dockerfiles, docker-compose for local, GitHub Actions pipeline, Terraform for all infra | Rolling deploy only; no blue/green |
| **Performance / Latency** | Image resizing to WebP, lazy loading, CloudFront in front of S3, Cache-Control headers, DB indexes, ALB close to users | Heavy animation JS costs mobile performance; mitigated by code-splitting and reduced-motion mode |
| **Cost-Efficiency** | t3.micro/t3.small instances, db.t3.micro, one NAT gateway, S3 lifecycle rules, scale-in schedule | Single NAT lowers cost but reduces AZ resilience |
| **Portability** | Everything runs in Docker; only S3 and RDS are provider-specific, both behind env-configured adapters (S3-compatible API works with MinIO locally) | Terraform code is AWS-specific |

The report must state these limitations honestly; graders reward that.

---

## 5. Data Model (PostgreSQL)

### 5.1 Tables

```sql
users(
  id UUID PK, email CITEXT UNIQUE NOT NULL, password_hash TEXT NOT NULL,
  display_name TEXT NOT NULL, bio TEXT, avatar_key TEXT,
  role TEXT CHECK (role IN ('creator','admin')) DEFAULT 'creator',
  created_at TIMESTAMPTZ DEFAULT now()
)

exhibitions(
  id UUID PK, owner_id UUID FK users ON DELETE CASCADE,
  slug TEXT UNIQUE NOT NULL, title TEXT NOT NULL, subtitle TEXT,
  theme TEXT DEFAULT 'monolith', cover_artwork_id UUID NULL,
  status TEXT CHECK (status IN ('draft','published')) DEFAULT 'draft',
  featured BOOLEAN DEFAULT false,
  published_at TIMESTAMPTZ, created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
)

artworks(
  id UUID PK, exhibition_id UUID FK exhibitions ON DELETE CASCADE,
  title TEXT NOT NULL, year INT, medium TEXT, description TEXT, alt_text TEXT,
  original_key TEXT NOT NULL, medium_key TEXT, thumb_key TEXT,
  width INT, height INT, dominant_color TEXT,
  layout TEXT DEFAULT 'full', position INT NOT NULL,
  UNIQUE (exhibition_id, position) DEFERRABLE INITIALLY DEFERRED
)

tags(id SERIAL PK, name TEXT UNIQUE NOT NULL)
exhibition_tags(exhibition_id UUID FK, tag_id INT FK, PRIMARY KEY (exhibition_id, tag_id))

view_events(
  id BIGSERIAL PK, exhibition_id UUID FK, visitor_hash TEXT,
  day DATE, UNIQUE (exhibition_id, visitor_hash, day)
)
```

### 5.2 Indexes

- `exhibitions(status, published_at DESC)` for home and browse
- `exhibitions(slug)` unique for public lookup
- `artworks(exhibition_id, position)` for ordered fetch
- GIN index on `to_tsvector(title || subtitle)` for search

### 5.3 Migrations

Alembic migrations only. No manual schema edits. Seed script creates one demo creator and one demo exhibition with 8 artworks for screenshots.

---

## 6. Frontend Specification (the portfolio centerpiece)

### 6.1 Stack

- **Next.js (App Router) + TypeScript**, output `standalone` for a small Docker image
- **Tailwind CSS** with design tokens from `design.md`
- **GSAP + ScrollTrigger** for scroll choreography; **Lenis** for smooth scroll
- **Framer Motion** for UI micro-interactions and page transitions
- **Three.js / React Three Fiber**: optional, only for the landing hero (one lightweight shader or particle field), lazy-loaded
- **TanStack Query** for API data; **Zod** for form validation

### 6.2 Pages

| Route | Purpose | Motion highlights |
| --- | --- | --- |
| `/` | Landing, featured exhibitions | Hero with kinetic typography, marquee of exhibition titles, magnetic CTA, cursor follower |
| `/browse` | Filter and search | Staggered card reveal, hover image distortion or scale, animated filter chips |
| `/x/[slug]` | **Public exhibition viewer** | Full-screen intro, pinned horizontal scroll sections, parallax images, caption reveals, progress indicator, theme transitions |
| `/login`, `/register` | Auth | Minimal form, animated validation |
| `/studio` | Creator dashboard | List exhibitions, stats |
| `/studio/[id]` | Exhibition editor | Drag-drop upload, drag reorder, live theme preview, publish toggle |

### 6.3 Design Principles (to be locked in `design.md` before coding)

1. One display typeface plus one text typeface; large type scale with strong contrast.
2. Restrained palette per theme (background, foreground, one accent).
3. Generous whitespace, 12-column grid, consistent 8 px spacing scale.
4. Motion has purpose: ease curves and durations defined once as tokens (for example 0.6 to 1.0 s, custom cubic-bezier), not ad hoc.
5. Every animation respects `prefers-reduced-motion`.
6. Custom cursor and magnetic effects are desktop only; touch devices get simpler interactions.

### 6.4 Frontend Performance Budget

- Initial JS for the landing page under 250 KB gzipped (excluding lazy-loaded 3D)
- All images via `next/image` or pre-generated WebP, with `loading=lazy` below the fold and blur-up placeholder from `dominant_color`
- Fonts self-hosted, `font-display: swap`, subsetted
- Three.js loaded with dynamic import, disabled on low-power or reduced-motion devices

### 6.5 Accessibility

Semantic HTML, keyboard-navigable viewer (arrow keys step through artworks), visible focus states, alt text required at upload, color contrast AA.

---

## 7. Backend Specification

### 7.1 Stack

FastAPI, SQLAlchemy 2.x, Alembic, Pydantic v2, asyncpg, boto3, Pillow (image resize), `python-jose` or `pyjwt`, `argon2-cffi`, pytest + httpx.

### 7.2 API (REST, prefix `/api/v1`)

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| POST | `/auth/register` | none | Create account |
| POST | `/auth/login` | none | Return access token, set refresh cookie |
| POST | `/auth/refresh` | cookie | New access token |
| GET | `/me` | JWT | Current user |
| PATCH | `/me` | JWT | Update profile |
| GET | `/exhibitions` | none | Published list; query: `tag`, `q`, `page`, `featured` |
| GET | `/exhibitions/{slug}` | none | Public exhibition with ordered artworks |
| GET | `/studio/exhibitions` | JWT | Owner's exhibitions |
| POST | `/studio/exhibitions` | JWT | Create |
| PATCH | `/studio/exhibitions/{id}` | JWT owner | Update (including publish) |
| DELETE | `/studio/exhibitions/{id}` | JWT owner | Delete |
| POST | `/studio/exhibitions/{id}/uploads` | JWT owner | Returns presigned S3 PUT URL and key |
| POST | `/studio/exhibitions/{id}/artworks` | JWT owner | Register uploaded artwork, triggers resize |
| PATCH | `/studio/artworks/{id}` | JWT owner | Edit metadata |
| PUT | `/studio/exhibitions/{id}/order` | JWT owner | Reorder (array of ids) |
| DELETE | `/studio/artworks/{id}` | JWT owner | Delete (and S3 objects) |
| POST | `/exhibitions/{slug}/view` | none | Record unique view |
| GET | `/health`, `/ready` | none | ALB checks |

### 7.3 Backend Rules

- Layered structure: `routers/`, `services/`, `repositories/`, `schemas/`, `models/`.
- Ownership checks in the service layer, never trust IDs from the client.
- All config via environment variables (12-factor). Database credentials injected at runtime from AWS Secrets Manager.
- Consistent error format: `{ "error": { "code": "...", "message": "..." } }`.
- CORS restricted to the web tier origin.
- Image processing runs as a FastAPI background task (acceptable for scope; note SQS + worker as an improvement).
- Minimum 70% test coverage on services; integration tests run against a Postgres service container in CI.

---

## 8. Cloud Architecture (AWS) and CEA Mapping

### 8.1 Topology

```
Internet
   |
[CloudFront] --> S3 (artwork images, private bucket + OAC)
   |
[Route 53 / ALB DNS]
   |
[ALB: public subnets, AZ-a + AZ-b, HTTPS 443]
   |  path routing: /api/* -> API target group, /* -> Web target group
   |
[Web tier: Next.js container, ASG, private or public subnet]
[App tier: FastAPI container, ASG, PRIVATE subnets AZ-a + AZ-b]
   |
[RDS PostgreSQL Multi-AZ, PRIVATE DB subnets]

NAT Gateway (public subnet) for private outbound (ECR pulls, Secrets Manager)
```

CI/CD: GitHub Actions to build images, push to ECR, then trigger rolling update through ASG instance refresh (or SSM Run Command).

### 8.2 Requirement Mapping

| CEA item | Implementation |
| --- | --- |
| 1. Network | VPC `10.0.0.0/16`; public subnets `10.0.1.0/24`, `10.0.2.0/24`; private app `10.0.11.0/24`, `10.0.12.0/24`; private db `10.0.21.0/24`, `10.0.22.0/24`; IGW; public route table to IGW; private route table to NAT |
| 2. Web tier | Next.js standalone container behind ALB |
| 3. App tier | FastAPI container in private subnets; docker-compose.yml for local (web, api, postgres, minio) |
| 4. Database | RDS PostgreSQL, not publicly accessible, SG allows 5432 only from app SG |
| 5. Registry | Amazon ECR with two repos (`gz-web`, `gz-api`); image scan on push |
| 6. CI/CD | GitHub Actions: test, build, push, deploy |
| 7. Load balancing | ALB, two target groups, health checks `/health`, ASG per tier (min 2, max 4) |
| 8. Security | See section 9 |
| 9. Enhancements | **S3** (artwork storage), **Terraform** (all infra), **CloudWatch** (logs, alarms, dashboard); optionally cost estimate with AWS Pricing Calculator and 2 optimizations |

### 8.3 Cost Notes (for the report)

Estimate with the AWS Pricing Calculator. Likely largest items: NAT gateway, ALB, RDS Multi-AZ. Two optimization measures to propose: (1) single NAT gateway plus scheduled scale-down of ASG to min 1 outside demo hours; (2) S3 lifecycle rule moving originals to Infrequent Access and CloudFront caching to cut egress. Also destroy the stack via `terraform destroy` after grading to avoid charges. Check current pricing and free-tier status yourself, since both change.

---

## 9. Security Design

| Control | Detail |
| --- | --- |
| **Security groups** | `alb-sg`: 80/443 from 0.0.0.0/0. `web-sg`: app port from `alb-sg` only. `api-sg`: 8000 from `alb-sg` (or `web-sg`) only. `db-sg`: 5432 from `api-sg` only. No SSH; use SSM Session Manager. |
| **Segmentation** | DB and API in private subnets with no route to the IGW. |
| **Least privilege** | EC2 instance role: read specific Secrets Manager secret, pull from ECR, read/write only the artwork S3 bucket prefix. |
| **Secrets** | DB password generated by Terraform into Secrets Manager; app reads at start. Nothing in the repo or image. |
| **CI/CD secrets** | GitHub OIDC federation to an AWS role (no long-lived access keys); repository secrets only for non-AWS values. |
| **Transport** | HTTPS at ALB with ACM certificate (a domain is needed; otherwise document HTTP-only as a limitation). |
| **App security** | Argon2 hashing, JWT expiry, rate limit, input validation, upload type and size validation, presigned URLs with short expiry. |
| **Supply chain** | Non-root container user, slim base images, ECR scan on push, Dependabot enabled. |

---

## 10. CI/CD Pipeline Design

**Trigger:** push to `main`; pull requests run tests only.

| Stage | Actions |
| --- | --- |
| 1. Test | Lint (ruff, eslint), unit tests, integration tests with Postgres service container |
| 2. Build | `docker build` for web and api with Buildx cache |
| 3. Push | Authenticate to AWS via OIDC, push to ECR tagged with git SHA and `latest` |
| 4. Deploy | Update launch template image tag and start ASG instance refresh (min healthy 50%), or SSM command to pull and restart |
| 5. Verify | Smoke test: `curl` the ALB `/health` and `/api/v1/exhibitions`; fail the job on non-200 |

Evidence to capture: green workflow run, ECR image list with SHA tags, ASG activity history showing the refresh.

---

## 11. Design and Build Workflow Using the Video's Tools

The reference video's core point: switching models alone leaves sites looking AI-generated; **real references, a locked style guide, and automated visual checks** are what make the difference. The workflow below applies that to GitHub Copilot.

| Video tool | Role in this project | Phase |
| --- | --- | --- |
| **Awwwards MCP** | Agent searches Awwwards for real award-winning sites, pulls screenshots and design details, studies motion before building | Design research |
| **Dembrandt** | Extracts colors, type, spacing from chosen reference sites into `design.md`; re-scan after edits to catch accidental drift | Design system |
| **Genjutsu skills (Cast, Paint, Bunshin)** | Patterns for changing one section, planning a full page, and running parallel build plus review agents | Build structure (adapted) |
| **Responsively App (MCP)** | Screenshots multiple viewport sizes so the agent can find and fix layout breaks | Responsive QA |
| **App Store Screenshots / Brag** | Optional: polished portfolio visuals and a 20-second launch video for your README and LinkedIn | Post-build marketing |

### 11.1 Honest compatibility notes

- I am working from the video's description, not the repos themselves. **Open each repo and read its README** to confirm install steps, since the Awwwards MCP link in your paste is truncated.
- Genjutsu's three skills are written as **Claude skills**. Copilot does not load them natively, but you can copy their instructions into `.github/prompts/` files or `copilot-instructions.md` and reuse the same patterns.
- Responsively App is a desktop app (the video shows a Mac). If you are on Windows or Linux, **use Playwright MCP** instead: it can open your dev server, resize the viewport, and take screenshots at 375, 768, 1280, and 1920 px.
- The video ran agents in Cline with open-weight models; the repos are agent-agnostic and work with any agent that supports MCP tools, including Copilot agent mode in VS Code.

### 11.2 Design Phase Steps (cheap, high impact)

1. Choose 5 to 8 Awwwards references with strong exhibition, gallery, or editorial feel.
2. Use Awwwards MCP to capture screenshots and design notes for each; save to `/design/references/`.
3. Run Dembrandt on the 2 or 3 best references; merge results into one `design.md` containing: color tokens per theme, type scale, spacing, grid, border radius, easing curves, durations, cursor behavior, scroll behavior.
4. Write `design/motion-notes.md`: for each page, the 3 to 5 signature interactions in plain language.
5. **Commit these files before any UI code.** Every later prompt references them.

---

## 12. How GitHub Copilot Will Build This

### 12.1 Setup

1. VS Code with GitHub Copilot, using **Agent mode**.
2. Create `.vscode/mcp.json` registering: Awwwards MCP, Playwright MCP (or Responsively MCP), and the GitHub MCP server. Follow each repo's README for exact command and args.
3. Create `.github/copilot-instructions.md` (always-on rules, see 12.2).
4. Create reusable prompt files in `.github/prompts/`: `new-endpoint.prompt.md`, `new-section.prompt.md`, `responsive-check.prompt.md`, `write-tests.prompt.md`.
5. Keep a `docs/` folder with this PRD, `design.md`, schema, and API spec so the agent always has ground truth.

### 12.2 Content of `copilot-instructions.md` (starter)

```
Project: Gallery Zero. Read docs/PRD.md and design/design.md before any task.
Backend: FastAPI, SQLAlchemy 2, Alembic, layered architecture. Never put logic in routers.
Frontend: Next.js App Router, TypeScript strict, Tailwind using tokens from design.md only.
Never hardcode colors, spacing, easing, or secrets. Never commit .env files.
Every animation must respect prefers-reduced-motion.
Every new endpoint needs a Pydantic schema, ownership check, and a test.
After UI changes, take screenshots at 375/768/1280/1920 and fix any overflow.
Work in small steps; show a plan first for tasks touching more than 3 files.
```

### 12.3 Phased Prompt Sequence

**Phase 0: Scaffold (cheap)**

- "Create the monorepo structure: `/web`, `/api`, `/infra`, `/docs`, `/design`, root `docker-compose.yml` with web, api, postgres, minio."

**Phase 1: Database**

- "Using docs/PRD.md section 5, create SQLAlchemy models and the first Alembic migration. Add indexes listed. Create a seed script."

**Phase 2: Backend, one module per prompt**

1. Auth module (register, login, refresh, `/me`) with tests.
2. Exhibitions module (public and studio routes) with ownership checks and tests.
3. Uploads and artworks (presigned URL, register, resize background task, reorder) with tests using MinIO locally.
4. Views, search, health endpoints, structured logging.
5. Dockerfile (multi-stage, non-root user, healthcheck).

**Phase 3: Frontend foundation**

- "Set up Next.js with Tailwind tokens generated from design/design.md, fonts, Lenis smooth scroll provider, GSAP registration, and a typed API client."

**Phase 4: Frontend, one section per prompt (this is where most credits go)** For each section (landing hero, marquee, exhibition viewer intro, horizontal pinned gallery, artwork detail overlay, browse grid, studio editor):

1. Reference the specific entry in `motion-notes.md`.
2. Ask the agent to implement it.
3. Run `responsive-check` prompt (Playwright or Responsively MCP screenshots at 4 widths).
4. Ask the agent to fix only the issues it found.
5. Optional: re-run Dembrandt scan on your own site to compare against `design.md`.

**Phase 5: Integration and polish**

- Connect frontend to real API, loading and error states, empty states, 404 page, OG images, Lighthouse pass, bundle analysis.

**Phase 6: Infra and CI/CD**

- Terraform modules: network, security groups, ALB, ASG and launch templates, RDS, S3 and CloudFront, ECR, IAM, Secrets Manager, CloudWatch. Then `.github/workflows/deploy.yml`. (You can draft these with me instead of Copilot to save credits.)

### 12.4 Credit-Saving Rules

- Spend about 60% of credits on Phase 4, 25% on Phases 2 and 3, 15% on everything else.
- Prefer prompts that name exact files and PRD sections; vague prompts cause rework.
- Do not ask the agent to "make it beautiful." Ask it to implement a specific interaction from `motion-notes.md` with tokens from `design.md`.
- Use the free chat model or me for Terraform, workflow YAML, and the report.
- Commit after every working step so you can revert bad generations.

### 12.5 Quality Gates (review everything Copilot writes)

- You must be able to explain every file in the viva. Read generated code; do not paste blindly.
- Run tests and `docker compose up` after each phase.
- Check that no secrets or `.env` files were committed.
- Verify auth and ownership logic manually: try accessing another user's exhibition with a valid token.

---

## 13. Delivery Timeline (suggested 4 weeks)

| Week | Focus | Exit criteria |
| --- | --- | --- |
| 1 | Design research, `design.md`, repo scaffold, DB, auth, exhibitions API | `docker compose up` runs API with tests passing |
| 2 | Uploads, artworks, remaining API, Dockerfiles; frontend foundation and landing page | Upload to MinIO works; landing page matches design |
| 3 | Exhibition viewer, browse, studio editor; Terraform network, RDS, ALB, ASG | App reachable via ALB in AWS |
| 4 | CI/CD, CloudWatch, S3 and CloudFront, polish, Lighthouse, screenshots, report | All CEA deliverables complete |

Build the cloud infra early (week 3) rather than last, since networking and IAM issues take longer than expected.

---

## 14. Deliverables Checklist (from the CEA)

- [ ] Architecture diagram (network, services, CI/CD flow, security boundaries)
- [ ] Screenshots: VPC and subnets, EC2/ASG instances, security groups, ALB and target groups, RDS, Docker build and ECR listing, successful pipeline run, app in browser
- [ ] Source: Dockerfiles, docker-compose.yml, deploy.yml, Terraform, repo link
- [ ] Technical report (4 to 6 pages): provider justification, architecture, services per tier, containers and CI/CD, security, test results, **NFR section with limitations**

---

## 15. Risks and Mitigations

| Risk | Likelihood | Mitigation |
| --- | --- | --- |
| Copilot credits run out | Medium | Credit budget in 12.4; use free tools for infra and docs |
| Generic AI-looking UI | High without references | Mandatory design phase, `design.md`, per-interaction prompts |
| Heavy animation hurts performance | High | Budget in 6.4, lazy loading, reduced-motion path; document trade-off in report |
| AWS costs overrun | Medium | Budget alert, single NAT, destroy after grading |
| Infra debugging eats time | High | Start infra in week 3, keep Terraform modular |
| MCP tooling does not work with Copilot as in the video | Medium | Fallback to Playwright MCP and manual reference gathering |
| Student cannot explain generated code | Medium | Review gate in 12.5 |

---

## 16. Acceptance Criteria (Definition of Done)

1. A new user can register, create an exhibition, upload at least 5 images, reorder them, and publish.
2. The public URL `/x/{slug}` loads through the ALB and plays the full scroll experience on desktop and mobile.
3. Database is unreachable from the internet (verified by failed connection attempt from outside and screenshot of security group).
4. A commit to `main` results in a new running version with no manual steps, verified by the pipeline log.
5. Terminating one instance does not take the site down (ALB and ASG replace it).
6. A load test (for example `hey` or k6) triggers a scale-out event visible in ASG activity history.
7. Report contains the NFR table with honest limitations.

---

## 17. Immediate Next Steps

1. Confirm Gallery Zero (or switch to Sonic Atlas).
2. Read the READMEs of the Awwwards MCP and Dembrandt repos and install them.
3. Create the repo and commit this PRD to `/docs`.
4. Do the design phase (section 11.2) before writing any UI code.

From here I can write the Terraform modules, the `deploy.yml`, the full `copilot-instructions.md`, and the report skeleton while you spend Copilot credits on the frontend.
