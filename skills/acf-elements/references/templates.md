# Templates

These are complete — they carry the default keys ACF writes itself (`aria-label`, `wrapper`, `conditional_logic`, `allow_in_bindings`, `menu_order`, `description`, `modified`). Copy them as-is and keep those keys, so Local JSON stays diff-clean against what the ACF UI would produce.

Part of the [acf-elements](../SKILL.md) skill.

- [Standalone group — `acf-json/group_<uid>.json`](#standalone-group--acf-jsongroup_uidjson)
- [Central "Elementen" flexible content group — once per project](#central-elementen-flexible-content-group--once-per-project)
- [Layout entry — inside the "Elementen" flexible content group](#layout-entry--inside-the-elementen-flexible-content-group)
- [Blade partial — `partials/element1.blade.php`](#blade-partial--partialselement1bladephp)
- [Banner title — fallback to the post title](#banner-title--fallback-to-the-post-title)
- [Rendered example — placeholders filled in](#rendered-example--placeholders-filled-in)

## Standalone group — `acf-json/group_<uid>.json`

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

## Central "Elementen" flexible content group — once per project

The parent group that holds every layout. Created once (first element of the project), then only its `layouts` map grows. Unlike the standalone element groups this one is `"active": true` — it is the metabox the editor works in:

```json
{
  "key": "group_<uid>",
  "title": "Elementen",
  "fields": [
    {
      "key": "field_<uid>",
      "label": "Elementen",
      "name": "elements",
      "aria-label": "",
      "type": "flexible_content",
      "instructions": "Voeg één of meer elementen toe aan de pagina.",
      "required": 0,
      "conditional_logic": 0,
      "wrapper": { "width": "", "class": "", "id": "" },
      "allow_in_bindings": 0,
      "layouts": {
        "layout_<uid>": { "…": "one layout entry per element — see below" }
      },
      "button_label": "Element toevoegen",
      "min": "",
      "max": ""
    }
  ],
  "location": [[{ "param": "post_type", "operator": "==", "value": "post" }]],
  "menu_order": 0,
  "position": "normal",
  "style": "default",
  "label_placement": "top",
  "instruction_placement": "label",
  "hide_on_screen": "",
  "active": true,
  "description": "",
  "show_in_rest": 0
}
```

The `layouts` value is a **map keyed by each layout's `key`**, not an array.

## Layout entry — inside the "Elementen" flexible content group

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

## Blade partial — `partials/element1.blade.php`

Every element partial follows the same skeleton:

1. **Define the variable at the top** (English, matching the element name) with `get_sub_field('elementN')`.
2. **Guard the whole element** with `@if (!empty($elementN))` — render nothing when empty.
3. **The wrapper is always a `<section>`, and it is the partial's outermost element** — never an extra `<div>` around it. It carries `data-element="elementN"`: a plain slug used as a hook for GSAP/QA (an identifier, not UI copy, so never run it through `__()`). The section carries the vertical spacing (`my-*`/`py-*`) and the `bg-*` when the background must run full-width; a background that must stay contained goes on a div inside the container instead.
4. Include the `<x-element-id>` anchor component, wrapped in `Begin`/`End` comments.
5. Wrap the markup in `Element #N: Begin` / `Element #N: End` comments.
6. Inside comes the container (`mx-auto max-w-screen-2xl px-4 sm:px-6 md:px-8`) — it only centers and pads horizontally; never put margins, vertical padding or full-width backgrounds on it.
7. Guard every sub-field with its own `!empty()`. Field output is echoed with `{!! !!}` — element fields are trusted editor content and may contain entities or intentional markup that `{{ }}` would double-escape into visible `&amp;`/`&#8211;`. Use `{!! esc_url() !!}` / `{!! esc_attr() !!}` inside attributes — never `{{ esc_url() }}`, which escapes twice — and run the WYSIWYG field through `wp_kses_post()`.

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
          url="{!! esc_url($element1['button']['url']) !!}"
          target="{!! esc_attr($element1['button']['target'] ?? '_self') !!}" />
      @endif
    </div>
    {{-- Element #1: End --}}
  </section>
@endif
```

## Banner title — fallback to the post title

The banner (the hero at the top of the page) always renders a title. When the editor leaves the ACF title empty, it falls back to the post title instead of rendering nothing:

```blade
<h1 class="...">{!! empty($banner['title']) ? get_the_title() : $banner['title'] !!}</h1>
```

Note the `{!! !!}`: `get_the_title()` returns texturized text — curly quotes, dashes and ampersands as HTML entities — which `{{ }}` would double-escape (the visitor would see a literal `&#8211;`). Any title output that falls back to `get_the_title()` is therefore echoed unescaped; both branches are trusted editor content.

## Rendered example — placeholders filled in

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
