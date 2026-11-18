# Field patterns

Recurring field structures and how each one is modelled in ACF and read in Blade. Part of the [acf-elements](../SKILL.md) skill.

- [Buttons side by side](#buttons-side-by-side)
- [Grouping related fields](#grouping-related-fields)
- [One element with a type](#one-element-with-a-type)
- [Populated (dynamic) fields](#populated-dynamic-fields)

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

**Icons** are a populated field too: use `g2o_populate_icons` (a `select` with `"choices": []`) and render it with the `<x-svg-icon>` component, never inline SVG — all theme SVGs live in that one component, see [SVG icons & logos](../../wordpress-theme/SKILL.md#svg-icons--logos):

```blade
@if (!empty($item['g2o_populate_icons']))
  <x-svg-icon :name="$item['g2o_populate_icons']" />
@endif
```
