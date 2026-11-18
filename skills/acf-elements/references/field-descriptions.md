# Field descriptions & placeholders

The `instructions` and `placeholder` every field gets, with the predefined Dutch default per field type. Part of the [acf-elements](../SKILL.md) skill.

- [Descriptions & placeholders](#descriptions--placeholders)
- [Which fields are optional](#which-fields-are-optional)
- [Predefined descriptions per field type](#predefined-descriptions-per-field-type)

## Descriptions & placeholders

Every field must guide the editor, exactly like the example group:

- **`instructions`** — a short, friendly sentence on every field describing what to enter (e.g. `"Voeg een titel toe aan het element."`). Keep it short and simple: one sentence, a second one only when the editor could not guess the effect otherwise; aim for under 100 characters. No explanation of how the template works. Mark optional fields with `indien gewenst` — see [Which fields are optional](#which-fields-are-optional).
- **Wording** — say `Afbeelding` / `Afbeeldingen`, never `Foto`, also in compounds (`Afbeeldingengalerij`, `portretafbeelding`, `hoofdafbeelding`).
- **`placeholder`** — on every field type that supports one (`text`, `textarea`, `email`, `url`, `number`, `password`), a concrete example starting with `Bijvoorbeeld: "…"`. **Take the example from the partial itself**: the text the HTML element contains is the field's placeholder — don't invent new copy or lorem ipsum. Field types without placeholder support (`wysiwyg`, `image`, `link`, `select`, `true_false`, `repeater`, `group`) get only `instructions`.

| Field type | `instructions` | `placeholder` |
|---|---|---|
| `text`, `textarea`, `email`, `url`, `number`, `password` | ✅ | ✅ `Bijvoorbeeld: "…"` |
| `wysiwyg`, `image`, `link`, `select`, `true_false`, `repeater`, `group` | ✅ | — (not supported) |

## Which fields are optional

`indien gewenst` goes right after the verb: `Voeg indien gewenst een label toe.`, `Selecteer indien gewenst een afbeelding.`, `Voer indien gewenst een volledige URL in.` It follows the `required` flag:

| Field | `indien gewenst` |
|---|---|
| `"required": 0` and holds content: `text`, `textarea`, `wysiwyg`, `image`, `file`, `link`, `url`, `email`, `number`, `post_object`, `relationship`, `repeater`, `gallery`, `icon_picker`, a `select` with `allow_null` | ✅ |
| `"required": 1` | — |
| `group` (a container), `message` | — |
| `true_false`, a `select` / `radio` without `allow_null` (they always hold a value) | — |
| a `repeater` or `gallery` with a `min` | — |

If "indien gewenst" reads oddly because the element is useless without the field (the link of a menu item, the file of a download), the field should be `required` instead.

## Predefined descriptions per field type

Every field starts from a **predefined default `instructions`** (and, where supported, a placeholder template). Reuse these verbatim, only tailoring the `{onderwerp}` to the field. See the [ACF field type reference](https://www.advancedcustomfields.com/resources/) for each type's options.

| ACF `type` | Default `instructions` | Default `placeholder` |
|---|---|---|
| `text` | `Voeg een {onderwerp} toe aan het element.` | `Bijvoorbeeld: "…"` |
| `textarea` | `Voeg een {onderwerp} toe, met elke regel op een nieuwe regel.` | `Bijvoorbeeld: "…"` |
| `wysiwyg` | `Voeg de inhoud toe aan het element.` | — |
| `number` | `Voer een aantal in voor {onderwerp}.` | `Bijvoorbeeld: "3"` |
| `email` | `Voer een e-mailadres in.` | `Bijvoorbeeld: "naam@voorbeeld.nl"` |
| `url` | `Voer een volledige URL in.` | `Bijvoorbeeld: "https://voorbeeld.nl"` |
| `link` | `Voeg indien gewenst een button of link toe aan het element.` | — |
| `image` | `Selecteer een afbeelding voor het element.` | — |
| `icon_picker` | `Selecteer een icoon voor {onderwerp}.` | — |
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
| `message` on an element filled from Opties | — (the `message` itself: `De inhoud beheer je onder <strong>Opties &gt; Elementen &gt; {naam}</strong>.`) | — |

Mark optional fields by inserting `indien gewenst` (e.g. `Voeg indien gewenst een label toe aan het element.`). A short unformatted text is a `text` field; the `textarea` default above is for the remaining case, a text whose line breaks matter.

For a `group` that bundles a logical unit, describe the unit itself instead of the generic default: `Voeg de gegevens van de contactpersoon toe.` — the generic `Voeg informatie toe aan het element.` is only for the outer `elementN` wrapper group.
