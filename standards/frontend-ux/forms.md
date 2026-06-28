# Forms & Inputs

Form controls are wrapped in `ion-item`, grouped in cards (on a page) or in
`ion-content` (in a modal), and validated with inline `color="danger"` messages
shown only after the field is touched.

## Each control in an `ion-item`

Wrap every input — `ion-input`, `ion-textarea`, `ion-select`, `ion-checkbox`,
`ion-radio`, `ion-datetime` — in an `<ion-item>`. Group related controls in an
`<ion-card>`, optionally with an `<ion-item-divider color="light">` section header.

```html
<ion-card>
  <ion-item-divider color="light"><ion-label>Address</ion-label></ion-item-divider>
  <ion-item>
    <ion-input label="Street" [formControl]="street" />
  </ion-item>
</ion-card>
```
*(contactus `address-form.component.html`.)*

## Labels — by control type

| Control | Label convention |
| --- | --- |
| `ion-input` | inline `label="Field"` |
| `ion-textarea` | `label="…" labelPlacement="stacked"` (label above), usually `auto-grow="true"` |
| `ion-checkbox` / `ion-radio` | `labelPlacement="end"` (control left, label right) |
| `ion-select` | `interface="popover"`, `label="…"` |

Examples: calendarius `happening-form.component.html` (stacked textarea),
`happening-slot-form.component.html` (`labelPlacement="end"` checkbox),
`happening-title-modal.component.html` (inline input label).

## Reactive forms are the default

Use Angular **reactive forms** (`FormGroup` + `FormControl`, `Validators.*`) as
the default — it's what calendarius and contactus use. Template-driven `ngModel`
(increasingly with signals) appears in listus for small one-field dialogs; that's
acceptable for trivial cases, but **Recommended:** reach for reactive forms for
anything with validation or more than one field, and don't mix `[formControl]`
and `[(ngModel)]` in the same form.

## Validation — inline, after touch

Show validation errors with an `ion-label color="danger"`, gated on the field
being touched so the form isn't red before the user types:

```html
@if (title.touched && !title.valid && !!title.errors) {
  <ion-label color="danger">Title is required</ion-label>
}
```
*(calendarius `start-end-datetime-form.component.html`,
`happening-form.component.html`.)*

- Prefer **specific** messages ("Email is not valid") over a bare "Required field".
- Pair any visual `required` marker with an actual `Validators.required` — don't
  set `required` on the input without the validator.
- For form-level errors, a single `ion-card` with `ion-card-header color="danger"`
  is the established pattern *(contactus `new-member-form.component.html`)*.

## Submit

- **Disable** the submit button while the form is invalid or a save is in flight,
  and show an inline spinner — see the disabled/in-flight pattern in
  [`buttons.md`](./buttons.md).
- **Placement:** in a **modal**, the submit/cancel pair goes in the modal
  `ion-footer` (see [`modals.md`](./modals.md)); on a **page**, put the primary
  submit in a full-width button (`expand="block"`) at the end of the form's card.

## Summary

- One control per `ion-item`; group in cards / dividers.
- Labels: inline for inputs, `stacked` for textareas, `end` for checkboxes/radios.
- Reactive forms by default; don't mix with `ngModel`.
- Errors: `color="danger"`, only after `touched`, specific messages.
- Submit: disabled-until-valid + in-flight spinner; footer in modals, card on pages.
