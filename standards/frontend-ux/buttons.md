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

> **Note on divergence (measured):** the apps do **not** actually agree here, and
> `expand="block"` is nearly nonexistent in practice. contactus has **zero**
> `expand="block"` usages — all 4 of its full-width page CTAs are
> `ion-grid > ion-row > ion-col > ion-button expand="full"` (e.g.
> `new-member-form`, `member-removal-page`). listus concentrates `expand` in one
> file (`list-page.component.html`: 6×`full`, 2×`block` in its action row/footer).
> calendarius barely uses `expand` at all (its page submit is `size="large"` with
> no `expand`, so it is **not** full-width).
> **Recommended (unchanged, but be honest about the gap):** for a full-width CTA
> in normal content flow, `expand="block"` is the intended house style; the
> de-facto majority is `expand="full"` inside an `ion-grid` column. Pick one per
> screen and don't leave a primary "Create X" button content-sized-by-accident
> (calendarius's `happening-form` submit is the cautionary example).

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

## State-reactive inline "Add" button

For an inline add/submit button paired with a single text field (an add-item bar),
drive both `color` and `fill` from the field's content so the button visibly
"arms" once there's something to submit:

```html
<ion-button
  [color]="isFocused() && title().trim() ? 'primary' : 'medium'"
  [fill]="title().trim() ? 'solid' : 'outline'"
  (click)="add()">
  <ion-icon name="add" slot="start" /><ion-label>Add</ion-label>
</ion-button>
```
*(listus `pages/list/new-list-item/new-list-item.component.html`.)*

This is a cleaner affordance than a permanently-primary button next to an empty
field — the emphasis appears only when the action is meaningful.

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
