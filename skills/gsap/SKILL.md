---
name: gsap
description: 'GSAP conventions for all animation: data-* hooks, transform and opacity, timelines, ScrollTrigger, reduced motion. Use whenever something moves, reveals or animates. Trigger on: gsap.to/from/fromTo/set, gsap.timeline, ScrollTrigger, scrub, stagger, easing, data-animate, entrance or reveal animations, scroll animations, parallax, page-load sequences, accordion or menu open/close animation, gsap.context, gsap.matchMedia, prefers-reduced-motion, replacing CSS @keyframes or x-transition. Do NOT trigger for: simple hover/focus color or opacity changes on buttons and links (Tailwind transition-* classes, see tailwindcss), state and events (alpinejs).'
---

# GSAP

## Contents

- [Documentation](#documentation)
- [When to use](#when-to-use)
- [Core principles](#core-principles)
- [Conventions](#conventions)
  - [Setup](#setup)
  - [ScrollTrigger](#scrolltrigger)
  - [Defaults & easing](#defaults--easing)
  - [Preventing FOUC](#preventing-fouc)
  - [Cleanup](#cleanup)
  - [Accessibility](#accessibility)
- [Anti-patterns](#anti-patterns)
- [Output checklist](#output-checklist)
- [Working with Alpine.js](#working-with-alpinejs)

## Documentation

- [GSAP docs](https://gsap.com/docs/v3/) — tweens, timelines, easing
- [ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/) — scroll-based animations
- [llms.txt](https://gsap.com/llms.txt) — full docs in LLM-friendly format; fetch for authoritative, up-to-date reference

## When to use

- Adding animations and transitions (entrances, hovers, micro-interactions)
- Scroll-based animations with ScrollTrigger
- Complex sequences with timelines

## Core principles

1. **GSAP for all animations** — no CSS `@keyframes`; a single animation system keeps everything consistent and manageable. The one exception: simple hover/focus state changes (color, opacity) on buttons and links use Tailwind `transition-*` classes — driving those through GSAP is overkill.
2. **Animate `transform` and `opacity`** — `x`, `y`, `scale`, `rotation`, `opacity` are GPU-friendly. Avoid animating layout properties (`width`, `height`, `top`, `left`); they force a reflow on every frame. The one accepted exception is a collapsing panel (accordion, dropdown) where the content height is unknown: there `height: 'auto'` has no transform equivalent — see [Working with Alpine.js](#working-with-alpinejs).
3. **Target via data attributes** — use `data-*` attributes as animation hooks, not HTML classes. Classes are for styling (Tailwind); data attributes make the animation intent explicit and don't break on restyling:

   ```html
   <h1 data-animate="hero-title">...</h1>
   <p data-animate="hero-sub">...</p>
   ```

4. **Timelines for sequences** — don't stack nested `delay`s:

   ```js
   const tl = gsap.timeline({ defaults: { ease: 'power2.out', duration: 0.6 } });
   tl.from('[data-animate="hero-title"]', { y: 40, opacity: 0 })
     .from('[data-animate="hero-sub"]', { y: 20, opacity: 0 }, '-=0.3')
     .from('[data-animate="hero-cta"]', { scale: 0.9, opacity: 0 }, '-=0.2');
   ```

## Conventions

### Setup

```js
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);
```

### ScrollTrigger

```js
gsap.from('[data-animate="card"]', {
  y: 60,
  opacity: 0,
  stagger: 0.15,
  scrollTrigger: {
    trigger: '[data-animate-section="cards"]',
    start: 'top 80%',
    toggleActions: 'play none none reverse',
  },
});
```

- Set `start`/`end` explicitly; use `markers: true` only while debugging
- `scrub: true` for scroll-linked animations; `toggleActions` for enter/leave-triggered animations (`'play none none reverse'` replays on scroll-back, `'play none none none'` fires once)

### Defaults & easing

- Set `defaults` on timelines to avoid repetition
- Default easing: `power2.out` for entrances, `power2.inOut` for movements
- Durations: 0.3–0.8s for UI; shorter (0.2s) for micro-interactions

### Preventing FOUC

Use `gsap.set()` or animate with `.from()` and put the end state in the markup — this keeps the page correct without JavaScript.

### Cleanup

For dynamic content or SPA-like navigation: use `gsap.context()` and `ctx.revert()` to clean up animations and ScrollTriggers.

### Accessibility

Respect `prefers-reduced-motion`:

```js
gsap.matchMedia().add('(prefers-reduced-motion: no-preference)', () => {
  // animations here
});
```

## Anti-patterns

| Avoid | Do instead |
|---|---|
| CSS keyframes or transitions for real animations | GSAP — only hover/focus state changes use `transition-*` |
| Targeting HTML classes (`.hero-title`) | Data attributes (`[data-animate="hero-title"]`) |
| Animating `width`/`height`/`top`/`left` | `x`, `y`, `scale`, `opacity` — only a collapsing panel's `height` is exempt |
| Separate tweens with stacked `delay`s | Timeline with position parameters (`'-=0.3'`) |
| ScrollTrigger per element in a loop without cleanup | `gsap.context()` + `stagger` |
| Animating without a reduced-motion check | `gsap.matchMedia()` |
| Writing your own `setInterval`/`requestAnimationFrame` | `gsap.ticker` or tweens |

## Output checklist

Verify every item before delivering:

- [ ] Every real animation runs through GSAP — no CSS `@keyframes`, no `x-transition`; only hover/focus color/opacity changes use Tailwind `transition-*`
- [ ] Targets are `data-*` attributes (`[data-animate="…"]`), never HTML classes
- [ ] Only `transform` properties (`x`, `y`, `scale`, `rotation`) and `opacity` are animated — the one exception is the `height` of a collapsing panel
- [ ] Sequences are a timeline with `defaults` and position parameters, not tweens with stacked `delay`s
- [ ] ScrollTriggers have an explicit `start`/`end`; no `markers: true` left in the code
- [ ] The end state is in the markup (`.from()` / `gsap.set()`), so the page is correct without JavaScript
- [ ] Everything is wrapped in `gsap.matchMedia()` with `(prefers-reduced-motion: no-preference)`
- [ ] Dynamic content cleans up through `gsap.context()` + `ctx.revert()`
- [ ] Plugins are registered with `gsap.registerPlugin()`; no hand-written `setInterval` / `requestAnimationFrame` loops

## Working with Alpine.js

Alpine manages state and events, GSAP performs the animation:

```js
Alpine.data('accordion', () => ({
  open: false,
  toggle() {
    this.open = !this.open;
    gsap.to(this.$refs.panel, {
      height: this.open ? 'auto' : 0,
      duration: 0.4,
      ease: 'power2.inOut',
    });
  },
}));
```

Don't use `x-transition` — animate via GSAP in the component (see [alpinejs skill](../alpinejs/SKILL.md)).
