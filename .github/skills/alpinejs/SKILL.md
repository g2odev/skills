---
name: alpinejs
description: 'Alpine.js conventions and best practices. Use when adding interactivity to the UI: dropdowns, modals, tabs, toggles, forms and other client-side behavior with x-data, x-on, x-show and other Alpine directives.'
---

# Alpine.js

## Contents

- [Documentation](#documentation)
- [When to use](#when-to-use)
- [Core principles](#core-principles)
- [Conventions](#conventions)
  - [Directives](#directives)
  - [Event modifiers](#event-modifiers)
  - [x-show vs x-if](#x-show-vs-x-if)
  - [Accessibility](#accessibility)
- [Anti-patterns](#anti-patterns)
- [Working with GSAP](#working-with-gsap)

## Documentation

- [Alpine.js docs](https://alpinejs.dev/start-here) — directives, magics, plugins

## When to use

- Adding interactivity to HTML (dropdowns, modals, tabs, accordions, toggles)
- Handling state and events in the UI without a build step
- Reviewing or refactoring existing Alpine components

## Core principles

1. **Declarative in the markup** — behavior belongs in `x-` directives on the element itself, not in separate script tags with query selectors.
2. **Small, local state** — `x-data` per component. For shared state: `Alpine.store()`.
3. **Reusable logic** — extract more complex components to `Alpine.data()`:

   ```js
   document.addEventListener('alpine:init', () => {
     Alpine.data('dropdown', () => ({
       open: false,
       toggle() { this.open = !this.open },
     }))
   })
   ```

   ```html
   <div x-data="dropdown">
     <button @click="toggle">Menu</button>
     <div x-show="open" @click.outside="open = false">...</div>
   </div>
   ```

## Conventions

### Directives

| Directive | Usage |
|---|---|
| `x-data` | Component state; keep it small |
| `x-show` | Show/hide (element stays in the DOM) |
| `x-if` | Conditional rendering (only on `<template>`) |
| `x-on:` / `@` | Events; use the short `@` syntax |
| `x-bind:` / `:` | Binding attributes; use the short `:` syntax |
| `x-model` | Two-way binding on form fields |
| `x-text` / `x-html` | Setting text/HTML — never `x-html` with user input (XSS) |
| `x-ref` / `$refs` | Element references instead of `querySelector` |
| `x-cloak` | Hide until Alpine initializes — rarely needed here, since GSAP owns initial states (see [Preventing FOUC](../gsap/SKILL.md#preventing-fouc)); if used, it needs the one-liner `[x-cloak] { display: none; }` in `app.css` |

### Event modifiers

Use modifiers instead of custom logic: `@click.outside`, `@keydown.escape`, `@submit.prevent`, `@input.debounce.300ms`, `@scroll.window.throttle`.

### x-show vs x-if

- `x-show` for frequent toggling (cheap)
- `x-if` when the content is heavy or must not be rendered

### Accessibility

- Bind ARIA attributes to state: `:aria-expanded="open"`
- Manage focus in modals: `x-trap` (focus plugin)
- `@keydown.escape.window` to close overlays

## Anti-patterns

| Avoid | Do instead |
|---|---|
| Large objects/logic inline in `x-data` | `Alpine.data()` component |
| `document.querySelector` in Alpine code | `x-ref` / `$refs` |
| Global variables for shared state | `Alpine.store()` |
| `x-html` with untrusted data | `x-text` |
| jQuery or vanilla event listeners alongside Alpine | Everything via `x-on` |
| `x-transition` for show/hide animations | GSAP — all animations go through one system (see [gsap skill](../gsap/SKILL.md)) |

## Working with GSAP

Use Alpine for state and events, GSAP for the animation itself — not `x-show` + `x-transition`. The markup binds state and refs:

```html
<div x-data="accordion">
  <button @click="toggle" :aria-expanded="open">Toggle</button>
  <div x-ref="panel" class="h-0 overflow-hidden">...</div>
</div>
```

The matching `Alpine.data('accordion')` component animates `$refs.panel` with `gsap.to()` — see the [gsap skill](../gsap/SKILL.md#working-with-alpinejs) for that side of the code.
