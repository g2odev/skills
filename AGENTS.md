# AGENTS.md

## Tech stack

Always build with the following stack:

- **Styling**: Tailwind CSS — no custom CSS or other CSS frameworks
- **Interactivity**: Alpine.js — no jQuery, React or Vue
- **Animations**: GSAP — no CSS animations or other animation libraries
- **Target platform**: most projects ship as WordPress themes on PHP 8.4 — write modern PHP (typed properties, enums, match, named arguments)
- **Theme framework**: themes are based on [Roots Sage](https://roots.io/sage/) — Blade templates for all markup, Composer for PHP dependencies, Vite for the asset pipeline

## Documentation

- [Tailwind CSS](https://tailwindcss.com/docs) — utilities, `@theme`, v4 configuration
- [Alpine.js](https://alpinejs.dev/start-here) — directives, magics, plugins
- [GSAP](https://gsap.com/docs/v3/) — tweens, timelines, ScrollTrigger
- [Sage](https://roots.io/sage/docs/) — theme structure, Blade, View Composers, Acorn
- [WordPress](https://developer.wordpress.org/) — hooks, template hierarchy, REST API
- [Gravity Forms](https://docs.gravityforms.com/) — hooks and filters (`gform_*`)

## Skills

Detailed conventions and best practices per tool:

- [tailwindcss](.github/skills/tailwindcss/SKILL.md) — Tailwind CSS v4 conventions and best practices
- [alpinejs](.github/skills/alpinejs/SKILL.md) — Alpine.js conventions and best practices
- [gsap](.github/skills/gsap/SKILL.md) — GSAP animation conventions and best practices
- [wordpress-theme](.github/skills/wordpress-theme/SKILL.md) — WordPress theme development with Sage, Blade and PHP 8.4
- [acf-elements](.github/skills/acf-elements/SKILL.md) — ACF flexible-content elements made manageable via local JSON, generated from Blade partials

## Rules

These are the always-on rules; the skills above hold the detail and the examples. Where a skill and this file disagree, **this file wins** — and the skill should be corrected.

- Always use the latest (stable) versions of Tailwind CSS, Alpine.js and GSAP
- Use Tailwind utility classes in the markup; avoid `@apply` and inline styles
- Prefer existing Tailwind classes; avoid arbitrary values (`m-[80px]`) when a standard class exists (`m-20`). When a design value is within a few pixels of a standard class, snap to that class; only when it deviates too much, use an arbitrary (rem) value
- Use `rem` for values, not `px` or `em` — just like the Tailwind scale itself (e.g. `m-[5.5rem]` instead of `m-[88px]`, and rem values in `@theme`)
- Use Alpine.js directives (`x-data`, `x-on`, `x-show`, ...) for UI behavior
- Use GSAP (with ScrollTrigger where needed) for all animations and transitions
- Target GSAP animations via `data-*` attributes (`[data-animate="hero-title"]`), never via HTML classes — classes are for styling
- Define colors and fonts as semantic tokens in `@theme`; no loose hex values or font names in the markup
- Define color tokens in `oklch` (`--color-brand: oklch(0.65 0.2 250)`), not hex or `rgb()` — perceptually uniform, wider gamut, and the format Tailwind v4 itself uses
- Tailwind class order is enforced by `prettier-plugin-tailwindcss`
- Use `prose` (`@tailwindcss/typography`) for large text blocks such as CMS or markdown content
- Always write comments in English
- Use English naming for files, folders, variables and functions
- Use Blade templates (`resources/views/`) for all theme markup; extract repeated markup into Blade partials/components (`@include`, `<x-component>`)
- Put theme logic in `app/` (View Composers, filters, setup) — keep Blade templates free of business logic

## Commands

Adjust per project; typical Sage setup:

- `npm run dev` — start the Vite dev server
- `npm run build` — production build
- `composer install` — install PHP dependencies
- `wp acorn view:clear` — clear compiled Blade views

## File structure

Sage conventions:

- Blade templates in `resources/views/`, partials in `resources/views/partials/`, components in `resources/views/components/`
- Any custom JS in a single `resources/js/app.js`, any custom CSS in a single `resources/css/app.css`
- PHP theme logic in `app/` (e.g. `setup.php`, `filters.php`, View Composers in `app/View/Composers/`)
- The preference always remains: solve it with Tailwind classes or Alpine directives in the markup; use `app.js`/`app.css` only when there is no other way

## Layout

- The container is always `mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8` — one pattern, used everywhere
- Place the container inside a section/element, never wrap the entire website in one container. This keeps the freedom for elements (heroes, images, backgrounds) to break out of the container full-width:

```html
<!-- Good: container per section -->
<section class="bg-gray-100">
  <div class="mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8">...</div>
</section>

<!-- Bad: entire site inside one container -->
<div class="mx-auto max-w-screen-2xl px-4">
  <section>...</section>
  <section>...</section>
</div>
```

- Do not put an `id` on the `<section>` itself for anchor navigation; use a hidden zero-height div as scroll anchor. This lets you correct the scroll position (e.g. for a sticky header) without affecting the section layout:

```html
<section class="relative">
  <div id="about" class="invisible absolute inset-x-0 top-0 -mt-24 opacity-0 pointer-events-none" aria-hidden="true"></div>
  <div class="mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8">...</div>
</section>
```

In a WordPress theme this anchor div is the `<x-element-id>` component — see the [acf-elements](.github/skills/acf-elements/SKILL.md) skill.

- Position `absolute` elements with `inset-0` (or `inset-x-0`/`inset-y-0`) and offset further with margins (`-mt-24`, `ml-8`); do not use offset values on `top`/`left`/`bottom`/`right` themselves:

```html
<!-- Good: inset to zero, offset with margin -->
<div class="absolute inset-0 -mt-24"></div>

<!-- Bad: offset via inset values -->
<div class="absolute -top-24 left-0"></div>
```

## Responsive

- Always mobile-first: base classes are for mobile; use `md:`/`lg:` only to scale up
- Never use `sm:` to target "mobile" — that is desktop-first thinking

## Styleguide

- Every project includes a styleguide page (e.g. `/styleguide`) showcasing all base elements:
  - Titles/headings (`h1`–`h6`)
  - Colors (all semantic tokens from `@theme`)
  - Text styles (paragraphs, links, lists, `prose` blocks)
  - Buttons (variants and states: hover, focus-visible, disabled)
  - Form elements (inputs, selects, checkboxes, radios, validation states)
- Build new components on the styleguide first; this keeps the design system consistent and reusable

## Forms (Gravity Forms)

Forms are converted to Gravity Forms, which renders its own markup — you cannot add Tailwind classes to it directly. Therefore:

- Disable the Gravity Forms default styling (`gform_disable_css`) and style from scratch
- Style Gravity Forms output centrally in `app.css` by targeting the GF classes (`.gform_wrapper`, `.gfield`, `.gform_button`, ...) — this is the accepted exception to the "no custom CSS" rule
- Use only semantic tokens from `@theme` in these styles; keep them in sync with the form elements on the styleguide
- Build form styles on the styleguide first with native elements, mirroring the Gravity Forms markup structure, so the styleguide and GF forms always match
- Where possible, add custom CSS classes per field/button in the Gravity Forms editor (or via the `gform_field_css_class` filter) instead of writing extra selectors
- Style error and success states explicitly, using semantic tokens (e.g. `--color-error`, `--color-success`) from `@theme`:
  - Field validation errors (`.gfield_error`, `.validation_message`)
  - The form-level error summary (`.gform_validation_errors`)
  - The confirmation/success message (`.gform_confirmation_message`)

## Accessibility

- Use semantic HTML: `<nav>`, `<section>`, `<header>`, `<button>` — no `<div @click>` for clickable elements
- Use `focus-visible:` states on all interactive elements
- Bind ARIA attributes to Alpine state: `:aria-expanded="open"`
- Respect `prefers-reduced-motion` in GSAP animations via `gsap.matchMedia()`

## Images

- Always include an `alt` attribute (empty `alt=""` for decorative images)
- `loading="lazy"` for images below the fold
- Modern formats (WebP/AVIF) where possible
- Set `width` and `height` to prevent layout shift
- In WordPress, output images via `wp_get_attachment_image()` — it generates `srcset`, `sizes`, lazy loading and dimensions automatically
