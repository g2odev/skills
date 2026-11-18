---
name: wordpress-theme
description: 'WordPress theme development conventions and best practices with Roots Sage, Blade and PHP 8.4. Use when building theme templates, View Composers, hooks/filters, custom post types, enqueueing assets, escaping output, translations and WordPress-specific PHP.'
---

# WordPress Theme (Roots Sage)

## Contents

- [Documentation](#documentation)
- [When to use](#when-to-use)
- [Core principles](#core-principles)
- [Conventions](#conventions)
  - [Templates & components](#templates--components)
  - [Naming & comments](#naming--comments)
  - [View Composers](#view-composers)
  - [Queries](#queries)
  - [Images](#images)
  - [SVG icons & logos](#svg-icons--logos)
  - [Security](#security)
  - [Translations](#translations)
  - [Custom post types & fields](#custom-post-types--fields)
  - [Admin defaults](#admin-defaults)
- [Anti-patterns](#anti-patterns)
- [Related skills](#related-skills)

## Documentation

- [Sage docs](https://roots.io/sage/docs/) — theme structure, Blade, View Composers, Acorn
- [WordPress developer](https://developer.wordpress.org/) — hooks, template hierarchy, REST API
- [Blade templates](https://laravel.com/docs/blade) — syntax, components, directives

## When to use

- Building or modifying theme templates, partials and components
- Adding theme logic: View Composers, hooks, filters, custom post types
- Enqueueing assets, handling images, translations or security in themes

## Core principles

1. **Sage structure** — Blade templates in `resources/views/`, logic in `app/`, assets via Vite. Never raw PHP templates in the theme root. The theme's custom stylesheet (`resources/css/app.css` / `custom.scss`) stays as empty as possible: styling lives as Tailwind classes in the Blade markup (existing classes first, arbitrary values otherwise) — in practice only the Gravity Forms styles belong in the stylesheet.
2. **Templates stay dumb** — no queries or business logic in Blade. Prepare data in View Composers (`app/View/Composers/`); templates only display it.
3. **Modern PHP 8.4** — typed properties, return types, enums, `match`, named arguments, constructor property promotion. No legacy array-style config where a typed class works.
4. **Hook into WordPress, never hack it** — use actions/filters in `app/setup.php` or `app/filters.php`; never modify core, plugins or the parent framework.

## Conventions

### Templates & components

- Follow the template hierarchy with Blade equivalents: `index.blade.php`, `page.blade.php`, `single-{cpt}.blade.php`
- Extract repeated markup into partials (`@include('partials.card')`) or components (`<x-button>`)
- Register nav menus in `app/setup.php` (`register_nav_menus(['primary_navigation' => __('Primary Navigation', 'theme-name')])`) and render them in a partial (`partials/navigation.blade.php`) — never hardcode menu items
- The `/styleguide` page (required per project, see `AGENTS.md`) is a Blade page template: `resources/views/template-styleguide.blade.php` with a `{{-- Template Name: Styleguide --}}` header
- Escape all output: `{{ $var }}` escapes automatically in Blade and is the default for untrusted values; `{!! !!}` is for trusted content — sanitized HTML (`wp_kses_post()`), WP titles (`get_the_title()` returns texturized entities that `{{ }}` would double-escape) and ACF element fields (trusted editor content, see the [acf-elements](../acf-elements/SKILL.md) skill). URLs and attributes always go through `esc_url()` / `esc_attr()`
- Never wrap an escaping function in `{{ }}`: `{{ esc_url($url) }}` escapes twice, which breaks a URL containing `&`. Echo it with `{!! esc_url($url) !!}` / `{!! esc_attr($value) !!}`
- Never mix the two `@php` forms in one file: an inline `@php(...)` followed by a `@php … @endphp` block throws a ParseError. In partials, use only the inline form
- Always check a variable is set and not empty before outputting it:

```blade
{{-- WP titles carry texturized entities — echo unescaped to avoid double-escaping --}}
@if(!empty($heroTitle))
  <h1>{!! $heroTitle !!}</h1>
@endif

{{-- Rich content is rendered unescaped only after sanitizing --}}
@if(!empty($heroContent))
  <div class="prose">{!! wp_kses_post($heroContent) !!}</div>
@endif
```

### Naming & comments

- Follow Laravel/PSR conventions for naming: `camelCase` for variables (`$heroTitle`) and methods, `StudlyCaps` for classes — consistent with Sage/Acorn; WordPress core functions keep their own `snake_case` names
- View files are kebab-case (`partials/key-figures.blade.php`); ACF field and layout names are snake_case (`key_figures`) — see the [acf-elements](../acf-elements/SKILL.md) skill
- Prefix all custom functions in `functions.php` (and other global functions) with `g2o_` (e.g. `g2o_get_hero_image()`) to avoid collisions with WordPress core and plugins
- Always wrap function definitions in a `function_exists()` check, and place a section comment block above each function:

```php
/*
|--------------------------------------------------------------------------
|  Add ACF Options Page
|--------------------------------------------------------------------------
*/
if (!function_exists('g2o_add_options_page')) {
    function g2o_add_options_page(): void
    {
        // ...
    }
}
```

- Write comments in English: full sentences with punctuation, `//` for inline comments, docblocks (`/** */`) for functions, classes and hooks

### View Composers

```php
namespace App\View\Composers;

use Roots\Acorn\View\Composer;

class Hero extends Composer
{
    protected static $views = ['partials.hero'];

    public function with(): array
    {
        return [
            'title' => get_the_title(),
            'image' => get_post_thumbnail_id(),
        ];
    }
}
```

- Pass data to the view through `with()`. A public method on the composer reaches the view as an object, so `!empty()` and array access on it do not work.

### Queries

- Use `WP_Query` with specific arguments; always set `no_found_rows => true` when pagination is not needed
- Never use `query_posts()`; reset with `wp_reset_postdata()` after custom loops
- Cache expensive queries with transients (`get_transient`/`set_transient`)

### Images

- Output via `wp_get_attachment_image()` — generates `srcset`, `sizes`, lazy loading and dimensions automatically

### SVG icons & logos

Every SVG used in the theme — icons and logos — lives in one Blade component: `resources/views/components/svg-icon.blade.php`. Never paste inline `<svg>` markup into partials and never scatter loose `.svg` files; add a `@case` to the component instead:

```blade
@props(['name'])

@switch($name)
  @case('icon-bloem')
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 96 96" {{ $attributes }}>
      <path fill="currentColor" d="M87.712 48.29a17.5 17.5 0 0 0 2.954-2.157 17.2 17.2 0 0 0 4.932-15.891…" />
    </svg>
  @break

  @case('icon-chat')
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 96 96" {{ $attributes }}>
      <path fill="currentColor" d="M84.267 0C90.775 0 96 4.91 96 10.988v55.146c0 6.08-5.225 10.987-11.733 10.987…" />
    </svg>
  @break
@endswitch
```

Usage — size, color and ARIA come from the caller:

```blade
<x-svg-icon name="icon-chat" class="size-6 text-brand" aria-hidden="true" />
```

- **`{{ $attributes }}` on the `<svg>` root** — callers pass Tailwind classes and `aria-*` attributes; the component itself carries none
- **`fill="currentColor"` on the paths** (`fill="none"` on the root) — the icon follows the text color, so `text-*` utilities style it
- **No hardcoded `width`/`height`** — keep the `viewBox` and size with `size-*` (square) or `h-*`/`w-*` (non-square) classes
- **Prefix names by kind**: `icon-*` for icons, `logo-*` for logos
- Decorative icons get `aria-hidden="true"` at the call site

### Security

- Escape late: `esc_html()`, `esc_attr()`, `esc_url()` at output (or Blade's `{{ }}`) — one or the other, never both: an `esc_*()` call is echoed with `{!! !!}`
- Sanitize early: `sanitize_text_field()`, `wp_kses_post()` on input
- Verify nonces and capabilities (`current_user_can()`) for any form or admin action

### Translations

- All strings translatable: `__('Read more', 'theme-name')`, `_x()`, `_n()`
- **Exactly one text domain per theme** — the theme's folder name (`wp-content/themes/<folder>` → `__('…', '<folder>')`), used verbatim in every translation call. Never a second domain in the same project; a mixed domain means those strings silently fall out of the theme's `.po`/`.mo` files
- No hardcoded strings in templates, including `aria-label`s

### Custom post types & fields

- Register CPTs and taxonomies in `app/setup.php` (or a dedicated `app/PostTypes/` class per CPT)
- Prefer ACF (or native meta) exposed through View Composers, so templates never call `get_field()` directly

### Admin defaults

- **ACF fields above Yoast SEO**: give the Yoast metabox a low priority in `app/filters.php`, so the ACF field groups sit at the top of the edit screen:

```php
add_filter('wpseo_metabox_prio', fn (): string => 'low');
```

## Anti-patterns

| Avoid | Do instead |
|---|---|
| Queries or logic in Blade templates | View Composers |
| `query_posts()` | `WP_Query` + `wp_reset_postdata()` |
| Echoing untrusted output unescaped | `{{ }}` / `esc_html()` / `esc_attr()` |
| Double escaping (`{{ esc_url($url) }}`) | `{!! esc_url($url) !!}` / `{!! esc_attr($value) !!}` |
| Inline `@php(...)` mixed with a `@php … @endphp` block | Only the inline form in partials |
| Composer data through public methods | Return it from `with()` |
| `get_field()` scattered in templates | Map fields in a View Composer |
| Modifying core/plugin files | Actions and filters |
| Hardcoded strings | `__('...', 'theme-name')` |
| More than one text domain in a theme | One domain — the theme folder name — in every translation call |
| Manual `<img>` tags for attachments | `wp_get_attachment_image()` |
| Inline `<svg>` pasted into partials, or loose `.svg` files | A `@case` in the `<x-svg-icon>` component |

## Related skills

- [acf-elements](../acf-elements/SKILL.md) — ACF flexible-content elements via local JSON. Flexible content is the one exception to "no `get_field()` in templates": sub-fields only exist inside the `have_rows()` loop, so each element partial reads its own `get_sub_field('elementN')`.
- [tailwindcss](../tailwindcss/SKILL.md) — styling the Blade markup produced here.
