---
name: wordpress-theme
description: 'WordPress theme development conventions and best practices with Roots Sage, Blade and PHP 8.4. Use when building theme templates, View Composers, hooks/filters, custom post types, enqueueing assets, escaping output, translations and WordPress-specific PHP.'
---

# WordPress Theme (Roots Sage)

## Documentation

- [Sage docs](https://roots.io/sage/docs/) — theme structure, Blade, View Composers, Acorn
- [WordPress developer](https://developer.wordpress.org/) — hooks, template hierarchy, REST API
- [Blade templates](https://laravel.com/docs/blade) — syntax, components, directives

## When to use

- Building or modifying theme templates, partials and components
- Adding theme logic: View Composers, hooks, filters, custom post types
- Enqueueing assets, handling images, translations or security in themes

## Core principles

1. **Sage structure** — Blade templates in `resources/views/`, logic in `app/`, assets via Vite. Never raw PHP templates in the theme root.
2. **Templates stay dumb** — no queries or business logic in Blade. Prepare data in View Composers (`app/View/Composers/`); templates only display it.
3. **Modern PHP 8.4** — typed properties, return types, enums, `match`, named arguments, constructor property promotion. No legacy array-style config where a typed class works.
4. **Hook into WordPress, never hack it** — use actions/filters in `app/setup.php` or `app/filters.php`; never modify core, plugins or the parent framework.

## Conventions

### Templates & components

- Follow the template hierarchy with Blade equivalents: `index.blade.php`, `page.blade.php`, `single-{cpt}.blade.php`
- Extract repeated markup into partials (`@include('partials.card')`) or components (`<x-button>`)
- Escape all output: `{{ $var }}` escapes automatically in Blade; use `{!! !!}` only for trusted, sanitized HTML (e.g. `wp_kses_post()`)
- Always check a variable is set and not empty before outputting it:

```blade
@if(!empty($heroTitle))
  <h1>{!! $heroTitle !!}</h1>
@endif
```

### Naming & comments

- Follow Laravel/PSR conventions for naming: `camelCase` for variables (`$heroTitle`) and methods, `StudlyCaps` for classes — consistent with Sage/Acorn; WordPress core functions keep their own `snake_case` names
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

### Queries

- Use `WP_Query` with specific arguments; always set `no_found_rows => true` when pagination is not needed
- Never use `query_posts()`; reset with `wp_reset_postdata()` after custom loops
- Cache expensive queries with transients (`get_transient`/`set_transient`)

### Images

- Output via `wp_get_attachment_image()` — generates `srcset`, `sizes`, lazy loading and dimensions automatically

### Security

- Escape late: `esc_html()`, `esc_attr()`, `esc_url()` at output (or Blade's `{{ }}`)
- Sanitize early: `sanitize_text_field()`, `wp_kses_post()` on input
- Verify nonces and capabilities (`current_user_can()`) for any form or admin action

### Translations

- All strings translatable: `__('Read more', 'theme-name')`, `_x()`, `_n()` — use the theme name as text domain, consistently everywhere
- No hardcoded strings in templates, including `aria-label`s

### Custom post types & fields

- Register CPTs and taxonomies in `app/setup.php` (or a dedicated `app/PostTypes/` class per CPT)
- Prefer ACF (or native meta) exposed through View Composers, so templates never call `get_field()` directly

## Anti-patterns

| Avoid | Do instead |
|---|---|
| Queries or logic in Blade templates | View Composers |
| `query_posts()` | `WP_Query` + `wp_reset_postdata()` |
| Echoing unescaped output | `{{ }}` / `esc_html()` / `esc_attr()` |
| `get_field()` scattered in templates | Map fields in a View Composer |
| Modifying core/plugin files | Actions and filters |
| Hardcoded strings | `__('...', 'themename')` |
| Manual `<img>` tags for attachments | `wp_get_attachment_image()` |
