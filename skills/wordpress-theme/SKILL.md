---
name: wordpress-theme
description: 'WordPress theme conventions with Roots Sage, Blade and PHP 8.4: dumb templates, View Composers, late escaping, one text domain. Use whenever touching theme PHP or Blade. Trigger on: .blade.php, resources/views, partials, Blade components (x-button, x-svg-icon), @include, @php, app/setup.php, app/filters.php, View Composers, add_action/add_filter, WP_Query, custom post types, taxonomies, nav menus, enqueueing assets, wp_get_attachment_image, SVG icons and logos, esc_url/esc_attr/wp_kses_post, escaped versus unescaped Blade echo, translations and text domain, functions.php, g2o_ functions, the styleguide template, Yoast metabox order. Do NOT trigger for: ACF flexible-content elements and acf-json (acf-elements), Tailwind class choices (tailwindcss), plugin or WordPress core development.'
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
  - [wp-admin](#wp-admin)
  - [External feeds & integrations](#external-feeds--integrations)
  - [Admin defaults](#admin-defaults)
- [Anti-patterns](#anti-patterns)
- [Output checklist](#output-checklist)
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
- **Content without a page of its own is a non-public post type**: team members, frequently asked questions, anything an element picks and shows. `'public' => false, 'show_ui' => true, 'rewrite' => false, 'has_archive' => false` — no URL, not in the site search, not in the sitemap, so nothing to keep out of Google by hand. Use the native title and editor where they fit (FAQ: title = question, content = answer; `'show_in_rest' => false` gives the plain editor for short texts)
- The detail page of a public post type is managed entirely on the post: it renders the post's own elements, never defaults appended from an options page. The overview page per post type (breadcrumb, back link) is one setting under Opties > Algemeen — see [acf-elements](../acf-elements/SKILL.md)
- An admin list of a post type that has a status or source worth seeing gets a column and a filter (`manage_{cpt}_posts_columns`, `restrict_manage_posts` + `pre_get_posts`); a short marker behind the title goes through `display_post_states`, never by changing the title

### wp-admin

- **Structure**: hook files (`app/hooks/*.php`, `app/setup.php`, `app/filters.php`) only register hooks. An admin page is a class that turns data into plain arrays plus a Blade view under `resources/views/admin/`; no markup in closures. Classes do one thing (fetching, mapping fields, media, mail, the log, the page) — a class that grows past a few hundred lines is doing several
- **Markup**: wp-admin has no Tailwind. Use the admin's own classes (`wrap`, `wp-heading-inline`, `page-title-action`, `wp-list-table widefat striped`, `tablenav`, `subsubsub`, `postbox`, `notice notice-*`); a few inline styles are acceptable there and only there
- **Notices are short**: what is wrong in a few words, plus a link — `Aanbod wordt niet bijgewerkt: IP-adres geweigerd (203.0.113.7). Bekijk logboek`. The explanation and what to do about it belong on the page the link opens, not in the notice. Show a blocking problem on every admin page, the rest only on the screens it concerns
- **Actions**: state-changing links and forms go through `admin-post.php` with a capability check and a nonce. `wp_nonce_url()` returns an HTML-escaped URL — escaping it again in a template (`{{ }}`, `esc_url()` on top) turns `&` into `&amp;amp;` and WordPress answers "De link die je volgde is verlopen". Build a plain URL (`add_query_arg(['action' => …, '_wpnonce' => wp_create_nonce(…)], admin_url('admin-post.php'))`) and escape once, at output
- **Request values** are only used when they are plain strings that match a known option: `is_string($_GET['status'] ?? null) && isset($options[$_GET['status']])`. `isset($options[$_GET['status']])` alone is a fatal error when the value is an array
- **Test the button on every page it is on.** A link that works above one list can be broken on another page that renders it through a different template

### External feeds & integrations

A link with an external system (a property feed, a CRM) follows the same rules every time:

- **On when the key is there.** The API key lives in `.env` and is exposed as a constant in `config/application.php`; the integration is active as soon as the key is set — no on/off switch in the admin. The key never goes into the database, a log, a notice or a mail (redact it from error messages), and requests to the service do not follow redirects
- **External wins, internal stays.** Imported posts carry a meta id of the source (`_<prefix>_id`) and are overwritten by every sync; posts made in WordPress are never touched. Show the difference with a post state ("Intern" / "Extern") and a notice on the edit screen of an imported post
- **A failure changes nothing.** The content already on the site stays online; a post that leaves the feed becomes a draft, nothing is deleted. An empty answer only takes everything offline when it repeats. The public site says that the content cannot be shown only when there is nothing to fall back on
- **Typed errors**: an enum with one case per cause (not configured, IP refused, key refused, unavailable, unreadable answer, cron not running), each with a short title for lists and notices and a full sentence for the detail page
- **A log page** under the post type: one row per sync (title = date and what happened, started by, status), a detail page per row with what was added, updated (and what changed), removed and skipped, filters, paging and delete. Cap the number of rows
- **Mail once per outage** to the developers and the site admin — when the link breaks or the cause changes, not on every attempt — and write a row in the log that says the mail went out and to whom
- **Watch the schedule**: WP-Cron only runs on visits. Detect an overdue event, report it like any other failure, and offer a WP-CLI command for a server cron
- **One sync at a time**: lock with `add_option()` (an atomic insert), not a transient, and take over a lock that is older than the longest possible run
- **Media is downloaded, not hotlinked**: only from the service's own host, over https, with a size limit and a check of the real file type; stored as WebP with a name of our own (`<slug>-01.webp`), attached to the post, removed when it leaves the feed, and hidden from the media library (`ajax_query_attachments_args`, `pre_get_posts` on the `upload` screen) so editors cannot reuse or delete it
- **Text from the feed** passes `wp_kses()` with a short allow-list (paragraphs, emphasis, lists, links) — not `wp_kses_post()`
- **XML** is parsed with `LIBXML_NONET` and refused when it holds a `DOCTYPE` or `ENTITY`
- **Removed posts need a redirect**: the log asks for one and names the old URL; making it stays a manual job
- **Test with the real response and with mocked ones** (`pre_http_request`): changed, removed, back again, refused, unreachable, malformed, hostile content. Field mappings built from documentation alone are wrong somewhere — compare one real object field by field as soon as there is one

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
| A public post type for content that has no page (team, FAQ) | `'public' => false, 'show_ui' => true` |
| Markup echoed from a hook closure, or one class that fetches, maps, mails and renders | A class per job; admin markup in a Blade view under `resources/views/admin/` |
| A long admin notice that explains the cause and the cure | A few words and a link to the page with the details |
| `{{ wp_nonce_url(…) }}` or `esc_url(wp_nonce_url(…))` | A plain URL with `wp_create_nonce()`, escaped once |
| An API key in the database or an on/off option for an integration | The key in `.env`; a key present means on |
| Hotlinking media from an external service | Download, convert, rename, attach, hide from the media library |
| Deleting or migrating stored content without a copy | `wp db export` first; write, read back, then delete |

## Output checklist

Verify every item before delivering:

- [ ] Markup is Blade in `resources/views/`; logic is in `app/` — no queries or business logic in a template
- [ ] View data comes from a View Composer through `with()`; no `get_field()` in templates (flexible-content partials are the exception, see acf-elements)
- [ ] Every output is guarded with `!empty()` before it is echoed
- [ ] Untrusted values are echoed with `{{ }}`; trusted content (sanitized HTML, WP titles, ACF element fields) with `{!! !!}`
- [ ] `esc_url()` / `esc_attr()` are echoed with `{!! !!}`, never wrapped in `{{ }}`; rich content goes through `wp_kses_post()`
- [ ] Only the inline `@php(...)` form in partials — never mixed with a `@php … @endphp` block
- [ ] Global functions are prefixed `g2o_`, wrapped in `function_exists()` and carry a section comment block; PHP is typed (parameter and return types)
- [ ] Variables and methods camelCase, classes StudlyCaps, view files kebab-case
- [ ] Every string is translatable and uses the one text domain of the theme (its folder name), `aria-label`s included
- [ ] Attachments are rendered with `wp_get_attachment_image()`; SVGs are a `@case` in `<x-svg-icon>`, never inline
- [ ] Custom loops use `WP_Query` (with `no_found_rows` when not paginating) and end with `wp_reset_postdata()`
- [ ] WordPress is extended through actions/filters in `app/` — no edits to core, plugins or the framework
- [ ] Comments are in English

## Related skills

- [acf-elements](../acf-elements/SKILL.md) — ACF flexible-content elements via local JSON. Flexible content is the one exception to "no `get_field()` in templates": sub-fields only exist inside the `have_rows()` loop, so each element partial reads its own `get_sub_field('elementN')`.
- [tailwindcss](../tailwindcss/SKILL.md) — styling the Blade markup produced here.
