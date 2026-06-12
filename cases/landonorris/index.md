# landonorris.com — Technical Case Study

**URL:** https://landonorris.com
**Agency:** OFF+BRAND
**CMS:** Webflow (all markup uses `data-wf-*` attributes)
**Date Analyzed:** 2026-06-02

---

## Technology Stack

| Technology | Usage |
|---|---|
| **Webflow** | Core CMS and hosting (`cdn.prod.website-files.com`) |
| **Rive** | Primary vector animation engine — canvas-rendered, state-machine driven |
| **GSAP 3.13.0** | ScrollTrigger, MotionPathPlugin, SplitText 3.13.0 (bundled in single JS) |
| **Lenis** | Smooth scrolling (`css-lenis` embed, `.lenis-smooth`, `.lenis-stopped`) |
| **CSS Mask (`mask-image`)** | Complex organic shapes (footer silhouette, helmet outlines, stats boxes, case study cards) |
| **Custom JS (OFF+BRAND)** | Rive-ScrollTrigger bridge, marquee system, nav theming, scroll indicator button rive hover, video streams, page transitions |

**Notable absence:** No React, Vue, Three.js, R3F, or WebGL (other than Rive Canvas). Pure server-rendered Webflow + vanilla JS.

**JS Bundle:** `lando.OFF+BRAND.gold-android-fix-03.js` (~450KB, from `lando.itsoffbrand.io/dev-js/`). Bundles GSAP 3.13.0 (EasePack, CSSPlugin, ScrollTrigger, MotionPathPlugin, SplitText 3.13.0) + custom Rive integration functions.

---

## Rive Animation System (Core Engine)

Every interactive Rive canvas follows this attribute pattern:

```html
<canvas data-rive-object
        data-rive-file="<filename>"
        data-rive-artboard="<artboard>"
        data-rive-state-machine="<machine>"
        data-rive-fit="contain">
</canvas>
```

### Rive Files

| File | Artboards | State Machines | Purpose |
|---|---|---|---|
| `reef` | `helmet-reef`, `off-icons` | `helmet-reef_scroll`, `helmet-reef_play`, `off-icons` | Animated racing helmet illustrations; nav helmet, color-input driven |
| `circuits` | `circuits` | `circuits` | Interactive F1 circuit maps (next race card, hover-driven) |
| `signature` | `signature` | `signature_scroll` | Lando's animated signature, scroll-driven |
| `phrases` | `phrase_on`, `collabs` | `phrase_on` | Animated text phrases (On Track section), scroll-triggered |
| `btn-ui` | `arrow`, `hamburger` | `arrow`, `hamburger` | Button arrow icons, hamburger menu, LN4 logo |

### Rive Initialization (Custom JS)

The custom JS bundle contains dedicated initialization functions:

| Function | Target File | Purpose |
|---|---|---|
| `B4()` | `reef` | Helmet animations with color input `data-rive-color-input` |
| `E4()` | `signature` | Signature scroll-driven animation |
| `I4()` | `circuits` | Circuit maps with input tracking (`data-rive-input-track`) |
| `C4()` | `btn-ui` (arrows) | Button arrow hover state machines |
| `iR()` | `btn-ui` (hamburger) | Hamburger menu toggle animation |
| `K4()` | `phrases` | Scroll-triggered phrase animations |
| `nR()` | LN4 logo | LN4 logo animation |

Preloading: All Rive files are preloaded via `kI()` and stored in a `DB` map. All Rive instances are tracked in the `UI` array for lifecycle management.

### Scroll-Driven Rive (`oR()`)

For state machines named with `_scroll` suffix, the `oR()` function creates a GSAP timeline + ScrollTrigger, mapping scroll progress to a Rive `scroll` input:

```js
// pseudo-logic: oR()
// 1. Find canvas with data-rive-scrolltrigger="true"
// 2. Create GSAP ScrollTrigger with target/start/end from data attrs
// 3. On update: riveInstance.input("scroll").value = progress (0→1)
```

### Play-on-Scroll Rive (`CH()`)

For state machines with `_play` suffix, `CH()` plays the state machine on ScrollTrigger enter:

```js
// pseudo-logic: CH()
// 1. Create ScrollTrigger that fires once on enter
// 2. riveInstance.play(stateMachineName)
```

### Rive Data Attributes

