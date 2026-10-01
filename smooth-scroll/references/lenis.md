# Lenis reference

Source: Lenis README (darkroomengineering/lenis, v1.3.26 at time of writing). Check npm for the latest version before pinning a CDN URL.

## Contents
- What it is
- Install
- Setup (basic, custom loop, CSS, React, no-code)
- GSAP ScrollTrigger sync
- Settings
- Methods and properties
- Events
- Nested scroll, anchors, reduced motion
- Limitations

## What it is

Lightweight, dependency-free smooth scroll that wraps the browser's native scroll. Because the scroll is still native, `position: sticky`, anchor links and accessibility keep working. Supports any axis, WebGL and ScrollTrigger sync, and has adapters for React, Vue and Framer plus a snap plugin.

Packages: `lenis`, `lenis/react`, `lenis/vue`, `lenis/framer`, `lenis/snap`.

## Install

```bash
npm i lenis
```
```js
import Lenis from 'lenis'
```
CDN:
```html
<script src="https://unpkg.com/lenis@1.3.26/dist/lenis.min.js"></script>
```

## Setup

### Basic
```js
const lenis = new Lenis({ autoRaf: true })
lenis.on('scroll', (e) => console.log(e))
```

### Custom raf loop
```js
const lenis = new Lenis()
function raf(time) {
  lenis.raf(time)
  requestAnimationFrame(raf)
}
requestAnimationFrame(raf)
```

### Recommended CSS (do not skip)
```js
import 'lenis/dist/lenis.css'
```
or
```html
<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.26/dist/lenis.css">
```

### React
Use the `lenis/react` package (`ReactLenis` component and `useLenis` hook). The README only links to the package README, so open `packages/react/README.md` in the repo and follow it for the exact props before writing React code. Typical shape is a `ReactLenis` wrapper with the `root` prop for page-level scroll and `options` for the settings below.

### No-code, one line
```html
<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.26/dist/lenis.css">
<script src="https://unpkg.com/lenis@1.3.26/dist/lenis.min.js"></script>
<script>new Lenis({ autoRaf: true, autoToggle: true, anchors: true, allowNestedScroll: true, naiveDimensions: true, stopInertiaOnNavigate: true })</script>
```
Handles compatibility with other packages, modals, smooth anchors and scroll reset on page change. Note `naiveDimensions` has a performance cost and `autoToggle` needs the recommended CSS plus Safari > 17.3, Chrome > 116, Firefox > 128.

## GSAP ScrollTrigger sync

Use this whenever ScrollTrigger animations exist on a Lenis page.

```js
const lenis = new Lenis()

// tell ScrollTrigger every time Lenis scrolls
lenis.on('scroll', ScrollTrigger.update)

// drive Lenis from GSAP's ticker (seconds to milliseconds)
gsap.ticker.add((time) => {
  lenis.raf(time * 1000)
})

// stop GSAP from smoothing over lag, which would desync the scroll
gsap.ticker.lagSmoothing(0)
```
Do not also set `autoRaf: true` here. One loop only.

## Settings

| Option | Default | Notes |
|---|---|---|
| `autoRaf` | false | runs the rAF loop for you |
| `lerp` | 0.1 | 0 to 1, lower is floatier. If set, `duration` and `easing` are ignored |
| `duration` | 1.2 | seconds, ignored if `lerp` set |
| `easing` | custom expo-out | any function, see easings.net |
| `smoothWheel` | true | smooth mouse wheel |
| `wheelMultiplier` | 1 | scroll speed for wheel |
| `touchMultiplier` | 1 | scroll speed for touch |
| `syncTouch` | false | mimic touch inertia with sync, unstable on iOS < 16 |
| `syncTouchLerp` | 0.075 | inertia lerp for syncTouch |
| `touchInertiaExponent` | 1.7 | syncTouch inertia strength |
| `orientation` | vertical | or horizontal |
| `gestureOrientation` | vertical | vertical, horizontal or both |
| `infinite` | false | needs `syncTouch: true` on touch devices |
| `wrapper` | window | scroll container |
| `content` | documentElement | element being scrolled |
| `eventsTarget` | wrapper | listens for wheel and touch |
| `autoResize` | true | ResizeObserver; if false call `resize()` |
| `autoToggle` | false | start/stop from wrapper overflow, needs the CSS |
| `anchors` | false | true or ScrollToOptions |
| `allowNestedScroll` | false | lets inner scrollers scroll natively, checks DOM each scroll |
| `prevent` | undefined | `(node) => boolean`, true skips smoothing |
| `virtualScroll` | undefined | modify events, return false to skip smoothing |
| `overscroll` | true | like CSS overscroll-behavior |
| `respectReducedMotion` | true | keep true |
| `stopInertiaOnNavigate` | false | stop inertia on internal link click |
| `naiveDimensions` | false | performance cost |

## Methods

- `scrollTo(target, options)`: target is a number (px), a selector or keyword (`top`, `left`, `start`, `bottom`, `right`, `end`), or an element. Options: `offset`, `lerp`, `duration`, `easing`, `immediate`, `lock`, `force`, `onComplete`, `userData`.
- `start()` / `stop()`: resume or pause scrolling (use for modals and loaders).
- `raf(time)` (ms), `resize()`, `destroy()`, `on(event, fn)`.

## Properties

`scroll`, `animatedScroll`, `targetScroll`, `actualScroll`, `velocity`, `lastVelocity`, `direction`, `progress`, `limit`, `isScrolling` (`'smooth'`, `'native'` or false), `isStopped`, `isHorizontal`, `prefersReducedMotion`, `options`, `rootElement`, `dimensions`, `time`.

## Events

`scroll` (receives the Lenis instance), `virtual-scroll` (`{deltaX, deltaY, event}`).

## Nested scroll

Simplest: `new Lenis({ allowNestedScroll: true })`. If performance suffers, mark elements instead:

| Attribute | Effect |
|---|---|
| `data-lenis-prevent` | block all smoothing on the element |
| `data-lenis-prevent-wheel` | wheel only |
| `data-lenis-prevent-touch` | touch only |
| `data-lenis-prevent-vertical` | vertical only |
| `data-lenis-prevent-horizontal` | horizontal only |

Or in JS: `prevent: (node) => node.id === 'modal'`.

## Anchors

Blocked while scrolling by default. Enable with `anchors: true`, or pass options such as `anchors: { offset: 100, onComplete: () => {} }`.

## Reduced motion

On by default. When the user prefers reduced motion, smoothing is disabled (`lerp` forced to 1) and `scrollTo` jumps instantly, but Lenis keeps running so WebGL sync still works. Check `lenis.prefersReducedMotion` to adapt custom animations. Opting out is not recommended.

## Limitations

- No CSS scroll-snap support, use `lenis/snap`.
- Capped at 60fps on Safari, 30fps in low power mode.
- Smooth scroll stops over iframes.
- `position: fixed` can lag on pre-M1 MacOS Safari.
- `syncTouch` can misbehave on iOS < 16.
- Nested scroll containers need configuration.
