---
name: gsap
description: 'GSAP conventions and best practices for animations. Use when creating animations, transitions, scroll animations (ScrollTrigger), timelines, hover effects and micro-interactions. Build all animations with GSAP, not with CSS animations.'
---

# GSAP

## Documentation

- [GSAP docs](https://gsap.com/docs/v3/) — tweens, timelines, easing
- [ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/) — scroll-based animations

## When to use

- Adding animations and transitions (entrances, hovers, micro-interactions)
- Scroll-based animations with ScrollTrigger
- Complex sequences with timelines

## Core principles

1. **GSAP for all animations** — no CSS `transition`/`@keyframes`; a single animation system keeps everything consistent and manageable.
2. **Animate `transform` and `opacity`** — `x`, `y`, `scale`, `rotation`, `opacity` are GPU-friendly. Avoid animating `width`, `height`, `top`, `left`.
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
- `scrub: true` for scroll-linked animations, `toggleActions` for one-time triggers

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
| CSS transitions/keyframes alongside GSAP | Everything via GSAP |
| Targeting HTML classes (`.hero-title`) | Data attributes (`[data-animate="hero-title"]`) |
| Animating `width`/`height`/`top`/`left` | `x`, `y`, `scale`, `opacity` |
| Separate tweens with stacked `delay`s | Timeline with position parameters (`'-=0.3'`) |
| ScrollTrigger per element in a loop without cleanup | `gsap.context()` + `stagger` |
| Animating without a reduced-motion check | `gsap.matchMedia()` |
| Writing your own `setInterval`/`requestAnimationFrame` | `gsap.ticker` or tweens |

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
