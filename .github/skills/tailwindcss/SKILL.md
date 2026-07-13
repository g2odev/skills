---
name: tailwindcss
description: 'Tailwind CSS v4 conventions and best practices. Use when writing, reviewing or refactoring Tailwind classes, styling components, theming with CSS variables, responsive design, dark mode and setting up a consistent utility-first design system.'
---

# Tailwind CSS

## Contents

- [Documentation](#documentation)
- [When to use](#when-to-use)
- [Core principles](#core-principles)
- [Conventions](#conventions)
  - [Colors: oklch](#colors-oklch)
  - [Base `<html>` element](#base-html-element)
  - [Layout & container](#layout--container)
  - [Class order](#class-order)
  - [Responsive: mobile-first](#responsive-mobile-first)
  - [Dark mode](#dark-mode)
  - [States](#states)
  - [Arbitrary values: sparingly](#arbitrary-values-sparingly)
  - [Units: rem](#units-rem)
  - [Large text blocks: prose](#large-text-blocks-prose)
- [Anti-patterns](#anti-patterns)
- [Tooling](#tooling)

## Documentation

- [Tailwind CSS docs](https://tailwindcss.com/docs) — utilities, `@theme`, v4 configuration
- [Typography plugin](https://github.com/tailwindlabs/tailwindcss-typography) — `prose` classes

## When to use

- Writing or adjusting styling with Tailwind utility classes
- Reviewing or refactoring components for consistent Tailwind usage
- Adding theming, dark mode or responsive design

## Core principles

1. **CSS-first configuration** — no `tailwind.config.js` needed anymore. Configure in CSS via `@theme`:

   ```css
   @import "tailwindcss";

   @theme {
     --color-brand: oklch(0.65 0.2 250);
     --font-display: "Inter", sans-serif;
     --spacing-18: 4.5rem;
   }
   ```

2. **Utility-first** — style in the markup, not with custom CSS. A repeated class set is a missing component, not a missing CSS class: extract it to a Blade partial or component.

3. **No `@apply`, no custom CSS** — there is exactly one accepted exception, and it is not a judgement call: Gravity Forms renders its own markup, so its output is styled centrally in `app.css` using semantic `@theme` tokens (see the Forms section in `AGENTS.md`). Everything else stays in utility classes.

## Conventions

### Colors: oklch

Define every color token in `@theme` in **oklch**, not hex or rgb — this is the format Tailwind v4's own palette uses:

```css
@theme {
  --color-brand: oklch(0.65 0.2 250);
  --color-brand-dark: oklch(0.55 0.2 250);
  --color-surface: oklch(1 0 0);
  --color-error: oklch(0.58 0.22 27);
  --color-success: oklch(0.7 0.17 145);
}
```

Why oklch, in short:

- **Perceptually uniform** — the same lightness step looks like the same step to the eye, so a scale (`-100` … `-900`) can be built by varying `L` while keeping `C`/`H` fixed. With hex you are guessing.
- **Wider gamut** — reaches P3 colors that hex simply cannot express, and degrades gracefully on sRGB screens.
- **Readable intent** — `oklch(L C H)` is lightness, chroma, hue: a hover state is "the same color, a bit darker" (lower `L`), not a new hex value.

Rules of thumb:

- Convert design-tool hex values to oklch once, when defining the token — not on every use
- Keep hue (`H`) constant within one color family; vary lightness (`L`) for the scale and chroma (`C`) for saturation
- Alpha stays available: `oklch(0.65 0.2 250 / 50%)`, or use Tailwind's `/` opacity modifier (`bg-brand/50`)
- Never write a raw color (hex, `rgb()`, oklch or otherwise) in the markup — always through a semantic token

### Base `<html>` element

The `<html>` tag always carries the global base styling so the whole document inherits a consistent font, background, font weight, text color, antialiasing and text-selection style. Set these as Tailwind classes on `<html>`, using semantic theme tokens:

```html
<html class="bg-white font-sans font-normal text-black antialiased selection:bg-black selection:text-white">
```

Always include (the Prettier plugin owns the literal class order):

- **Font family** — `font-sans` (a `--font-*` token)
- **Background** — `bg-white` (a `--color-*` token)
- **Font weight** — `font-normal`
- **Text color** — `text-black` (a `--color-*` token)
- **Antialiasing** — `antialiased`
- **Selection styling** — `selection:bg-black selection:text-white`

Use only semantic tokens from `@theme` here; no loose hex values or raw font names.

### Layout & container

The layout rules are project-wide and live in `AGENTS.md` § Layout — they are not repeated here, so there is one source of truth. In short: one container **per section** (`mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8`), never one container around the whole site; anchor navigation via a hidden anchor div instead of an `id` on the `<section>`; absolute elements positioned with `inset-0` and offset with margins.

`max-w-screen-*` is not part of the v4 max-width scale in the docs — make sure the project defines the matching `--container-*` token in `@theme` so the class resolves:

```css
@theme {
  --container-screen-2xl: 96rem;
}
```

### Class order

Class order is enforced by `prettier-plugin-tailwindcss` — run the plugin and never hand-sort. As a mental model, the order it produces is roughly:

```
layout → flex/grid → spacing → sizing → typography → colors → borders → effects → states/responsive
```

```html
<div class="flex items-center gap-4 p-6 text-sm font-medium text-gray-900 rounded-lg border shadow-sm hover:bg-gray-50">
```

### Responsive: mobile-first

Base classes are mobile; use breakpoint prefixes only to scale up:

```html
<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
```

Never use `sm:` to target "mobile" — that is desktop-first thinking.

### Dark mode

Use `dark:` variants and define colors as semantic theme tokens so you don't have to repeat `dark:` everywhere:

```css
@theme {
  --color-surface: var(--color-white);
}
.dark {
  --color-surface: var(--color-gray-900);
}
```

### States

- Use `hover:`, `focus-visible:` (not `focus:` for keyboard focus), `disabled:`, `aria-*:` and `data-*:` variants
- Group/peer for parent-child interaction: `group-hover:`, `peer-checked:`

### Arbitrary values: sparingly

Always use the existing Tailwind scale when available: `m-20` instead of `m-[80px]`. `w-[347px]` is a code smell — add recurring values to `@theme`. Arbitrary values only for true one-offs without a standard equivalent.

### Units: rem

Use `rem` for all values, not `px` or `em` — consistent with the Tailwind scale and better for accessibility (scales with the root font size):

- Arbitrary values: `m-[5.5rem]`, not `m-[88px]`
- Theme tokens: `--spacing-18: 4.5rem;`, not `72px`

### Large text blocks: prose

For large blocks of (CMS/markdown) text, use the [`@tailwindcss/typography` plugin](https://github.com/tailwindlabs/tailwindcss-typography) instead of styling each element separately:

```html
<article class="prose lg:prose-lg">
  <!-- headings, paragraphs, lists, etc. automatically styled -->
</article>
```

- Customize via `prose` modifiers (`prose-headings:`, `prose-a:`) or theme tokens
- `not-prose` for elements inside the block you style yourself

## Anti-patterns

| Avoid | Do instead |
|---|---|
| `@apply` or custom CSS | Utility classes; extract repetition to a component. Gravity Forms styling in `app.css` is the only exception |
| One container wrapping the whole site | One container per section, so elements can break out full-width |
| `!important` / `!` prefix | Fix specificity conflicts at the source |
| Long unreadable class strings | Fixed class order + Prettier plugin |
| Hardcoded colors (`text-[#3b82f6]`) | Theme tokens (`text-brand`) |
| Hex or `rgb()` in `@theme` (`--color-brand: #3b82f6`) | oklch (`--color-brand: oklch(0.65 0.2 250)`) |
| `style=""` alongside Tailwind | Everything in classes; arbitrary values if needed |
| Styling each element separately in long text blocks | `prose` (typography plugin) |
| Dynamically composed classnames (`text-${color}-500`) | Full classnames, possibly via a map — otherwise purging breaks |

## Tooling

- **Prettier**: `prettier-plugin-tailwindcss` for automatic class sorting — the plugin's order is the source of truth, never hand-sort
- **Conditional classes**: Blade's `@class([...])` directive server-side, Alpine's `:class` binding client-side — this stack has no React/JS component layer, so `clsx`, `cva` and `tailwind-merge` don't belong here
