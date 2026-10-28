# Gallery Zero Design System

Gallery Zero is an immersive exhibition builder for art and photography. The system favors quiet editorial hierarchy, deliberate pacing, and image-led composition. Reference research informed the direction only; no reference assets, copy, or layouts are reused.

## Themes

Use one theme per exhibition or section. Components consume semantic tokens rather than raw values.

### Dark minimal

```text
--color-bg: #0B0B0C
--color-surface: #151518
--color-surface-raised: #202024
--color-text: #F1EEE8
--color-text-muted: #AAA6A0
--color-accent: #D7C6A0
--color-border: #343238
--color-scrim: #0B0B0CCC
```

### Light editorial

```text
--color-bg: #F3F0EA
--color-surface: #FBFAF7
--color-surface-raised: #E8E3DA
--color-text: #171717
--color-text-muted: #67635D
--color-accent: #9B4635
--color-border: #D2CCC2
--color-scrim: #F3F0EACC
```

### High-contrast neon

```text
--color-bg: #090A0D
--color-surface: #13161B
--color-surface-raised: #1C222A
--color-text: #F5F7F2
--color-text-muted: #A7B0B7
--color-accent: #D8FF36
--color-accent-secondary: #FF4FD8
--color-border: #3A444D
--color-scrim: #090A0DE6
```

Color contrast must be checked for every theme/state combination. Accent colors are for focus, selection, progress, and small calls to action, not long-form text.

## Typography

Free Google Fonts only:

- **Display:** Space Grotesk, 500–700. Use for exhibition titles, large numerals, and compact navigation labels.
- **Text:** Manrope, 400–600. Use for descriptions, controls, metadata, and accessible UI labels.

```text
--font-display: "Space Grotesk", sans-serif
--font-text: "Manrope", sans-serif
```

### Type scale

The scale is fluid between the listed minimum and maximum values. Line heights are unitless.

| Token | Size | Line height | Use |
|---|---:|---:|---|
| `--type-display-xl` | clamp(4rem, 11vw, 10rem) | 0.9 | Landing/exhibition title |
| `--type-display-lg` | clamp(3rem, 7vw, 6rem) | 0.94 | Section title |
| `--type-display-md` | clamp(2.25rem, 4vw, 4rem) | 1 | Work title |
| `--type-heading` | clamp(1.5rem, 2.4vw, 2.25rem) | 1.1 | Panel heading |
| `--type-body-lg` | clamp(1.125rem, 1.4vw, 1.375rem) | 1.45 | Intro/curatorial copy |
| `--type-body` | 1rem | 1.5 | Default body |
| `--type-label` | 0.75rem | 1.2 | Metadata and controls |
| `--type-micro` | 0.625rem | 1.2 | Indexes and secondary metadata |

Use `letter-spacing: -0.02em` for display text and `0.04em` for uppercase labels.

## Spacing and layout

All spacing uses an 8px base scale:

```text
--space-1: 8px
--space-2: 16px
--space-3: 24px
--space-4: 32px
--space-5: 40px
--space-6: 48px
--space-7: 56px
--space-8: 64px
--space-10: 80px
--space-12: 96px
--space-16: 128px
--space-20: 160px
--space-24: 192px
```

Use a 12-column grid:

```text
--grid-columns: 12
--grid-gutter: clamp(16px, 2vw, 32px)
--grid-margin: clamp(16px, 5vw, 80px)
```

Desktop layouts use all 12 columns. Tablet layouts collapse to 8 columns and mobile layouts to 4 columns while retaining the same semantic order. Artwork may break the grid intentionally, but captions, controls, and reading measure remain aligned to it. Keep long-form copy between 4 and 6 columns.

## Motion tokens

Motion should feel weighted and cinematic rather than busy.

```text
--ease-standard: cubic-bezier(0.22, 1, 0.36, 1)
--ease-enter: cubic-bezier(0.16, 1, 0.3, 1)
--ease-exit: cubic-bezier(0.7, 0, 0.84, 0)
--ease-linear: linear

--duration-instant: 120ms
--duration-fast: 240ms
--duration-standard: 480ms
--duration-slow: 800ms
--duration-cinematic: 1200ms
```

Prefer opacity, transform, clip-path, and mask transitions. Avoid animating layout properties. Stagger related items by `--duration-instant` and cap a sequence at five items.

## Cursor behavior

The default cursor remains available and functional. On pointer devices, a small theme-colored dot may expand to a labeled ring over interactive artwork, navigation, and draggable studio controls. The label states the action (`View`, `Open`, or `Drag`) and never replaces an accessible text label. Disable custom cursor rendering for touch, coarse pointers, and reduced motion.

## Scroll behavior

Use Lenis for smooth interpolation and GSAP ScrollTrigger for section choreography:

1. Lenis owns the virtual scroll loop and is synchronized with GSAP's ticker.
2. ScrollTrigger owns pinning, scrubbed artwork scale/opacity, caption reveals, and progress indicators.
3. Keep pinned sections bounded to one viewport-height sequence where possible.
4. Use scrubbed movement for spatial continuity, not decorative parallax on every element.
5. Keep keyboard focus order independent of visual pinning and expose a skip-to-content control.
6. On mobile, reduce pinned complexity and preserve native-feeling momentum.

## Reduced motion

When `prefers-reduced-motion: reduce` is enabled, stop Lenis interpolation, remove ScrollTrigger scrubbing and pinning, disable cursor morphing, and replace staged transitions with immediate state changes or a single short opacity fade. Artwork, captions, controls, and all content must remain available in the same logical order.
