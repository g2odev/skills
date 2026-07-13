---
name: acf-elements
description: 'Make Blade "elements" manageable through ACF using local JSON field groups. Use when adding, editing or generating flexible-content elements (element1, element2, …) where each Blade partial gets a matching ACF field group and a clone layout in the central "Elementen" flexible content — generated as acf-json so it versions with the theme.'
---

# ACF Elements (flexible content via local JSON)

## Contents

- [Documentation](#documentation)
- [When to use](#when-to-use)
- [Core principles](#core-principles)
- [Conventions](#conventions)
  - [Workflow — generate ACF JSON from a Blade element](#workflow--generate-acf-json-from-a-blade-element)
  - [Field-type mapping](#field-type-mapping)
  - [Grouping related fields](#grouping-related-fields)
  - [Descriptions & placeholders](#descriptions--placeholders)
  - [Predefined descriptions per field type](#predefined-descriptions-per-field-type)
  - [Populated (dynamic) fields](#populated-dynamic-fields)
  - [Keys](#keys)
  - [JSON field & group defaults](#json-field--group-defaults)
  - [Naming, language & escaping](#naming-language--escaping)
- [Templates](#templates)
  - [Standalone group](#standalone-group--acf-jsongroup_uidjson)
  - [Layout entry](#layout-entry--inside-the-elementen-flexible-content-group)
  - [Blade partial](#blade-partial--partialselement1bladephp)
  - [Banner title fallback](#banner-title--fallback-to-the-post-title)
  - [Rendered example](#rendered-example--placeholders-filled-in)
- [Anti-patterns](#anti-patterns)
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

Every element is built from **three files that share one name** (`elementN`):

```
acf-json/group_<uid>.json          →  layout "elementN"              →  partials/elementN.blade.php
standalone group, field: elementN     in flexible content "elements"    get_sub_field('elementN')
(active: false)                       (single clone field)              rendered by get_row_layout()
```

1. **Standalone ACF group** (`acf-json/group_*.json`) — one `group` field named `elementN` holding the actual sub-fields (label, title, content, …). Set `"active": false`: it only exists to be cloned, it must not render its own metabox.
2. **Central "Elementen" flexible content** (`field: elements`) — one `layout` per element whose `name` equals the partial name. Each layout contains a single **clone** field that clones the standalone group with `"display": "seamless"` and `"prefix_name": 0`, so the group stays reachable as `get_sub_field('elementN')`.
3. **Blade partial** (`partials/elementN.blade.php`) — read once into `$elementN = get_sub_field('elementN')`, then render each sub-field guarded by `!empty()`.

The loop is generic — `partials/elements.blade.php` walks the rows and includes the partial whose name matches the layout:

```blade
@while (have_rows('elements'))
  @php(the_row())
  @if (View::exists('partials.' . get_row_layout()))
    @include('partials.' . get_row_layout())
  @endif
@endwhile
```

Because the three names match (`elementN`), adding an element is purely additive: no central registry to edit by hand.

> **Why `get_sub_field()` lives in the partial.** The [wordpress-theme](../wordpress-theme/SKILL.md) skill says to map fields in a View Composer instead of scattering `get_field()` through templates. Flexible content is the exception: sub-fields only exist inside the `have_rows()` / `the_row()` loop, so they cannot be resolved up-front in a composer. One `get_sub_field('elementN')` at the top of the partial is the intended pattern — everything below it reads the resulting array, not ACF.

> **Theme components.** The templates below use `<x-element-id>` (scroll anchor), `<x-button>` and `<x-svg-icon>` from `resources/views/components/`. They are project components, not part of ACF — swap them for whatever your theme provides.

## Conventions

### Workflow — generate ACF JSON from a Blade element

Given an element partial (e.g. `element1.blade.php`), generate its field group:

1. **Read the partial** and collect every sub-field access `$elementN['<name>']` plus any `have_rows('<name>')` (a repeater) inside it. The array key = the ACF field `name`.
2. **Infer the ACF type** for each key from how it is used — see [Field-type mapping](#field-type-mapping).
3. **Bundle related fields into a `group`** when they form one logical unit: an element that shows a contact person gets a single `contact` group holding the contact details and image, not five loose fields. Give the group its own `instructions` describing the unit (`Voeg de gegevens van de contactpersoon toe.`) — see [Grouping related fields](#grouping-related-fields).
4. **Write a description and placeholder for every field**: give each field a clear `instructions` sentence, and — for every field type that supports it (text, textarea, email, url, number, password) — a `placeholder` starting with `Bijvoorbeeld: "…"`. **The placeholder text is the content the HTML element holds in the partial** (the example copy from the design): `<h2>Met de juiste kennis vooruit</h2>` becomes `"placeholder": "Bijvoorbeeld: \"Met de juiste kennis vooruit\""`. See [Descriptions & placeholders](#descriptions--placeholders).
5. **Build the standalone group JSON** (`acf-json/group_<uid>.json`): a single `group` field named `elementN` whose `sub_fields` are the inferred fields, in the order they appear in the markup. `"active": false`.
6. **Register the layout** in the central "Elementen" flexible content group: add a `layout` named `elementN` containing one `clone` field that clones the new group (`"display": "seamless"`, `"prefix_label": 0`, `"prefix_name": 0`).
7. **Keys**: give every `key`, `group`, `layout` a fresh unique id (see [Keys](#keys)). Never reuse a key from another element.
8. **Sync**: open Custom Fields → Field Groups in wp-admin and run *Sync available* — ACF picks up the new/changed JSON from `acf-json/` by itself. There is no built-in WP-CLI sync command; only use one if the project ships its own. Then verify the block renders in the editor.

### Field-type mapping

Infer the ACF `type` from the Blade usage:

| Blade usage | ACF `type` | Notes |
|---|---|---|
| `{!! $el['title'] !!}` short text | `text` | `required` when the markup has no `@if` guard |
| `{!! $el['label'] !!}` | `text` | optional intro/eyebrow |
| `{!! wp_kses_post($el['content']) !!}` rich text | `wysiwyg` | `"toolbar": "full"`, `"media_upload": 0` |
| `esc_url($el['button']['url'])`, `['title']`, `['target']` | `link` | `"return_format": "array"` |
| `wp_get_attachment_image($el['image'], …)` | `image` | `"return_format": "id"` — always; render through `wp_get_attachment_image()` so `srcset`/dimensions come for free |
| `$el['contact']['name']`, `$el['contact']['image']`, … | `group` | related fields forming one unit — see [Grouping related fields](#grouping-related-fields) |
| `@foreach ($el['items'] as $item)` / `have_rows('items')` | `repeater` | model each `$item['x']` as a sub-field |
| `$el['x'] == 'value'` toggle/choice | `true_false` / `select` | choices from the compared values |
| number used in markup | `number` | |

When unsure, prefer `text` for single line and `wysiwyg` for multi-line/HTML content.

### Grouping related fields

When several fields form one logical unit, bundle them in a `group` sub-field instead of leaving them as a flat list. An element that shows a contact person gets one `contact` group holding the contact details and image: the editor sees them as one block in the admin, and the Blade side reads one nested array. Give the group its own `instructions` describing the unit as a whole:

```json
{
  "key": "field_<uid>",
  "label": "Contactpersoon",
  "name": "contact",
  "aria-label": "",
  "type": "group",
  "instructions": "Voeg de gegevens van de contactpersoon toe.",
  "required": 0,
  "conditional_logic": 0,
  "wrapper": { "width": "", "class": "", "id": "" },
  "allow_in_bindings": 0,
  "layout": "block",
  "sub_fields": [
    {
      "key": "field_<uid>",
      "label": "Naam",
      "name": "name",
      "aria-label": "",
      "type": "text",
      "instructions": "Voer de naam van de contactpersoon in.",
      "required": 1,
      "conditional_logic": 0,
      "wrapper": { "width": "", "class": "", "id": "" },
      "allow_in_bindings": 0,
      "default_value": "",
      "maxlength": "",
      "placeholder": "Bijvoorbeeld: \"Jan Jansen\"",
      "prepend": "",
      "append": ""
    },
    {
      "key": "field_<uid>",
      "label": "Afbeelding",
      "name": "image",
      "aria-label": "",
      "type": "image",
      "instructions": "Selecteer een afbeelding van de contactpersoon.",
      "required": 0,
      "conditional_logic": 0,
      "wrapper": { "width": "", "class": "", "id": "" },
      "allow_in_bindings": 0,
      "return_format": "id",
      "library": "all",
      "preview_size": "medium"
    }
  ]
}
```

Extend `sub_fields` the same way with the other unit fields (`role`, `phone`, `email`, …). In Blade the group is one nested array:

```blade
@if (!empty($element1['contact']))
  <div class="...">
    @if (!empty($element1['contact']['image']))
      {!! wp_get_attachment_image($element1['contact']['image'], 'medium') !!}
    @endif

    @if (!empty($element1['contact']['name']))
      <p class="...">{!! $element1['contact']['name'] !!}</p>
    @endif
  </div>
@endif
```

### Descriptions & placeholders

Every field must guide the editor, exactly like the example group:

- **`instructions`** — a short, friendly sentence on every field describing what to enter (e.g. `"Voeg een titel toe aan het element."`). Mark optional fields with `"… indien gewenst …"`.
- **`placeholder`** — on every field type that supports one (`text`, `textarea`, `email`, `url`, `number`, `password`), a concrete example starting with `Bijvoorbeeld: "…"`. **Take the example from the partial itself**: the text the HTML element contains is the field's placeholder — don't invent new copy or lorem ipsum. Field types without placeholder support (`wysiwyg`, `image`, `link`, `select`, `true_false`, `repeater`, `group`) get only `instructions`.

| Field type | `instructions` | `placeholder` |
|---|---|---|
| `text`, `textarea`, `email`, `url`, `number`, `password` | ✅ | ✅ `Bijvoorbeeld: "…"` |
| `wysiwyg`, `image`, `link`, `select`, `true_false`, `repeater`, `group` | ✅ | — (not supported) |

### Predefined descriptions per field type

Every field starts from a **predefined default `instructions`** (and, where supported, a placeholder template). Reuse these verbatim, only tailoring the `{onderwerp}` to the field. See the [ACF field type reference](https://www.advancedcustomfields.com/resources/) for each type's options.

| ACF `type` | Default `instructions` | Default `placeholder` |
|---|---|---|
| `text` | `Voeg een {onderwerp} toe aan het element.` | `Bijvoorbeeld: "…"` |
| `textarea` | `Voeg een korte {onderwerp} toe aan het element.` | `Bijvoorbeeld: "…"` |
| `wysiwyg` | `Voeg de inhoud toe aan het element.` | — |
| `number` | `Voer een aantal in voor {onderwerp}.` | `Bijvoorbeeld: "3"` |
| `email` | `Voer een e-mailadres in.` | `Bijvoorbeeld: "naam@voorbeeld.nl"` |
| `url` | `Voer een volledige URL in.` | `Bijvoorbeeld: "https://voorbeeld.nl"` |
| `link` | `Voeg indien gewenst een button of link toe aan het element.` | — |
| `image` | `Selecteer een afbeelding voor het element.` | — |
| `file` | `Upload een bestand voor het element.` | — |
| `gallery` | `Voeg één of meer afbeeldingen toe aan de galerij.` | — |
| `select` | `Kies een optie voor {onderwerp}.` | — |
| `radio` | `Kies één optie voor {onderwerp}.` | — |
| `checkbox` | `Selecteer één of meer opties voor {onderwerp}.` | — |
| `true_false` | `Schakel {onderwerp} in of uit.` | — |
| `repeater` | `Voeg één of meer {onderwerp} toe aan het element.` | — |
| `group` | `Voeg informatie toe aan het element.` | — |
| `relationship` / `post_object` | `Selecteer de gerelateerde items voor het element.` | — |
| `date_picker` | `Kies een datum.` | — |

Mark optional fields by inserting `indien gewenst` (e.g. `Voeg indien gewenst een label toe aan het element.`).

For a `group` that bundles a logical unit, describe the unit itself instead of the generic default: `Voeg de gegevens van de contactpersoon toe.` — the generic `Voeg informatie toe aan het element.` is only for the outer `elementN` wrapper group.

### Populated (dynamic) fields

Fields whose choices are filled programmatically (not typed in the ACF UI) get the **`g2o_populate_` name prefix**, e.g. `g2o_populate_forms`, `g2o_populate_filters`. The prefix is fixed — always `g2o_populate_`, in every project. In the JSON these fields ship with an empty `"choices": []`; the options are injected at runtime via an `acf/load_field/name=g2o_populate_*` filter in `app/`.

```json
{
  "key": "field_<uid>",
  "label": "Formulier",
  "name": "g2o_populate_forms",
  "aria-label": "",
  "type": "select",
  "instructions": "Maak een keuze welk formulier getoond kan worden.",
  "required": 1,
  "conditional_logic": 0,
  "wrapper": { "width": "", "class": "", "id": "" },
  "allow_in_bindings": 0,
  "choices": [],
  "default_value": "",
  "return_format": "value",
  "multiple": 0,
  "allow_null": 0,
  "ui": 0
}
```

The `g2o_populate_` prefix signals “choices come from PHP”, so an empty `choices` array in the JSON is expected and correct — never hand-fill them.

**Icons** are a populated field too: use `g2o_populate_icons` (a `select` with `"choices": []`) and render it with the `<x-svg-icon>` component, never inline SVG:

```blade
@if (!empty($item['g2o_populate_icons']))
  <x-svg-icon :name="$item['g2o_populate_icons']" />
@endif
```

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
- **Repeater**: `"layout": "table"`, a Dutch `"button_label"` (e.g. `"Onderdeel toevoegen"`), `"parent_repeater"` set to the repeater's `key` on every sub-field, and `min` / `max` only when the design needs them.
- **Location**: give a clone-only element group the same `location` as the "Elementen" flexible content (`post_type == post`), even though it never renders its own metabox.
- **One file per group**: the filename is `group_<key>.json` and must contain the matching `"key"`.
- Store field groups as **local JSON** in `acf-json/` so they version with the theme; edit in the ACF UI and let it write the JSON, or hand-edit and sync. Do not register fields in PHP for these.

### Naming, language & escaping

- **Names must match everywhere**: layout `name` = partial filename (`element1` → `partials/element1.blade.php`) = the inner group field `name` read via `get_sub_field('element1')`.
- **Partial skeleton**: define `$elementN = get_sub_field('elementN')` at the top (English name matching the element), wrap everything in `@if (!empty($elementN))`, add `data-element="elementN"` on the `<section>`, include `<x-element-id>` with `Begin`/`End` comments and bracket the markup with `Element #N: Begin` / `Element #N: End` comments — see the [Blade partial](#blade-partial--partialselement1bladephp) template.
- **Section wrapper**: every element partial wraps in a `<section>`; vertical margin/padding and any full-width `bg-*` sit on the section, the container inside only centers and pads horizontally. A background that must stay contained goes on a div inside the container.
- Keep `"prefix_name": 0` and `"display": "seamless"` on the clone, otherwise the sub-fields land under a prefixed key and `get_sub_field('element1')` returns nothing.
- Set `"active": false` on every standalone element group so it never shows as its own metabox — it is only consumed through the clone.
- **Language**: field `label`, `instructions` and `placeholder` are in Dutch; field `name`s are always English snake_case.
- **Populated fields**: prefix any field whose choices are filled from PHP with `g2o_populate_` (e.g. `g2o_populate_forms`) and leave `"choices": []` in the JSON — see [Populated (dynamic) fields](#populated-dynamic-fields).
- Give **every** field a description (`instructions`) and, where the type supports it, a `placeholder` (`Bijvoorbeeld: "…"`) taken from the text the HTML element holds in the partial — see [Descriptions & placeholders](#descriptions--placeholders).
- Guard every output in Blade with `!empty()`; echo field output with `{!! !!}` (trusted editor content — `{{ }}` would double-escape entities), use `esc_url()`/`esc_attr()` inside attributes and `wp_kses_post()` on the WYSIWYG field. Titles may fall back to `get_the_title()` — see [Banner title](#banner-title--fallback-to-the-post-title).
- **Text domain**: a theme has exactly one text domain — the theme's folder name; the templates here write `theme-name` as a placeholder for it. `data-*` values are identifiers and are never translated.

## Templates

These are complete — they carry the default keys ACF writes itself (`aria-label`, `wrapper`, `conditional_logic`, `allow_in_bindings`, `menu_order`, `description`, `modified`). Copy them as-is and keep those keys, so Local JSON stays diff-clean against what the ACF UI would produce.

### Standalone group — `acf-json/group_<uid>.json`

```json
{
  "key": "group_<uid>",
  "title": "Element #1",
  "fields": [
    {
      "key": "field_<uid>",
      "label": "Element #1",
      "name": "element1",
      "aria-label": "",
      "type": "group",
      "instructions": "Voeg informatie toe aan het element.",
      "required": 0,
      "conditional_logic": 0,
      "wrapper": { "width": "", "class": "", "id": "" },
      "allow_in_bindings": 0,
      "layout": "block",
      "sub_fields": [
        {
          "key": "field_<uid>",
          "label": "Label",
          "name": "label",
          "aria-label": "",
          "type": "text",
          "instructions": "Voeg indien gewenst een label toe aan het element.",
          "required": 0,
          "conditional_logic": 0,
          "wrapper": { "width": "", "class": "", "id": "" },
          "allow_in_bindings": 0,
          "default_value": "",
          "maxlength": "",
          "placeholder": "Bijvoorbeeld: \"Gerichte trainingen met een praktisch resultaat\"",
          "prepend": "",
          "append": ""
        },
        {
          "key": "field_<uid>",
          "label": "Titel",
          "name": "title",
          "aria-label": "",
          "type": "text",
          "instructions": "Voeg een titel toe aan het element.",
          "required": 1,
          "conditional_logic": 0,
          "wrapper": { "width": "", "class": "", "id": "" },
          "allow_in_bindings": 0,
          "default_value": "",
          "maxlength": "",
          "placeholder": "Bijvoorbeeld: \"Met de juiste kennis vooruit\"",
          "prepend": "",
          "append": ""
        },
        {
          "key": "field_<uid>",
          "label": "Inhoud",
          "name": "content",
          "aria-label": "",
          "type": "wysiwyg",
          "instructions": "Voeg de inhoud toe aan het element.",
          "required": 1,
          "conditional_logic": 0,
          "wrapper": { "width": "", "class": "", "id": "" },
          "allow_in_bindings": 0,
          "default_value": "",
          "tabs": "all",
          "toolbar": "full",
          "media_upload": 0,
          "delay": 0
        },
        {
          "key": "field_<uid>",
          "label": "Button",
          "name": "button",
          "aria-label": "",
          "type": "link",
          "instructions": "Voeg indien gewenst een button toe aan het element.",
          "required": 0,
          "conditional_logic": 0,
          "wrapper": { "width": "", "class": "", "id": "" },
          "allow_in_bindings": 0,
          "return_format": "array"
        }
      ]
    }
  ],
  "location": [[{ "param": "post_type", "operator": "==", "value": "post" }]],
  "menu_order": 0,
  "position": "normal",
  "style": "default",
  "label_placement": "top",
  "instruction_placement": "label",
  "hide_on_screen": "",
  "active": false,
  "description": "",
  "show_in_rest": 0
}
```

### Layout entry — inside the "Elementen" flexible content group

Add this object to the `layouts` map of the `elements` flexible-content field, keyed by its own `layout_<uid>`. The `clone` array references the standalone group's `key`:

```json
{
  "key": "layout_<uid>",
  "name": "element1",
  "label": "Element #1",
  "display": "block",
  "sub_fields": [
    {
      "key": "field_<uid>",
      "label": "Element #1",
      "name": "element1_clone",
      "aria-label": "",
      "type": "clone",
      "instructions": "",
      "required": 0,
      "conditional_logic": 0,
      "wrapper": { "width": "", "class": "", "id": "" },
      "allow_in_bindings": 0,
      "clone": ["group_<uid>"],
      "display": "seamless",
      "layout": "block",
      "prefix_label": 0,
      "prefix_name": 0
    }
  ],
  "min": "",
  "max": ""
}
```

### Blade partial — `partials/element1.blade.php`

Every element partial follows the same skeleton:

1. **Define the variable at the top** (English, matching the element name) with `get_sub_field('elementN')`.
2. **Guard the whole element** with `@if (!empty($elementN))` — render nothing when empty.
3. **The wrapper is always a `<section>`**, with `data-element="elementN"` — a plain slug used as a hook for GSAP/QA (an identifier, not UI copy, so never run it through `__()`). The section carries the vertical spacing (`my-*`/`py-*`) and the `bg-*` when the background must run full-width; a background that must stay contained goes on a div inside the container instead.
4. Include the `<x-element-id>` anchor component, wrapped in `Begin`/`End` comments.
5. Wrap the markup in `Element #N: Begin` / `Element #N: End` comments.
6. Inside comes the container (`mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8`) — it only centers and pads horizontally; never put margins, vertical padding or full-width backgrounds on it.
7. Guard every sub-field with its own `!empty()`. Field output is echoed with `{!! !!}` — element fields are trusted editor content and may contain entities or intentional markup that `{{ }}` would double-escape into visible `&amp;`/`&#8211;`. Use `esc_url()` / `esc_attr()` inside attributes, and run the WYSIWYG field through `wp_kses_post()`.

Replace `theme-name` with the project's actual text domain: the theme's folder name.

```blade
@php($element1 = get_sub_field('element1'))

@if (!empty($element1))
  <section class="relative my-12 bg-white lg:my-16 2xl:my-20" data-element="element1">
    {{-- Component - Add ID to element: Begin --}}
    <x-element-id :text="sanitize_title_with_dashes($element1['title'] ?? __('Element', 'theme-name'))" />
    {{-- Component - Add ID to element: End --}}

    {{-- Element #1: Begin --}}
    <div class="mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8">
      @if (!empty($element1['label']))
        <p class="...">{!! $element1['label'] !!}</p>
      @endif

      @if (!empty($element1['title']))
        <h2 class="...">{!! $element1['title'] !!}</h2>
      @endif

      @if (!empty($element1['content']))
        <div class="prose ...">{!! wp_kses_post($element1['content']) !!}</div>
      @endif

      @if (!empty($element1['button']['url']))
        <x-button :message="$element1['button']['title'] ?? ''"
          url="{{ esc_url($element1['button']['url']) }}"
          target="{{ esc_attr($element1['button']['target'] ?? '_self') }}" />
      @endif
    </div>
    {{-- Element #1: End --}}
  </section>
@endif
```

### Banner title — fallback to the post title

The banner (the hero at the top of the page) always renders a title. When the editor leaves the ACF title empty, it falls back to the post title instead of rendering nothing:

```blade
<h1 class="...">{!! empty($banner['title']) ? get_the_title() : $banner['title'] !!}</h1>
```

Note the `{!! !!}`: `get_the_title()` returns texturized text — curly quotes, dashes and ampersands as HTML entities — which `{{ }}` would double-escape (the visitor would see a literal `&#8211;`). Any title output that falls back to `get_the_title()` is therefore echoed unescaped; both branches are trusted editor content.

### Rendered example — placeholders filled in

Drop example text into the exact HTML spots the partial outputs each field. This makes it obvious where every field lands — and it is the **source of each field's `placeholder`**: the text an element holds here becomes `Bijvoorbeeld: "…"` in the generated JSON (compare the standalone group above):

```html
<section class="relative my-12 lg:my-16" data-element="element1">
  <!-- label → placeholder: Bijvoorbeeld: "Gerichte trainingen met een praktisch resultaat" -->
  <p class="...">Gerichte trainingen met een praktisch resultaat</p>

  <!-- title → placeholder: Bijvoorbeeld: "Met de juiste kennis vooruit" -->
  <h2 class="...">Met de juiste kennis vooruit</h2>

  <!-- content → wysiwyg (no placeholder support, instructions only) -->
  <div class="prose ...">
    <p>Onze trainingen combineren theorie met praktijk, zodat je het geleerde direct kunt toepassen in je eigen werk.</p>
  </div>

  <!-- button → link (no placeholder support, instructions only) -->
  <a href="#" class="...">Bekijk het aanbod</a>
</section>
```

## Anti-patterns

| Avoid | Do instead |
|---|---|
| Duplicating the same fields inside each layout | One standalone group per element, cloned into the layout |
| `"prefix_name": 1` on the clone | Keep `0` so `get_sub_field('elementN')` resolves |
| Reusing or editing a `field`/`group`/`layout` key | Generate a fresh unique key; never mutate saved keys |
| Standalone group with `"active": true` | `false` — it is clone-only |
| Registering these fields in PHP (`acf_add_local_field_group`) | Local JSON in `acf-json/`, versioned with the theme |
| Mismatched names between layout, partial and group field | Use the exact same `elementN` in all three |
| Fields without `instructions` or (where supported) `placeholder` | Describe every field; add `Bijvoorbeeld: "…"` placeholders |
| Invented/lorem-ipsum placeholder text | The placeholder is the text the HTML element holds in the partial |
| A flat list of loose fields for one logical unit | Bundle them in a `group` with its own `instructions` (e.g. `contact`) |
| Margin/padding or a full-width `bg-*` on the container | On the `<section>` wrapper; the container only centers and pads horizontally |
| A `<div>` as the element wrapper | Always a `<section data-element="elementN">` |
| Dutch field `name`s, or English `instructions`/`placeholder` | Names always English snake_case; descriptions/placeholders in Dutch |
| Hand-filling `choices` on a dynamic select | Prefix with `g2o_populate_`, keep `"choices": []`, fill via `acf/load_field` |
| Stripping ACF's default keys when hand-editing JSON | Keep `aria-label`, `wrapper`, `conditional_logic`, `allow_in_bindings`, group `menu_order`/`modified` |
| `get_field()`/`get_sub_field()` scattered without guards | Read once into `$elementN`, guard with `!empty()` |
| Unescaped URLs or attributes (`href="{!! $url !!}"`) | `esc_url()` / `esc_attr()` inside attribute output |
| `__()` around a `data-element` value or other identifier | Plain slug: `data-element="element1"` |
| Hardcoding another project's text domain (`academies`, …) | Use the project's own text domain everywhere |

## Related skills

- [wordpress-theme](../wordpress-theme/SKILL.md) — Blade, escaping, View Composers and the rest of the theme conventions these partials sit in.
- [tailwindcss](../tailwindcss/SKILL.md) — the utility classes used in the element markup.
- [gsap](../gsap/SKILL.md) — animate elements through the `data-element` / `data-animate` hooks, never through classes.