| Attribute | Example | Purpose |
|---|---|---|
| `data-rive-file` | `reef`, `circuits`, `signature`, `phrases`, `btn-ui` | Which Rive `.riv` file to load |
| `data-rive-artboard` | `helmet-reef`, `circuits`, `arrow` | Which artboard within the file |
| `data-rive-state-machine` | `helmet-reef_scroll` | Naming: `_scroll` = scroll-driven, `_play` = play on enter, no suffix = autoplay |
| `data-rive-input-track` | `monaco` | Circuit input name (circuits only) |
| `data-rive-input-color` | (empty) | Circuit fill color input (circuits only) |
| `data-rive-input-weight` | (empty) | Circuit stroke weight input (circuits only) |
| `data-rive-color-input` | `color_green-off-white-2` | Rive color input name (reef only) |
| `data-rive-scrolltrigger` | `true` | Enable scroll-driven progress mapping |
| `data-rive-scrolltrigger-target` | `.hero-rive-tracker` | Scroll trigger reference element selector |
| `data-rive-scrolltrigger-start` | `top center` | Scroll start boundary |
| `data-rive-scrolltrigger-end` | `bottom 80%` | Scroll end boundary |
| `data-rive-instant-play` | `true` | Autoplay state machine immediately on load |
| `data-rive-circuit-hover` | (empty) | Hover interaction for circuit map items |
| `data-rive-hover` | `false` | Boolean flag — flips to `true` on hover (btn-ui arrows) |
| `data-rive-input` | `color_lime` | Direct Rive input name for color driving |
| `data-rive-fit` | `contain` | Layout fit for Rive canvas |
| `data-rive-primary` | (present) | Page transition overlay canvas |
| `data-rive-nav-hamburger` | (present) | Hamburger menu Rive canvas (btn-ui file) |
| `data-rive-mob-landscape` | (present) | Mobile rotation prompt animation |
| `data-rive-ln4` | (present) | LN4 logo animation |
| `data-rive-nav-object` | (present) | Nav helmet (reef file) |
| `data-rive-placeholder` | (present) | SVG placeholder shown before Rive loads |

---

## All Interactive Effects

### 1. Hero — Sticky Section with Split-Text + Rive + Touch Lock

- **Structure:** `data-sticky-hero="track"` on `.sticky-track.home-hero`
- **Components:** Next race card (circuits Rive), helmet Rive (`helmet-reef_play` with `data-rive-instant-play`), hero text with `data-hero-anim="msg"` and `data-hero-anim="img"`, touch controls (`data-home-swipe-wrap`, `data-home-swipe-toggle`)
- **Detection:** `data-hero-animation-container` attribute triggers the custom JS hero init
- **Touch Lock:** `data-home-swipe-toggle` toggles between locked (`tap to lock`) and unlocked (`Back to scroll`) states — controls whether scroll advances past hero

### 2. Next Race Card — Rive Circuit Map

- **Structure:** `<canvas data-rive-file="circuits" data-rive-artboard="circuits" data-rive-state-machine="circuits">`
- **Mechanism:** `data-rive-input-track="monaco"` dynamically switches the Rive artboard input to the current F1 race circuit. Hover triggers circuit animation via `data-rive-circuit-hover`. Supports `data-rive-input-color` and `data-rive-input-weight` for dynamic styling.
- **Innovation:** Real-time F1 calendar data driving a Rive animation input.

### 3. Signature Scroll Reveal

- **Structure:** `<canvas data-rive-file="signature" data-rive-state-machine="signature_scroll" data-rive-input="color_lime">`
- **Mechanism:** Rive animation progress mapped to scroll via `data-rive-scrolltrigger-target=".hero-rive-tracker"`, `start="top center"`, `end="bottom 80%"`. The `oR()` function maps GSAP progress (0→1) to Rive `scroll` input.
- **Result:** Lando's signature draws itself as the user scrolls.

### 4. "Message from Lando" Text Char Reveal

- **Structure:** `[split-text="chars"]` + `data-hero-anim="msg"`
- **Mechanism:** GSAP SplitText 3.13.0 wraps each character in `<span class="char">`. Animated in sequence on scroll with stagger.

### 5. Horizontal Scroll Gallery

- **Structure:** `[data-horizontal-section]` on section, `[data-horizontal-track]` on inner track
- **Mechanism:** GSAP horizontal scroll — spacer creates scroll distance, track is pinned. Color transitions via `data-h-color-from="dark-green"` / `data-h-color-to="white"`.
- **Content:** Mixed grid of images, text callouts, and signature SVGs.

### 6. Scroll-Driven Nav Theme

- **Structure:** `.hero-nav-theme.is-1` (dark), `.hero-nav-theme.is-2` (light) — empty divs at scroll positions
- **Mechanism:** ScrollTrigger flips `data-nav-theme="dark"` / `data-nav-theme="light"` on `<nav>`. CSS transitions for brand logo fill (`nav-brand-path` paths), nav text color, hamburger border color via `--animation-default`.

