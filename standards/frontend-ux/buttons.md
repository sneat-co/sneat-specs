# Buttons

`ion-button` conventions: colour carries meaning, fill marks emphasis level, and
`expand` controls width. Icons are used freely; text labels are reserved for
actions that need to be sought out.

## Colour — semantic, never decorative

| `color` | Use for | Example |
| --- | --- | --- |
| `primary` | Affirmative primary action (submit, add, save) | new-contact submit |
| `medium` | Secondary / neutral (edit, cancel-ish, header actions) | edit/add in headers |
| `light` | De-emphasised / outline-style | "Reactivate completed" |
| `success` | Positive confirmation | "Done", share |
| `warning` | Caution | "Delete completed", archive |
| `danger` | Destructive | delete / remove / "Clear list" |

Example: listus `list-page.component.html` uses `light` (reactivate), `warning`
(delete completed), and `danger` (clear list) side by side for an action row.

## Fill — emphasis level

| `fill` | Meaning |
| --- | --- |
| *(default, solid)* | Standard button, e.g. toolbar actions, the primary submit |
| `outline` | Secondary action (form cancel, secondary "Add") |
| `clear` | Tertiary / link-like (text links inside cards) |

Examples: `fill="outline"` for an in-card "Add" (listus `lists-page`),
`fill="clear"` for a link-style button (listus `list-page`).

## Expand — width

| `expand` | Use |
| --- | --- |
| *(none)* | Content-sized — toolbars, headers, inline actions |
| `expand="block"` | **Recommended** for full-width actions in normal content flow |
| `expand="full"` | Full-width inside dialogs / modal content / grid cells |

> **Note on divergence:** listus uses `expand` heavily (`full` in grid cells,
> `block` in content); contactus uses it sparingly. **Recommended:** reach for
> `expand="block"` for full-width buttons in page content, and reserve
> `expand="full"` for dialogs/modals and grid columns.

## Disabled & in-flight state

Disable a primary/submit button while its action is invalid or in flight, and
show progress inline so the user knows something is happening:

```html
<ion-button color="primary" [disabled]="!formIsValid() || $isSubmitting()"
  (click)="submit()">
  @if ($isSubmitting()) {
    <ion-spinner name="lines-small" slot="start" />
  } @else {
    <ion-icon name="save-outline" slot="start" />
  }
  <ion-label>Save</ion-label>
</ion-button>
```
*(pattern from calendarius `happening-form` / `happening-title-modal`.)*

Disabling on submit also guards against double-clicks. See
[`states.md`](./states.md) for in-flight spinners more broadly.

## Icons

- Icon-only buttons for quick/secondary actions in headers, toolbars, and list
  items (`name="add"`, `"create"`, `"trash"`, `"close-outline"`, …).
- Icon **+** label for primary actions ("Add", "Share", "Done") or when space
  allows.
- Use standard Ionicons names. Prefer the `-outline` variant for neutral/utility
  actions (`add-outline`, `close-outline`, `save-outline`); reserve solid icons
  for emphasis.

## `ion-fab`

Not used in the surveyed extensions — actions are placed inline, in headers, or in
footers rather than as a floating action button. Prefer those over `ion-fab`
unless a screen genuinely calls for it.
