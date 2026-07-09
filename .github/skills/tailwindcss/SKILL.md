---
name: tailwindcss
description: 'Tailwind CSS v4 conventions and best practices. Use when writing, reviewing or refactoring Tailwind classes, styling components, theming with CSS variables, responsive design, dark mode and setting up a consistent utility-first design system.'
---

# Tailwind CSS

## Documentation

- [Tailwind CSS docs](https://tailwindcss.com/docs) — utilities, `@theme`, v4 configuration
- [Typography plugin](https://github.com/tailwindlabs/tailwindcss-typography) — `prose` classes

## When to use

- Writing or adjusting styling with Tailwind utility classes
- Reviewing or refactoring components for consistent Tailwind usage
- Adding theming, dark mode or responsive design

## Core principles (v4)

1. **CSS-first configuration** — no `tailwind.config.js` needed anymore. Configure in CSS via `@theme`:

   ```css
   @import "tailwindcss";

   @theme {
     --color-brand: oklch(0.65 0.2 250);
     --font-display: "Inter", sans-serif;
     --spacing-18: 4.5rem;
   }
   ```

2. **Utility-first** — style in the markup, not with custom CSS. Only create a component class (`@utility`) when a pattern is repeated literally 3+ times and can't be captured in a component.

3. **Reuse via components, not via `@apply`** — extract repeated class sets to a (framework) component. Avoid `@apply` except for small, truly global base elements.

## Conventions

### Class order

Keep a fixed order (enforceable with `prettier-plugin-tailwindcss`):

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
| `@apply` everywhere | Extract to components |
| `!important` / `!` prefix | Fix specificity conflicts at the source |
| Long unreadable class strings | Fixed class order + Prettier plugin |
| Hardcoded colors (`text-[#3b82f6]`) | Theme tokens (`text-brand`) |
| `style=""` alongside Tailwind | Everything in classes; arbitrary values if needed |
| Styling each element separately in long text blocks | `prose` (typography plugin) |
| Dynamically composed classnames (`text-${color}-500`) | Full classnames, possibly via a map — otherwise purging breaks |

## Tooling

- **Prettier**: `prettier-plugin-tailwindcss` for automatic class sorting
- **Conditional classes**: `clsx` or `cva` (class-variance-authority) for variants
- **Merge conflicts**: `tailwind-merge` when combining prop classes