### 7. Section Color Theme Transitions

- **Structure:** `data-gl-change-to="white, dark-green-tint-1-low"` / `data-gl-change-from="dark-green, dark-green-tint-1"` / `data-gl-change-track`
- **Mechanism:** ScrollTrigger-driven background/text color transitions. Custom JS tracks color changes and updates Rive color inputs (`data-rive-color-input`) to match.

### 8. Split-Text Character/Line Animations

- **Structure:** `[split-text="chars"]`, `[split-text="lines"]`, `[data-anim-high]`
- **Mechanism:** GSAP SplitText 3.13.0 wraps chars/lines in spans. Scroll reveal via `data-anim-high`:
  - `data-anim-high="right, lime"` — entrance from right, colored lime
  - `data-anim-high="left, dark-green-tint-1, 200"` — entrance from left with 200px offset
  - Lines use `clip-path: polygon(...)` for wipe-in reveal (`[split-text] .line { clip-path: polygon(0 -2%, 0 94%, 100% 94%, 100% -2%); }`)

### 9. Helmet Hover — Dual-Image Reveal

- **Structure:** `.helmet-grid-item-w` (base WebP image + hover reveal WebP image)
- **Mechanism:** Pure CSS. Reveal image default: `clip-path: ellipse(0% 0% at 50% 0%)`. Hover: `clip-path: ellipse(100% 120% at 50% 0%)` — smooth vertical wipe. SVG lime frames fade in via `opacity` transition. All use `--cubic-default: cubic-bezier(0.65, 0.05, 0, 1); --duration-default: 0.75s`.
- **Innovation:** Zero JavaScript for the reveal animation.

### 10. CSS Marquee (Infinite Scroll)

- **Structure:** `.marquee-offest-w`, `[data-css-marquee-list="left"|"right"]`
- **Mechanism:** `@keyframes translateXLeft { to { transform: translateX(-100%); } }`. `data-marquee-duplicate` controls clone count. Desktop mask at ≥1920px: `mask-image: linear-gradient(90deg, transparent, white 7.5%, white 92.5%, transparent)`.

### 11. Button Rive Hover

- **Structure:** `<canvas data-rive-file="btn-ui" data-rive-artboard="arrow" data-rive-state-machine="arrow" data-rive-hover="false">`
- **Mechanism:** On hover, `data-rive-hover` flips to `"true"`, triggering Rive arrow morph/slide. Some buttons also rotate 180° via `data-btn-rive-rotate="true"` or 90° via `data-btn-rive-rotate="90"`. `data-rive-btn-invert="true"` inverts icon color via CSS `filter: invert(1)`.

### 12. Social Media Video Cards

- **Structure:** `.callout-socials-card-layout`, `[data-video-stream]`, `[data-stream-hover]`
- **Mechanism:** Video hidden behind placeholder image (`[data-video-stream-placeholder]`). On hover (`data-stream-hover="true"`), video plays. Supports `data-stream-muted`, `data-stream-loop`, `data-stream-autoplay`, and `data-stream-url` (Vimeo ID).

### 13. Footer SVG Mask Clip

- **Structure:** `footer` with `.footer-layout-clip` — `--mask-url: url('...ln4-footer-mask-desktop.svg')`
- **Mechanism:** Mobile variant: `url('...ln4-footer-mask-mobile.svg')` at ≤479px. Creates organic silhouette shape for the footer content area. Pure CSS.

### 14. Animated Helmet SVG Masks

- **Structure:** `[data-helmet-item]` with `--mask-url: url('...ln4-2-helm-mask-fill.svg')`
- **Mechanism:** Mobile variant at ≤479px: `url('...ln4-helm-mob-refactor-lime-fill.svg')`. Same pattern used for stats boxes (`on-t-hero-stats-ui-box1-fill-c.svg`), calendar track masks, and case study cards (`data-mask-case-left`, `data-mask-case-right`, `data-mask-callout-left`, `data-mask-callout-right`).

### 15. "On Track / Off Track" Parallax Section

- **Structure:** `.sticky-track.is-home-otot` with `data-otot-section`
- **Mechanism:** Two columns (ON TRACK / OFF TRACK) with Rive phrase animation (`phrases` file, `data-rive-scrolltrigger="true"`). Bottom section `.s.is-otot-end` reveals Lando lifting his helmet via GSAP.

### 16. Page Transitions (Rive Overlay)

- **Structure:** `.transition-w` with full-screen `<canvas data-rive-primary>`
- **Mechanism:** Full-screen Rive state machine plays in/out transitions. Separate `transitions-rive-isolate.js` referenced (commented out in production HTML).

