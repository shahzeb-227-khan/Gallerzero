# Gallery Zero Motion Notes

These are original interaction directions for Gallery Zero. The Awwwards references are cited as research prompts, not implementation templates. No reference assets or text are used.

## Landing

- **Breathing title (assumed):** The title enters as a single quiet reveal while a thin exhibition index settles into place. Inspired by the restrained, title-led pacing associated with **A24**.
- **Scroll-to-discover (assumed):** The first scroll gently transitions from the title into a full-bleed image, with the image moving less than the page so it feels discovered rather than pushed. AGNESTOTH's live motion capture timed out, so its macro-detail/editorial motion is not measured.
- **Theme preview (measured: OFFFForum uses Lenis; 0 triggers, 0 scrubs, 0 pins):** Hovering or focusing a theme changes the page treatment through a short color and grain transition; the content stays put so the theme choice feels intentional. The transition details remain assumed.
- **Progressive invitation (measured: Maria Vasilyeva has 5 CSS keyframe animations, 12 hover rules, and easing `cubic-bezier(0.16, 1, 0.36, 1)`):** A small progress mark grows as the user approaches the first exhibition; it becomes a clear keyboard-focusable link at the end of the opening sequence. The progress-marker behavior remains assumed.

## Browse

- **Editorial filter shift (assumed):** Selecting a medium or theme crossfades the work grid while preserving the scroll anchor and filter focus. Inspired by **G. Colombel Portfolio 2026**'s portfolio-led navigation direction; its capture found no measured easing or scroll triggers.
- **Image hover crop (measured: Maria Vasilyeva has 12 hover rules and 1 sampled hover candidate):** A hovered work subtly changes its crop position and reveals title/artist metadata; the transition is reversible and does not alter the source image. The exact changed properties remain assumed.
- **Column drift (measured: OFFFForum uses Lenis; 0 triggers, 0 scrubs, 0 pins):** Featured works enter on alternating grid columns with a restrained vertical offset that resolves when they reach the reading line. The drift choreography remains assumed.
- **Load-more continuity (assumed):** New works append below the current viewport and the browse index updates without a full-page jump. A24 has 1 measured trigger but no scrub or pin, which does not establish load-more behavior.

## Exhibition viewer

- **Pinned work sequence (assumed; measured A24 pin count: 0):** The current work remains anchored while captions, edition metadata, and the next work pass through a shared 12-column frame. A24's capture does not support a pinned implementation.
- **Scrubbed image depth (assumed; measured A24 scrub count: 0, AGNESTOTH unavailable):** Scroll adjusts a small, bounded scale and focal-point shift on the artwork; the effect stops at the section boundary rather than continuing indefinitely.
- **Caption reveal (measured: A24 has 1 non-scrubbed, non-pinned ScrollTrigger; easing `power3.out`, duration median 500ms):** The caption mask opens from the reading edge after the artwork reaches its trigger line, then closes before the next work takes focus. The mask direction remains assumed.
- **Exhibition index (measured: A24 has 1 trigger and Maria Vasilyeva has 5 CSS keyframe animations):** A vertical index fills as works are viewed; selecting an item jumps to it and updates the accessible current-position label. The index fill and jump behavior remain assumed.
- **Quiet transition (measured: G. Colombel uses Three/WebGL but reports 0 triggers, 0 scrubs, and 0 pins):** Moving between works uses a short crossfade with a directional title shift, while the artwork itself changes only after the next image is ready. The transition choreography remains assumed.

## Studio

- **Canvas placement (assumed):** Dragging an artwork shows a low-contrast grid and snaps to column boundaries; release resolves with a short spring-like settle. Inspired by the spatial experimentation associated with **OFFFForum**.
- **Timeline scrubber (assumed; measured A24 and Maria scroll scrub counts: 0):** A horizontal timeline previews section timing and updates the selected scene without changing the saved exhibition order.
- **Theme switcher (assumed):** Switching between dark minimal, light editorial, and high-contrast neon updates the preview through a coordinated token transition, with no layout jump. The three themes are a Gallery Zero requirement, not a reference-site measurement.
- **Preview mode (assumed):** Entering preview dims studio chrome and expands the canvas; exiting returns to the same selected element and scroll position. Inspired by the immersive portfolio presentation associated with **G. Colombel Portfolio 2026**.
- **Validation feedback (assumed):** Missing alt text, title, or focal point appears inline beside the affected work and uses a restrained opacity/outline transition rather than a modal interruption. This is an original Gallery Zero usability pattern.

## Measured reference evidence

- **A24:** GSAP, ScrollTrigger, Lenis, Three/WebGL; 1 trigger, 0 scrubs, 0 pins; easing `power3.out` and `function(e){return 1-(1-e)**n}`; duration p25 300ms, median 500ms, p75 650ms; 13 hover rules and no sampled hover candidate.
- **OFFFForum:** Lenis; 0 triggers, 0 scrubs, 0 pins; no easing or duration vocabulary detected.
- **AGNESTOTH:** Motion capture unavailable because live navigation timed out; no motion values are asserted.
- **Maria Vasilyeva Portfolio:** WebGL; 5 CSS keyframe animations, 0 scroll triggers/scrubs/pins; easing `cubic-bezier(0.16, 1, 0.36, 1)`; 3.5s duration vocabulary; 12 hover rules and 1 sampled hover candidate.
- **G. Colombel Portfolio 2026:** Three/WebGL; 0 triggers, 0 scrubs, 0 pins; no easing or duration vocabulary detected.
