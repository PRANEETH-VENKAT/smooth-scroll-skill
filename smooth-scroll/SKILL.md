---
name: smooth-scroll
description: Make websites scroll smoothly using Lenis or GSAP ScrollSmoother. Use this skill whenever the user wants smooth scrolling, buttery or inertia scroll, scroll parallax, scroll-linked animation, "make my site feel smoother", "scroll feels stiff or janky", a portfolio scroll journey, or mentions Lenis, ScrollSmoother, ScrollTrigger, data-speed, data-lag or lerp. Also use it to debug scroll problems such as broken position fixed, sticky headers, anchor links, modals that still scroll, nested scroll areas, or mobile scroll jumps. Covers plain HTML and React, and decides which library fits.
---

# Smooth scroll

Two libraries cover almost every case. Pick one, set it up from the matching reference file, then run the bug checklist at the bottom.

## Step 1: Pick the library

Ask yourself these in order and stop at the first match.

1. Does the project already use GSAP ScrollTrigger heavily and the user wants built-in parallax (`data-speed`) or lag (`data-lag`) effects with almost no code? Use **ScrollSmoother**. Read `references/gsap-scrollsmoother.md`.
2. Is it React, Vue, Framer, or a site that needs `position: fixed` / `sticky` to keep working without rearranging markup? Use **Lenis**. Read `references/lenis.md`.
3. Does the user need WebGL or canvas scroll syncing, horizontal scroll, infinite scroll, or a no-build one-line drop-in? Use **Lenis**.
4. Still unsure, or the user just says "make it smooth"? Default to **Lenis**. It needs no wrapper markup, no GSAP, and is a few KB.

Why this order: ScrollSmoother moves a content div with CSS transforms, so it forces a wrapper structure and breaks `position: fixed` inside it. Lenis smooths the real scroll position, so existing layouts keep working. That makes Lenis the safer default and ScrollSmoother the specialist.

## Step 2: Never run both on one page

Both take over the root page scroll. Running Lenis and ScrollSmoother together fights over the same scroll position and produces jitter. If the user already has one, extend it or migrate fully. If they want GSAP ScrollTrigger animations alongside smooth scroll, use Lenis plus ScrollTrigger sync (recipe in `references/lenis.md`), or ScrollSmoother alone.

## Step 3: Implement

- Read only the one reference file you chose.
- Match the user's stack. Plain HTML gets a script tag version, React gets the React version.
- Keep the first version minimal: default settings, correct structure, correct loop wiring. Tune feel (lerp, duration, smooth) after it works, because most "it feels wrong" complaints are actually wiring bugs.

## Step 4: Tuning the feel

Tell the user what each knob does so they can steer you:

| Want | Lenis | ScrollSmoother |
|---|---|---|
| Floatier, longer glide | lower `lerp` (e.g. 0.05) | raise `smooth` (e.g. 1.5) |
| Snappier | higher `lerp` (e.g. 0.2) | lower `smooth` (e.g. 0.5) |
| Faster or slower scrolling overall | `wheelMultiplier` | `speed` |
| Depth / parallax | build with ScrollTrigger yourself | `data-speed`, `data-lag` |

## Accessibility and mobile (always check)

- Respect `prefers-reduced-motion`. Lenis does this by default (`respectReducedMotion: true`). Do not turn it off. With ScrollSmoother, consider skipping creation when the user prefers reduced motion.
- Smoothing on touch devices usually feels wrong under a finger. ScrollSmoother is off on touch by default (`smoothTouch`), and Lenis only smooths wheel by default (`syncTouch: false`). Leave these defaults unless the user asks.
- Lenis is capped at 60fps on Safari and 30fps in low power mode. Warn the user if they report Safari "not as smooth as Chrome".

## Bug checklist

Run through this when the user says scroll is broken, janky, or partly smooth.

| Symptom | Likely cause | Fix |
|---|---|---|
| No smoothing at all (Lenis) | raf loop not running | set `autoRaf: true` or call `lenis.raf(time)` each frame |
| Scroll feels broken or styles odd (Lenis) | recommended CSS missing | import `lenis/dist/lenis.css` |
| Anchor links do nothing (Lenis) | blocked by default | `anchors: true` |
| Modal or dropdown scrolls the page behind it | scroll not stopped | `lenis.stop()` / `lenis.start()`, or `smoother.paused(true)` |
| Inner scrollable box won't scroll (Lenis) | nested scroll captured | `allowNestedScroll: true` or `data-lenis-prevent` on the box |
| Fixed navbar jumps or detaches (ScrollSmoother) | fixed element inside wrapper | move it outside `#smooth-wrapper` |
| ScrollTrigger animations out of sync with Lenis | not wired together | use the GSAP sync recipe |
| Parallax element starts offset (ScrollSmoother) | natural position is viewport center | `data-speed="clamp(0.5)"` |
| Scroll jumps on phone when address bar hides | viewport resize triggers refresh | `normalizeScroll: true` and `ignoreMobileResize` (ScrollSmoother) |
| Smooth scroll stops over an embed | iframes swallow wheel events | known limit, no fix |
| CSS scroll-snap behaves badly (Lenis) | unsupported with Lenis | use `lenis/snap` |
| Effects stack wrong (ScrollSmoother) | nested `data-speed` elements | effects must not be nested |

If none match, follow the Lenis troubleshooting order: latest version, CSS included, test the page without the library, then check the frame loop.

## Delivering the result

State which library you chose and why in one or two sentences, give the code, and list the two or three settings the user is most likely to tweak. If you are editing an existing project, change the minimum number of files and say which ones.
