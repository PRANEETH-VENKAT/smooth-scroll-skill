# GSAP ScrollSmoother reference

Source: GSAP docs v3.15 (ScrollSmoother, added in v3.10.0). Check the current GSAP docs for install and licensing before writing install instructions.

## Contents
- What it is
- Required structure
- Setup
- Config options
- Effects (data-speed, data-lag)
- Methods and properties
- Recipes
- Caveats

## What it is

Adds smooth scrolling to a ScrollTrigger-based page. It uses native scrolling (no fake scrollbar, no touch hijacking), then applies CSS transforms (`matrix3d()`) to a content element so it gradually catches up to the native scroll position. It is built on ScrollTrigger and also avoids multi-thread issues such as pin/unpin jumps and pinned-element jitter. Only one instance can exist at a time.

## Required structure

All page content lives in one content element, inside a wrapper that acts as the viewport. The scrollbar stays on `<body>`.

```html
<body>
  <div id="smooth-wrapper">
    <div id="smooth-content">
      <!-- ALL CONTENT HERE -->
    </div>
  </div>
  <!-- position: fixed elements go OUTSIDE, here -->
</body>
```
The ids `smooth-wrapper` and `smooth-content` are found automatically. If no wrapper exists, one is created.

## Setup

```js
gsap.registerPlugin(ScrollTrigger, ScrollSmoother)

// create BEFORE any other ScrollTriggers
const smoother = ScrollSmoother.create({
  smooth: 1,        // seconds to catch up
  effects: true,    // enable data-speed and data-lag
  smoothTouch: 0.1  // light smoothing on touch (default is none)
})
```

React: use the `useGSAP()` hook from GSAP's React helper and create the smoother inside it so it is cleaned up on unmount. Verify the current pattern in the GSAP React docs.

## Config options

| Option | Notes |
|---|---|
| `smooth` | seconds to catch up, default 0.8 |
| `smoothTouch` | false by default; `true` or a number like 0.1 to smooth touch |
| `ease` | default `"expo"` |
| `speed` | overall scroll speed multiplier (3.11.4+) |
| `effects` | `true`, a selector, or an array of elements |
| `effectsPadding` | pixels to extend when effects start and end (3.11.4+) |
| `effectsPrefix` | e.g. `"scroll-"` makes `data-scroll-speed` (3.10.5+) |
| `normalizeScroll` | forces scroll onto the JS thread, stops most address-bar hide/show on mobile |
| `ignoreMobileResize` | skip refresh on small vertical resizes on touch devices to avoid jumps |
| `wrapper`, `content` | override the default elements |
| `onUpdate`, `onStop`, `onFocusIn` | callbacks; return false from `onFocusIn` to skip auto scroll-into-view |

## Effects

Needs `effects: true`.

```html
<div data-speed="0.5"></div>  <!-- half speed -->
<div data-speed="2"></div>    <!-- double speed -->
<div data-speed="auto"></div> <!-- auto parallax -->
<div data-speed="clamp(0.5)"></div> <!-- starts at natural position if above the fold (3.12+) -->
<div data-lag="0.5"></div>    <!-- takes 0.5s to catch up -->
```

- Elements reach their natural position when centered vertically in the viewport. That is why they can start offset.
- `data-speed="auto"`: make the child larger than its parent, align its top (or bottom) edge, set `overflow: hidden` on the parent. It computes the travel distance itself.
- Give neighbors slightly different `data-lag` values for a staggered feel.
- Effects must not be nested.

Via JS:
```js
smoother.effects(".box", { speed: 0.5, lag: 0.1 })
smoother.effects(".box", { speed: (i, el) => 0.5 + i * 0.1, lag: (i, el) => 0.3 + i * 0.05 })
smoother.effects(".box", { speed: 1, lag: 0 })       // remove
smoother.effects().forEach((t) => t.kill())          // kill all effects
```
Function-based values refresh on `ScrollTrigger.refresh()`.

## Methods and properties

| Item | Purpose |
|---|---|
| `ScrollSmoother.create(vars)` | create (kills any existing instance) |
| `ScrollSmoother.get()` | get the current instance |
| `.smooth(seconds)` | get/set smoothing |
| `.paused(bool)` | `true` stops all scrolling including scrollbar drag; good for modals |
| `.scrollTo(target, smooth, position)` | `smooth` true uses configured smoothing, false jumps; `position` like `"top 100px"` |
| `.scrollTop(px)` | instant get/set, works while paused |
| `.offset(target, position)` | pixel scroll position where target reaches `position` |
| `.getVelocity()` | px per second of smoothed scroll |
| `.effects(targets, config)` | add or get effects |
| `.content(el)`, `.wrapper(el)` | get/set elements |
| `.kill()` | remove smoother and effects |
| `.progress` | 0 to 1 page progress |
| `.scrollTrigger` | the internal ScrollTrigger |
| `.vars` | original config |

## Recipes

Smooth jump to a section, 100px from the top:
```js
button.addEventListener("click", () => smoother.scrollTo("#box1", true, "top 100px"))
```

Custom-animated scroll (clamped to the page end):
```js
gsap.to(smoother, {
  scrollTop: Math.min(ScrollTrigger.maxScroll(window), smoother.offset("#box1", "top 100px")),
  duration: 1
})
```

Modal toggle:
```js
smoother.paused(true)   // opening
smoother.paused(false)  // closing
```

## Caveats

- `position: fixed` inside the wrapper breaks, because the transformed content becomes the containing block. Put fixed elements outside the wrapper, or use ScrollTrigger pinning.
- `normalizeScroll: true` cannot stop the address bar from hiding on iOS in portrait. If that causes jumps, use `ScrollTrigger.config({ ignoreMobileResize: true })`.
- Touch devices get no smoothing unless `smoothTouch` is set; `scrollTo(..., true)` also has no smoothing on mobile by default.
- Create the smoother before any ScrollTriggers.
