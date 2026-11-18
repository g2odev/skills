# Field patterns

Recurring field structures and how each one is modelled in ACF and read in Blade. Part of the [acf-elements](../SKILL.md) skill.

- [Buttons side by side](#buttons-side-by-side)
- [Grouping related fields](#grouping-related-fields)
- [Field order in a group](#field-order-in-a-group)
- [One element with a type](#one-element-with-a-type)
- [Populated (dynamic) fields](#populated-dynamic-fields)
- [Icon fields](#icon-fields)

## Buttons side by side

Two or more buttons next to each other are **one repeater `buttons` with a single `link` sub-field `button`** — never `button_primary`/`button_secondary` or `button1`/`button2`. The template decides the style by position; which position is the main button is stated in the field's `instructions` (e.g. `Voeg één of meer buttons toe. De eerste button is de hoofdbutton.`). One single button stays a plain `link` field.

```blade
@foreach ($element1['buttons'] as $row)
  @if (!empty($row['button']['url']))
    <x-button :url="$row['button']['url']" :target="$row['button']['target'] ?: '_self'"
      class="{{ $loop->first ? 'bg-red' : 'border border-beige bg-forest' }} …">
      {!! $row['button']['title'] !!}
    </x-button>
  @endif
@endforeach
```

## Grouping related fields

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
      "preview_size": "thumbnail"
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

## Field order in a group

Inside a `group` — the element's own wrapper group included — the fields follow one order: text fields first (label, title, text), then buttons and repeaters, then the anchor, and **image fields last**. Several images keep their own order at the bottom; when the group has tabs, an image goes to the end of its own tab. Images inside a repeater row are not moved.

```
label, title, text  →  buttons, items  →  anchor  →  image, image_back
```

## One element with a type

Variants of the same kind of block become **one element with a type**, not a separate element per variant. That keeps the "Elementen" list short for the editor. There are two forms:

**Managed under Options (CTA).** The versions live in a repeater under *Opties > Elementen*, each row with a `name` and a `type`. The element itself only holds the populated select `g2o_populate_ctas` (see [Populated (dynamic) fields](#populated-dynamic-fields)). The partial `partials/cta.blade.php` looks up the chosen row and hands over to `partials/ctas/<type>.blade.php`.

**Type on the page (Publicaties, Media).** The element has a select `type` and, per type, a `group` whose `name` equals the type value, shown through conditional logic on `type`. The partial hands over to `partials/publications/<type>.blade.php` or `partials/media/<type>.blade.php`:

```blade
{{-- partials/publications.blade.php --}}
@php($publications = get_sub_field('publications'))
@php($publicationType = $publications['type'] ?? '')
@php($publicationView = 'partials.publications.' . str_replace('_', '-', $publicationType))

@if (!empty($publicationType) && View::exists($publicationView))
  @include($publicationView, ['publication' => $publications[$publicationType] ?? []])
@endif
```

- Type values are snake_case like every other ACF name; the type partial's filename is the same value in kebab-case.
- The type partial receives only its own group (`$publication`), not the whole element.
- Items that such a type lists but that have no page of their own — team members, frequently asked questions — are a **non-public post type** (`'public' => false, 'show_ui' => true`), picked with a `relationship` field (`"return_format": "id"`). They get no URL, stay out of the site search and the sitemap, and can be reused on several pages. A new kind of list becomes a new type, not a new element.
- Show the chosen type behind the row title in the editor (`Publicaties › Aanbod: overzicht met filters`) through the `acf/fields/flexible_content/layout_title` filter in `app/`, so collapsed rows stay recognisable.

## Populated (dynamic) fields

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

## Icon fields

A field where the editor chooses an icon is ACF's [Icon Picker](https://www.advancedcustomfields.com/resources/icon-picker/) (`"type": "icon_picker"`, name `icon`) with a custom tab that shows the theme's own icons — never a `select` with icon names. The editor sees the icons and clicks one; the template gets the icon name for `<x-svg-icon>` (all theme SVGs live in that one component, see [SVG icons & logos](../../wordpress-theme/SKILL.md#svg-icons--logos)).

```json
{
  "key": "field_<uid>",
  "label": "Icoon",
  "name": "icon",
  "aria-label": "",
  "type": "icon_picker",
  "instructions": "Selecteer indien gewenst een icoon voor de USP.",
  "required": 0,
  "conditional_logic": 0,
  "wrapper": { "width": "", "class": "", "id": "" },
  "allow_in_bindings": 0,
  "library": "all",
  "tabs": ["theme_icons"],
  "return_format": "string",
  "default_value": { "type": null, "value": null }
}
```

Register the tab and its icons in `app/filters.php`. Offer a **curated list with Dutch labels** (a helper such as `picker_icons()` returning `name => label`) — icons that mean something next to a text, not the arrows, form controls and logos that also live in the component:

```php
add_filter('acf/fields/icon_picker/tabs', function (array $tabs): array {
    $tabs['theme_icons'] = __('Iconen', 'theme-name');

    return $tabs;
});

add_filter('acf/fields/icon_picker/theme_icons/icons', function (): array {
    return array_map(function (string $name, string $label): array {
        $svg = trim(Blade::render('<x-svg-icon :name="$name" />', ['name' => $name]));
        // The picker draws the icon as a CSS background: an image that knows nothing of the site.
        $svg = preg_replace('/var\(--color-[a-z0-9-]+\)|currentColor|#fff(?:fff)?\b/i', '#1d2327', $svg);

        return ['key' => $name, 'label' => $label, 'url' => 'data:image/svg+xml;base64,' . base64_encode($svg)];
    }, array_keys(picker_icons()), picker_icons());
});
```

- **Data URI, base64** — the picker puts the URL unquoted in `background: url( … )`; a raw SVG string breaks there.
- **Write the colors out** — `currentColor` and `var(--color-*)` do not resolve inside an image, and light colors vanish on the white admin. Keep an accent color if you like; make everything else dark.
- **`"tabs"` holds only the theme tab**, so Dashicons, media library and URL are not offered.
- **An empty picker returns the default array, not a string** — check `is_string()` before handing the value to `<x-svg-icon>`:

```blade
@if (is_string($item['icon'] ?? null) && $item['icon'] !== '')
  <x-svg-icon :name="$item['icon']" class="size-8" aria-hidden="true" />
@endif
```
