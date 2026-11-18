---
name: acf-elements
description: 'Make Blade elements manageable through ACF flexible content using local JSON: one clone-only field group per element, one layout in the central "Elementen" group and one Blade partial, all sharing one name. Use whenever adding, editing or generating a content element. Trigger on: acf-json, group_*.json, field group, flexible content, layout, clone field, get_sub_field, have_rows, get_row_layout, partials/elements.blade.php, an elementN partial, data-element, x-element-id, repeater or group fields, ACF instructions and placeholders ("Bijvoorbeeld: …"), g2o_populate_ fields, making a design block editable in wp-admin, a new page block or element. Do NOT trigger for: general theme PHP and Blade conventions (wordpress-theme), styling the markup (tailwindcss), animating elements (gsap).'
---

# ACF Elements (flexible content via local JSON)

## Contents

- [Documentation](#documentation)
- [When to use](#when-to-use)
- [Core principles](#core-principles)
- [Conventions](#conventions)
  - [Workflow — generate ACF JSON from a Blade element](#workflow--generate-acf-json-from-a-blade-element)
  - [Field-type mapping](#field-type-mapping)
  - [Field patterns](#field-patterns)
  - [Shared content & existing content](#shared-content--existing-content)
  - [Editor helpers](#editor-helpers)
  - [Descriptions & placeholders](#descriptions--placeholders)
  - [Keys](#keys)
  - [JSON field & group defaults](#json-field--group-defaults)
  - [Naming, language & escaping](#naming-language--escaping)
- [Reference files](#reference-files)
- [Anti-patterns](#anti-patterns)
- [Output checklist](#output-checklist)
- [Related skills](#related-skills)

## Documentation

- [ACF resources](https://www.advancedcustomfields.com/resources/) — full field, function and feature reference
- [ACF Local JSON](https://www.advancedcustomfields.com/resources/local-json/) — auto-sync field groups from `acf-json/`
- [ACF Flexible Content](https://www.advancedcustomfields.com/resources/flexible-content/) — layouts and `have_rows()`
- [ACF Clone field](https://www.advancedcustomfields.com/resources/clone/) — reuse a group inside a layout
- [Sage Blade](https://roots.io/sage/docs/blade-templates/) — the partial that renders each element

## When to use

- Adding a new manageable content block (a new `elementN`) to a page/post
- Regenerating or fixing the ACF JSON for an existing Blade element partial
- Understanding how the "Elementen" flexible content maps to `resources/views/partials/`

## Core principles

Every element is built from **three parts that share one name** (`elementN`), each written in the case its side expects:

| Part | Case | Single word | Multiple words |
|---|---|---|---|
| ACF layout `name` and field `name` | snake_case | `element1` | `key_figures` |
| View file | kebab-case | `partials/element1.blade.php` | `partials/key-figures.blade.php` |
| Blade variable | camelCase | `$element1` | `$keyFigures` |

For a single-word name the three are identical; for a name of several words they are the same name in a different case — never literally the same string.

```
acf-json/group_<uid>.json          →  layout "elementN"              →  partials/elementN.blade.php
standalone group, field: elementN     in flexible content "elements"    get_sub_field('elementN')
(active: false)                       (single clone field)              rendered by get_row_layout()
```

1. **Standalone ACF group** (`acf-json/group_*.json`) — one `group` field named `elementN` holding the actual sub-fields (label, title, content, …). Set `"active": false`: it only exists to be cloned, it must not render its own metabox.
2. **Central "Elementen" flexible content** (`field: elements`) — one `layout` per element whose `name` is the partial name in snake_case (`key_figures` → `partials/key-figures.blade.php`). Each layout contains a single **clone** field that clones the standalone group with `"display": "seamless"` and `"prefix_name": 0`, so the group stays reachable as `get_sub_field('elementN')`.
3. **Blade partial** (`partials/elementN.blade.php`) — read once into `$elementN = get_sub_field('elementN')`, then render each sub-field guarded by `!empty()`.

The loop is generic — `partials/elements.blade.php` walks the rows, turns the snake_case layout name into the kebab-case view name and includes that partial:

```blade
{{-- partials/elements.blade.php --}}
@while (have_rows('elements'))
  @php(the_row())
  @php($elementView = 'partials.' . str_replace('_', '-', get_row_layout()))
  @if (View::exists($elementView))
    @include($elementView)
  @endif
@endwhile
```

```blade
{{-- partials/key-figures.blade.php --}}
@php($keyFigures = get_sub_field('key_figures'))
```

Page templates include this partial once — `@include('partials.elements')` — wherever the flexible content should render.

Because the three names map onto each other, adding an element is purely additive: no central registry to edit by hand.

**Which name an element gets.** A general-purpose block — text with an image, a content block, anything without a fixed meaning — is numbered: label `Element #1`, name `element1`. An element with one specific purpose gets a descriptive English name and a Dutch label (`usp` / "USP's", `key_figures` / "Kerncijfers", `publications` / "Publicaties"). When in doubt whether a block is generic, ask — it is the client's call.

**Detail pages own their elements.** The detail page of a custom post type renders only the elements of the post itself (`single-{cpt}.blade.php` → `@include('partials.elements')`). Never append "default elements" from an options page to every detail page. Per-post-type settings such as the overview page (for the breadcrumb and back link) live under *Opties > Algemeen*, not on an options sub-page per post type.

> **Why `get_sub_field()` lives in the partial.** The [wordpress-theme](../wordpress-theme/SKILL.md) skill says to map fields in a View Composer instead of scattering `get_field()` through templates. Flexible content is the exception: sub-fields only exist inside the `have_rows()` / `the_row()` loop, so they cannot be resolved up-front in a composer. One `get_sub_field('elementN')` at the top of the partial is the intended pattern — everything below it reads the resulting array, not ACF.

> **Theme components.** The templates in [references/templates.md](references/templates.md) use `<x-element-id>` (scroll anchor), `<x-button>` and `<x-svg-icon>` from `resources/views/components/`. They are project components, not part of ACF — swap them for whatever your theme provides.

## Conventions

### Workflow — generate ACF JSON from a Blade element

Given an element partial (e.g. `element1.blade.php`), generate its field group. Read [references/templates.md](references/templates.md) before writing any JSON or a new partial — it holds the complete, copyable templates:

1. **Read the partial** and collect every sub-field access `$elementN['<name>']` plus any `have_rows('<name>')` (a repeater) inside it. The array key = the ACF field `name`.
2. **Infer the ACF type** for each key from how it is used — see [Field-type mapping](#field-type-mapping).
3. **Bundle related fields into a `group`** when they form one logical unit: an element that shows a contact person gets a single `contact` group holding the contact details and image, not five loose fields. Give the group its own `instructions` describing the unit (`Voeg de gegevens van de contactpersoon toe.`) — see [Grouping related fields](references/field-patterns.md#grouping-related-fields).
4. **Write a description and placeholder for every field**: give each field a clear `instructions` sentence, and — for every field type that supports it (text, textarea, email, url, number, password) — a `placeholder` starting with `Bijvoorbeeld: "…"`. **The placeholder text is the content the HTML element holds in the partial** (the example copy from the design): `<h2>Met de juiste kennis vooruit</h2>` becomes `"placeholder": "Bijvoorbeeld: \"Met de juiste kennis vooruit\""`. See [Descriptions & placeholders](#descriptions--placeholders).
5. **Build the standalone group JSON** (`acf-json/group_<uid>.json`) from the [standalone group template](references/templates.md#standalone-group--acf-jsongroup_uidjson): a single `group` field named `elementN` whose `sub_fields` are the inferred fields, in the order they appear in the markup. `"active": false`.
6. **Register the layout** in the central "Elementen" flexible content group, using the [layout entry template](references/templates.md#layout-entry--inside-the-elementen-flexible-content-group): add a `layout` named `elementN` containing one `clone` field that clones the new group (`"display": "seamless"`, `"prefix_label": 0`, `"prefix_name": 0`).
7. **Keys**: give every `key`, `group`, `layout` a fresh unique id (see [Keys](#keys)). Never reuse a key from another element.
8. **Sync**: open Custom Fields → Field Groups in wp-admin and run *Sync available* — ACF picks up the new/changed JSON from `acf-json/` by itself. There is no built-in WP-CLI sync command; only use one if the project ships its own. Then verify the block renders in the editor.

### Field-type mapping

Infer the ACF `type` from the Blade usage:

| Blade usage | ACF `type` | Notes |
|---|---|---|
| `{!! $el['title'] !!}` short text | `text` | `required` when the markup has no `@if` guard. Also for short unformatted copy of a sentence or two (intro, card text, short quote): a `text` field, not a `textarea` |
| a title whose lines are lines in the design | `textarea` | `"new_lines": "br"` — line breaks are the only reason for a `textarea`, besides longer plain text |
| `{!! $el['label'] !!}` | `text` | optional intro/eyebrow |
| `{!! wp_kses_post($el['content']) !!}` rich text | `wysiwyg` | `"toolbar": "full"`, `"media_upload": 0` |
| `esc_url($el['button']['url'])`, `['title']`, `['target']` | `link` | `"return_format": "array"` — one single button |
| `@foreach ($el['buttons'] as $row)` with `$row['button']` | `repeater` `buttons` with one `link` sub-field `button` | two or more buttons side by side — see [Buttons side by side](references/field-patterns.md#buttons-side-by-side) |
| `wp_get_attachment_image($el['image'], …)` | `image` | `"return_format": "id"` and `"preview_size": "thumbnail"` — always; render through `wp_get_attachment_image()` so `srcset`/dimensions come for free |
| `$el['contact']['name']`, `$el['contact']['image']`, … | `group` | related fields forming one unit — see [Grouping related fields](references/field-patterns.md#grouping-related-fields) |
| `@foreach ($el['items'] as $item)` / `have_rows('items')` | `repeater` | model each `$item['x']` as a sub-field |
| `$el['x'] == 'value'` toggle/choice | `true_false` / `select` | choices from the compared values. Only when the editor can see the effect — a setting that swaps a few classes is not a field |
| `<x-svg-icon :name="$item['icon']" />` chosen by the editor | `icon_picker` | a tab with the theme's own icons — see [Icon fields](references/field-patterns.md#icon-fields) |
| number used in markup | `number` | |

When unsure, prefer `text` for anything short without formatting and `wysiwyg` for multi-line/HTML content.

### Field patterns

Six structures recur across elements. The rule is here; the JSON and Blade for each are in [references/field-patterns.md](references/field-patterns.md) — read it before modelling one of them.

- **Buttons side by side** — two or more buttons are **one repeater `buttons` with a single `link` sub-field `button`**, never `button_primary`/`button_secondary` or `button1`/`button2`. The template styles by position, and the field's `instructions` state which position is the main button (e.g. the first). One single button stays a plain `link` field. → [Buttons side by side](references/field-patterns.md#buttons-side-by-side)
- **Grouping related fields** — fields that form one logical unit (a contact person: name, role, image, …) go in one `group` sub-field with its own `instructions`, not a flat list of loose fields. → [Grouping related fields](references/field-patterns.md#grouping-related-fields)
- **One element with a type** — variants of the same kind of block are **one element with a `type`**, not an element per variant: either managed under *Opties > Elementen* and picked through a populated select (CTA), or a `type` select on the page with one `group` per type (Publicaties, Media). The partial hands over to a type partial (`partials/ctas/<type>.blade.php`, `partials/publications/<type>.blade.php`). → [One element with a type](references/field-patterns.md#one-element-with-a-type)
- **Populated (dynamic) fields** — a field whose choices come from PHP gets the fixed `g2o_populate_` name prefix (`g2o_populate_forms`, `g2o_populate_ctas`) and ships with `"choices": []`; an `acf/load_field/name=g2o_populate_*` filter in `app/` fills them. Never hand-fill the choices. → [Populated (dynamic) fields](references/field-patterns.md#populated-dynamic-fields)
- **Icon fields** — a field where the editor chooses an icon is ACF's Icon Picker (`"type": "icon_picker"`, name `icon`) with one custom tab that shows a curated set of icons from `<x-svg-icon>` — never a `select` of icon names. → [Icon fields](references/field-patterns.md#icon-fields)
- **Field order in a group** — text fields first, then buttons and repeaters, then the anchor, and **image fields last**. Also in the element's own wrapper group. → [Field order in a group](references/field-patterns.md#field-order-in-a-group)

### Shared content & existing content

- **An element that recurs with the same content is managed under Opties.** When the same content sits in one element on several pages (USP's, values, quotes), its fields move to *Opties > Elementen*; the element on the page keeps only a note that says where the content is managed — `De inhoud beheer je onder Opties > Elementen > …`, nothing more — and a View Composer hands the option to the partial. Notice it, say so and offer the move; which elements move is the client's call. → [Elements managed under Opties](references/shared-content.md#elements-managed-under-opties)
- **Changing an element that already holds content** — renaming it, moving it to Opties, turning it into a type — touches stored data: export the database first, write through the field **key**, read the new value back, and only then delete the old meta. → [Changing an element that already holds content](references/shared-content.md#changing-an-element-that-already-holds-content)

### Editor helpers

- **Layouts in alphabetical order.** Keep the `layouts` map of the "Elementen" group sorted by label, so the "Element toevoegen" menu reads A–Z. Insert a new layout at its place; after renaming one, move it.
- **The element's name on the front end.** `<x-element-id>` shows a small badge with the layout label ("Element #1", "Publicaties › Team") for logged-in users and on a development environment (`is_user_logged_in() || wp_get_environment_type() === 'development'`). A click copies the element's `#anchor`. The badge is absolutely positioned inside the element's `relative` section and hidden below `xl`.

### Descriptions & placeholders

Every field must guide the editor:

- **`instructions`** — a short, friendly Dutch sentence on every field. Start from the predefined default for the field type and only tailor the subject. Keep it short and simple: one sentence, a second only when the editor could not guess the effect; no explanation of how the template works.
- **Optional fields say `indien gewenst`**, right after the verb (`Voeg indien gewenst een label toe.`). That follows the `required` flag, for fields that hold content — not for a `group`, a `true_false`, a select without `allow_null` or a repeater with a `min`. → [Which fields are optional](references/field-descriptions.md#which-fields-are-optional)
- **Wording**: `Afbeelding`, never `Foto` — also in compounds (`Afbeeldingengalerij`, `portretafbeelding`).
- **`placeholder`** — on every type that supports one (`text`, `textarea`, `email`, `url`, `number`, `password`), a concrete example starting with `Bijvoorbeeld: "…"`. **The example is the text the HTML element holds in the partial** — never invented copy or lorem ipsum. Types without placeholder support (`wysiwyg`, `image`, `link`, `select`, `true_false`, `repeater`, `group`) get only `instructions`.

The predefined `instructions` and `placeholder` per field type are in [references/field-descriptions.md](references/field-descriptions.md) — reuse them verbatim.

### Keys

- Formats: field group `group_<uid>`, field `field_<uid>`, layout `layout_<uid>`.
- `<uid>` is a 13-character lowercase hex string (ACF uses `uniqid()`). Generate a fresh one per key, e.g. `php -r 'echo uniqid();'`.
- **Keys are permanent identifiers.** Never reuse a key across elements, and never change a key after content is saved — ACF stores values against the key, so changing it orphans the data.
- The clone's `"clone": ["group_<uid>"]` must exactly match the standalone group's `key`.

### JSON field & group defaults

When hand-editing JSON, match what the ACF UI writes so Local JSON stays diff-clean:

- **Preserve ACF's default keys** on every field — `aria-label`, `wrapper` (`{ "width": "", "class": "", "id": "" }`), `conditional_logic: 0`, `allow_in_bindings: 0` — and the group-level `menu_order`, `description`, `modified`. Don't strip them.
- **Group defaults**: `"label_placement": "top"`, `"instruction_placement": "label"`, `"position": "normal"`, `"style": "default"`, `"show_in_rest": 0`.
- **`return_format` per type**: `image` and `gallery` → `"id"` (render via `wp_get_attachment_image()`), `link` → `"array"`, `select` / `radio` → `"value"`.
- **`preview_size`**: `image` and `gallery` → `"thumbnail"` — always a small preview in the admin, whatever size the partial renders.
- **Repeater**: `"layout": "table"`, a Dutch `"button_label"` (e.g. `"Onderdeel toevoegen"`), `"parent_repeater"` set to the repeater's `key` on every sub-field, and `min` / `max` only when the design needs them.
- **Location**: give a clone-only element group the same `location` as the "Elementen" flexible content (`post_type == post`), even though it never renders its own metabox.
- **One file per group**: the filename is `group_<key>.json` and must contain the matching `"key"`.
- Store field groups as **local JSON** in `acf-json/` so they version with the theme; edit in the ACF UI and let it write the JSON, or hand-edit and sync. Do not register fields in PHP for these.

### Naming, language & escaping

- **One name, three cases**: ACF layout and field `name`s are snake_case, view files kebab-case, Blade variables camelCase. The layout `name` equals the inner group field `name` (`key_figures`, read via `get_sub_field('key_figures')`), the partial is `partials/key-figures.blade.php` and the variable `$keyFigures`. For a single-word name (`element1`) all three coincide.
- **Partial skeleton**: define `$elementN = get_sub_field('elementN')` at the top (English camelCase variable matching the element), wrap everything in `@if (!empty($elementN))`, add `data-element="elementN"` on the `<section>`, include `<x-element-id>` with `Begin`/`End` comments and bracket the markup with `Element #N: Begin` / `Element #N: End` comments — see the [Blade partial](references/templates.md#blade-partial--partialselement1bladephp) template.
- **Section wrapper**: every element partial wraps in a `<section>`; vertical margin/padding and any full-width `bg-*` sit on the section, the container inside only centers and pads horizontally. A background that must stay contained goes on a div inside the container.
- Keep `"prefix_name": 0` and `"display": "seamless"` on the clone, otherwise the sub-fields land under a prefixed key and `get_sub_field('element1')` returns nothing.
- Set `"active": false` on every standalone element group so it never shows as its own metabox — it is only consumed through the clone.
- **Language**: field `label`, `instructions` and `placeholder` are in Dutch; field and layout `name`s are always English snake_case.
- **Populated fields**: prefix any field whose choices are filled from PHP with `g2o_populate_` (e.g. `g2o_populate_forms`) and leave `"choices": []` in the JSON — see [Populated (dynamic) fields](references/field-patterns.md#populated-dynamic-fields).
- Give **every** field a description (`instructions`) and, where the type supports it, a `placeholder` (`Bijvoorbeeld: "…"`) taken from the text the HTML element holds in the partial — see [Descriptions & placeholders](#descriptions--placeholders).
- Guard every output in Blade with `!empty()`; echo field output with `{!! !!}` (trusted editor content — `{{ }}` would double-escape entities), use `esc_url()`/`esc_attr()` inside attributes — echoed with `{!! !!}`, because `{{ esc_url($url) }}` escapes twice and breaks a URL containing `&` — and `wp_kses_post()` on the WYSIWYG field. Titles may fall back to `get_the_title()` — see [Banner title](references/templates.md#banner-title--fallback-to-the-post-title).
- **Only inline `@php(...)` in partials**: an inline `@php(...)` followed by a `@php … @endphp` block in the same file throws a ParseError — never mix the two forms.
- **Text domain**: a theme has exactly one text domain — the theme's folder name; the templates here write `theme-name` as a placeholder for it. `data-*` values are identifiers and are never translated.

## Reference files

Detail that is only needed for part of the work lives in `references/`. Read the relevant file BEFORE writing the JSON or Blade it covers:

| Topic | Reference |
|---|---|
| Standalone group JSON (`acf-json/group_<uid>.json`) | [references/templates.md](references/templates.md#standalone-group--acf-jsongroup_uidjson) |
| Central "Elementen" flexible content group (once per project) | [references/templates.md](references/templates.md#central-elementen-flexible-content-group--once-per-project) |
| Layout entry with the clone field | [references/templates.md](references/templates.md#layout-entry--inside-the-elementen-flexible-content-group) |
| Blade partial skeleton | [references/templates.md](references/templates.md#blade-partial--partialselement1bladephp) |
| Banner title falling back to the post title | [references/templates.md](references/templates.md#banner-title--fallback-to-the-post-title) |
| Rendered example — where each placeholder comes from | [references/templates.md](references/templates.md#rendered-example--placeholders-filled-in) |
| Buttons side by side (repeater `buttons`) | [references/field-patterns.md](references/field-patterns.md#buttons-side-by-side) |
| Grouping related fields (`group`) | [references/field-patterns.md](references/field-patterns.md#grouping-related-fields) |
| One element with a type (CTA, Publicaties, Media) | [references/field-patterns.md](references/field-patterns.md#one-element-with-a-type) |
| Populated fields (`g2o_populate_*`) | [references/field-patterns.md](references/field-patterns.md#populated-dynamic-fields) |
| Icon fields (`icon_picker` with a theme tab) | [references/field-patterns.md](references/field-patterns.md#icon-fields) |
| Field order in a group (images last) | [references/field-patterns.md](references/field-patterns.md#field-order-in-a-group) |
| Elements managed under Opties (shared content) | [references/shared-content.md](references/shared-content.md#elements-managed-under-opties) |
| Renaming or moving an element that already holds content | [references/shared-content.md](references/shared-content.md#changing-an-element-that-already-holds-content) |
| Predefined `instructions` / `placeholder` per field type, `indien gewenst` | [references/field-descriptions.md](references/field-descriptions.md) |

## Anti-patterns

| Avoid | Do instead |
|---|---|
| Duplicating the same fields inside each layout | One standalone group per element, cloned into the layout |
| `"prefix_name": 1` on the clone | Keep `0` so `get_sub_field('elementN')` resolves |
| Reusing or editing a `field`/`group`/`layout` key | Generate a fresh unique key; never mutate saved keys |
| Standalone group with `"active": true` | `false` — it is clone-only |
| Registering these fields in PHP (`acf_add_local_field_group`) | Local JSON in `acf-json/`, versioned with the theme |
| Mismatched names between layout, partial and group field | One name in all three: snake_case in ACF (`key_figures`), kebab-case view (`key-figures.blade.php`), camelCase variable (`$keyFigures`) |
| A snake_case view file or variable (`key_figures.blade.php`, `$key_figures`) | Kebab-case for the file, camelCase for the variable; snake_case only in ACF |
| A separate element per variant of the same block | One element with a `type` — see [One element with a type](references/field-patterns.md#one-element-with-a-type) |
| `button_primary`/`button_secondary` or `button1`/`button2` | One repeater `buttons` with a `link` sub-field `button`; the template styles by position |
| Fields without `instructions` or (where supported) `placeholder` | Describe every field; add `Bijvoorbeeld: "…"` placeholders |
| Invented/lorem-ipsum placeholder text | The placeholder is the text the HTML element holds in the partial |
| A flat list of loose fields for one logical unit | Bundle them in a `group` with its own `instructions` (e.g. `contact`) |
| Margin/padding or a full-width `bg-*` on the container | On the `<section>` wrapper; the container only centers and pads horizontally |
| A `<div>` as the element wrapper, or an extra `<div>` around the `<section>` | The `<section data-element="elementN">` is the partial's outermost element |
| Dutch field `name`s, or English `instructions`/`placeholder` | Names always English snake_case; descriptions/placeholders in Dutch |
| Hand-filling `choices` on a dynamic select | Prefix with `g2o_populate_`, keep `"choices": []`, fill via `acf/load_field` |
| `"preview_size": "medium"` (or larger) on an image field | `"thumbnail"` — the admin preview is always small |
| Stripping ACF's default keys when hand-editing JSON | Keep `aria-label`, `wrapper`, `conditional_logic`, `allow_in_bindings`, group `menu_order`/`modified` |
| `get_field()`/`get_sub_field()` scattered without guards | Read once into `$elementN`, guard with `!empty()` |
| Unescaped URLs or attributes (`href="{!! $url !!}"`) | `esc_url()` / `esc_attr()` inside attribute output |
| Double escaping (`href="{{ esc_url($url) }}"`) | `{!! esc_url($url) !!}` / `{!! esc_attr($value) !!}` — escape once |
| Inline `@php(...)` and a `@php … @endphp` block in one file | Only the inline form in partials |
| `__()` around a `data-element` value or other identifier | Plain slug: `data-element="element1"` |
| Hardcoding another project's text domain (`academies`, …) | Use the project's own text domain everywhere |
| A descriptive name for a general-purpose block ("Contentblok") | `Element #N` / `elementN`; descriptive names are for elements with one purpose |
| A `textarea` for a sentence of plain text | `text`; a `textarea` only where line breaks matter |
| An image field first or in the middle of a group | Image fields last in the group |
| "Foto" in a label or instruction | "Afbeelding" |
| An optional field without `indien gewenst`, or a long instruction that explains the template | `indien gewenst` after the verb; one short sentence |
| A `select` listing icon names (`g2o_populate_icons`) | `icon_picker` with a tab of theme icons |
| The same content typed into one element on several pages | Manage it under *Opties > Elementen*; the page element only holds a note |
| A note on such an element that describes what it shows | Only where it is managed: `De inhoud beheer je onder Opties > Elementen > …` |
| A setting that only swaps a few classes | No field; one layout |
| Default elements from an options page appended to every detail page | The post's own `elements` only |
| `update_field('<name>', …)` when two fields share the name, then deleting the old meta right away | Write by field key, read back, then delete — after a database export |
| A new layout appended at the end of the `layouts` map | Insert it in alphabetical order by label |

## Output checklist

Verify every item before delivering an element:

- [ ] One name in three cases: ACF layout and field `name` snake_case, view file kebab-case, Blade variable camelCase
- [ ] Standalone group: a single `group` field named after the element, `"active": false`, filename `group_<key>.json` matching its `"key"`
- [ ] Layout added to the `layouts` map of the "Elementen" group, with one clone field: `"display": "seamless"`, `"prefix_label": 0`, `"prefix_name": 0`
- [ ] The clone's `"clone": ["group_<uid>"]` matches the standalone group's `key`
- [ ] Every `group_`, `field_` and `layout_` key is freshly generated and unique; no existing key was changed
- [ ] ACF's default keys are kept on every field (`aria-label`, `wrapper`, `conditional_logic`, `allow_in_bindings`) and on the group (`menu_order`, `description`)
- [ ] Every field has Dutch `instructions`; every type that supports it has a `Bijvoorbeeld: "…"` placeholder taken from the text in the partial
- [ ] Field and layout `name`s are English snake_case; `label`, `instructions` and `placeholder` are Dutch
- [ ] `image` / `gallery`: `"return_format": "id"`, `"preview_size": "thumbnail"`; `link`: `"return_format": "array"`
- [ ] Several buttons are one repeater `buttons` with a `link` sub-field `button`; related fields are bundled in a `group`, with image fields last
- [ ] Short unformatted copy is `text`, not `textarea`; an icon is an `icon_picker` with the theme tab
- [ ] Optional fields say `indien gewenst`; instructions are one short sentence and say `Afbeelding`, not `Foto`
- [ ] A general-purpose block is `elementN` / "Element #N"; the layout sits at its alphabetical place in the `layouts` map
- [ ] Content that repeats on several pages is managed under *Opties > Elementen* (or the move was offered)
- [ ] Stored content was exported before a rename or move, written by field key and read back before the old meta was removed
- [ ] Dynamic choices use the `g2o_populate_` prefix with `"choices": []`
- [ ] Partial: `get_sub_field()` read once at the top, the whole element guarded with `@if (!empty(…))`, every sub-field guarded with its own `!empty()`
- [ ] The `<section data-element="…">` is the outermost element, with `<x-element-id>` and the `Begin` / `End` comments; spacing and full-width `bg-*` sit on the section, not the container
- [ ] Field output echoed with `{!! !!}`; URLs and attributes through `{!! esc_url() !!}` / `{!! esc_attr() !!}`; WYSIWYG through `wp_kses_post()`
- [ ] Only inline `@php(...)` in the partial; the project's own text domain in every translation call
- [ ] Synced in wp-admin (Custom Fields → Field Groups → *Sync available*) and checked in the editor — or reported as still to do

## Related skills

- [wordpress-theme](../wordpress-theme/SKILL.md) — Blade, escaping, View Composers and the rest of the theme conventions these partials sit in.
- [tailwindcss](../tailwindcss/SKILL.md) — the utility classes used in the element markup.
- [gsap](../gsap/SKILL.md) — animate elements through the `data-element` / `data-animate` hooks, never through classes.