### 17. Mobile Rotation Prompt

- **Structure:** `<canvas data-rive-mob-landscape>` inside `.mob-landscape-block`
- **Mechanism:** Animated phone-rotation graphic. CSS media queries detect landscape orientation on mobile.

### 18. Custom Scroll Indicator

- **Structure:** `.scroll-indicator-bar` inside `.scroll-indicator`
- **Mechanism:** Width percentage set by custom JS based on document scroll progress.

### 19. Nav Menu with Rive Helmet

- **Structure:** Nav menu contains `<canvas data-rive-nav-object>` with `reef` file, `helmet-reef` artboard, `helmet-reef_scroll` state machine, `data-rive-color-input="color_green-off-white-2"`.
- **Mechanism:** Helmet color can change based on theme. Menu also features image galleries with blend modes.

---

## Key Innovations

### 1. Rive as Primary Animation Engine

No Three.js, no Lottie, no GSAP timelines for most effects — **Rive** handles helmet animations, circuit maps, signatures, buttons, page transitions, logo, phrases. Rive acts as both design tool and runtime. Benefits: single pipeline (design → code), canvas-rendered (performant), state-machine driven (complex interactions without code).

### 2. GSAP-Style API Mapped to Rive Inputs

The custom JS framework creates a bridge between GSAP ScrollTrigger and Rive. `data-rive-scrolltrigger-target/start/end` map directly to scroll position via `oR()` — giving Rive scroll-driven capabilities with Canvas rendering performance.

### 3. CSS Custom Property Fluid Typography System

```css
:root {
  --min-width: 992px;  --max-width: 1920px;
  --design-width: 1728; --design-unit: 16;
  --fluid-container: clamp(var(--min-width), 100vw, var(--max-width));
  --fluid-font: calc(var(--fluid-container) / var(--design-width) * var(--design-unit));
}
html { font-size: var(--fluid-font); }
```

Every `rem` value scales proportionally. Breakpoint overrides adjust `--min-width`, `--max-width`, and `--design-unit` (tablet: 20, mobile portrait: 48).

### 4. Pure CSS Helmet Hover

Two stacked WebP images, `clip-path: ellipse()` transition on the reveal layer — zero JavaScript. SVG lime frames fade in on hover via CSS `opacity`.

### 5. Unified Custom Easing

```css
--cubic-default: cubic-bezier(0.65, 0.05, 0, 1);
--duration-default: 0.75s;
--animation-default: var(--duration-default) var(--cubic-default);
```

Used universally across CSS transitions and GSAP tweens.

### 6. No-Framework Architecture

Server-rendered Webflow + vanilla JS. No SPA framework, no React/Vue. Page transitions via Rive overlay. Lightweight despite visual complexity.

### 7. State Machine Naming Convention

`_scroll` suffix → scroll-driven via `oR()`. `_play` suffix → play-on-enter via `CH()`. No suffix → autoplay on load. This convention makes the animation intent clear from HTML alone.

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────┐
│                 Webflow (CMS + Hosting)                  │
├─────────────────────────────────────────────────────────┤
│  ┌────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │   Rive     │  │ GSAP 3.13.0 │  │   Custom JS      │  │
│  │   Canvas   │  │  ScrollTr.  │  │  (OFF+BRAND)     │  │
│  │────────────│  │─────────────│  │──────────────────│  │
│  │B4() reef   │  │oR() scroll  │  │kI() preloader    │  │
│  │E4() signat │  │CH() play    │  │DB map + UI arr   │  │
│  │I4() circ   │  │horizontal   │  │marquee cloning   │  │
│  │C4() btn-ui │  │charReveal   │  │nav theming       │  │
│  │iR() hambrg │  │pin/sticky   │  │scroll indicator  │  │
│  │K4() phrses │  │color swps   │  │video streams     │  │
│  │nR() LN4    │  │parallax     │  │touch lock        │  │
│  └────────────┘  └──────────────┘  └─────────────────┘  │
├─────────────────────────────────────────────────────────┤
│              CSS (Fluid + Masks + Clip-path)             │
│  ┌──────────────┐  ┌────────────┐  ┌──────────────────┐ │
│  │fluid font    │  │SVG masks   │  │clip-path ellipse │ │
│  │scaling       │  │footer/     │  │helmet hover wipe │ │
│  │              │  │helmets/    │  │                  │ │
│  │              │  │stats/cases │  │  Lenis smooth    │ │
│  └──────────────┘  └────────────┘  │  scrolling       │ │
│                                     └──────────────────┘ │
└─────────────────────────────────────────────────────────┘
```
