# Shared content & existing content

Elements whose content is managed once under Opties, and how to change an element that already holds content. Part of the [acf-elements](../SKILL.md) skill.

- [Elements managed under Opties](#elements-managed-under-opties)
- [Changing an element that already holds content](#changing-an-element-that-already-holds-content)

## Elements managed under Opties

When an element comes back on several pages with the **same content** (USP's, values, quotes, a marquee), its content is managed once under *Opties > Elementen*, not filled in per page. When you notice identical content in one element on several pages, say so and offer the move — which elements move is the client's call.

- **Options field group** ("Opties - Elementen"): one tab per element, holding a `group` named after the element (`usp`, `values`) with all its fields — give them new keys, never the keys of the page element. A set the editor picks from (several CTA's) is a repeater with a required internal `name` instead, see [One element with a type](field-patterns.md#one-element-with-a-type).
- **The element on the page** keeps its layout and wrapper group but has no content fields — only a `message` field that says where the content lives, and nothing else (no description of what the element shows):

```json
{
  "key": "field_<uid>",
  "label": "USP's",
  "name": "info",
  "aria-label": "",
  "type": "message",
  "instructions": "",
  "required": 0,
  "conditional_logic": 0,
  "wrapper": { "width": "", "class": "", "id": "" },
  "allow_in_bindings": 0,
  "message": "De inhoud beheer je onder <strong>Opties &gt; Elementen &gt; USP's</strong>.",
  "new_lines": "wpautop",
  "esc_html": 0
}
```

- **A View Composer** for the partial reads the option and hands it over; the partial no longer calls `get_sub_field()` and renders when the option holds content:

```php
class Usp extends Composer
{
    protected static $views = ['partials.usp'];

    public function with(): array
    {
        return ['usp' => get_field('usp', 'option') ?: []];
    }
}
```

```blade
{{-- The element has no fields of its own: all content comes from "Opties > Elementen > USP's" (Usp composer). --}}
@if (!empty($usp['title']) || !empty($usp['items']))
  <section class="relative …" data-element="usp">
    …
  </section>
@endif
```

- This does not contradict "detail pages own their elements": the element is still placed on each page, only its content is shared.

## Changing an element that already holds content

Renaming an element, moving its content to Opties or turning it into a type touches stored data. ACF stores a flexible-content row as `elements_<n>_<field>…` meta plus a `_elements_<n>_<field>…` reference to the field key, and the layout names as a serialized array in the `elements` meta.

1. **Export the database first** (`wp db export`) and save a copy of the affected pages' HTML to compare afterwards.
2. **Write through the field key, not the name**, when two fields share a name (the page element and the options group are both called `usp`): `update_field('field_<uid>', $value, 'option')`. By name ACF may resolve the wrong field and save nothing — while reporting success.
3. **Read the new value back and check it before deleting the old meta.** Never delete in the same step as the write; make the script stop when the check fails.
4. **Rename** = rename the `elements_<n>_<old>…` and `_elements_<n>_<old>…` meta keys and replace the name in the serialized `elements` array, for every post (drafts too). Field keys stay the same. Rename the partial, the variable, `data-element` and the `Begin` / `End` comments with it, and move the layout to its alphabetical place.
5. **Removing a row** from a page goes through `delete_row('<field key of elements>', $index, $postId)`, which renumbers the rows that follow.
6. **Compare** the rendered sections before and after; they should be identical apart from nonces and obfuscated e-mail addresses.
