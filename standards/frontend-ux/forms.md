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

> **Note on divergence (measured):** this file is not internally consistent
> yet, and that's a signal of what the codebase actually does. The table
> above prescribes plain inline `label="Field"` for `ion-input`, but this
> same file's own [money-pair example](#money-amount-inputs--value--currency-as-a-pair)
> — sourced from real assetus code — uses `label="Appraised value"
> labelPlacement="stacked"` on an `ion-input`. A third style, the pre-Ionic
> "sibling `ion-label` next to the control" (`<ion-item><ion-label>Field</ion-label>
> <ion-input .../></ion-item>`, no `label=` attribute at all), is what docus's
> `new-document-page.component.html` uses throughout its typed-fields and
> legacy-fields sections. All three coexist across the surveyed apps.
> **Recommended:** `labelPlacement="stacked"` is closer to the de-facto
> majority for data-entry inputs (not just textareas) — treat the table's
> "inline for `ion-input`" row as the aspirational case for short single-word
> labels, and reach for `stacked` when the label is more than one or two
> words, or when the field sits in a dense grid of similar fields. Don't
> silently convert working sibling-`ion-label` forms to chase this — apply
> the attribute-based form only to new fields or when a form is touched
> wholesale anyway.

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

## Money amount inputs — value + currency as a pair

A monetary field is **two controls treated as one unit**: a numeric amount and
a currency, always rendered adjacently, both bound into a single `{ value,
currency }` shape on submit (not two independent form fields the user could
mismatch):

```html
<ion-item>
  <ion-input label="Appraised value" labelPlacement="stacked" type="number"
    [(ngModel)]="appraisedValue" />
</ion-item>
<ion-item>
  <ion-select label="Currency" labelPlacement="stacked" [(ngModel)]="appraisedCurrency">
    @for (currency of currencies; track currency) {
      <ion-select-option [value]="currency">{{ currency }}</ion-select-option>
    }
  </ion-select>
</ion-item>
```
*(assetus `asset-add-valuable.component.html` — `appraisedValue`/`appraisedCurrency`;
the identical shape repeats for `renewalCost`/`renewalCurrency` in
`asset-add-digital-asset.component.html`.)*

- Default the currency (`CurrencyCode`, from the shared `CurrencyList`) rather
  than leaving it blank — assetus defaults to `'USD'`.
- On submit, map the pair into one object and **omit it entirely** when the
  amount is empty, rather than sending a zero or a currency with no amount:
  ```ts
  appraisedValue: this.appraisedValue
    ? { value: this.appraisedValue, currency: this.appraisedCurrency }
    : undefined,
  ```
- Keep the two `ion-item`s adjacent (no unrelated field between them) so the
  pairing reads visually, even though they're separate controls.

## Displaying a signed balance — color by direction

A balance between two parties (who owes whom) is directional: the same numeric
field means opposite things depending on who's asking. debtus's balance screens
(home summary, per-contact list, transfer receipt) establish the convention:

- **Color by direction, not by the sign of the raw number** — money owed *to*
  the user is `color="success"`, money the user owes is `color="warning"`,
  applied consistently everywhere a balance or a transfer amount is shown:
  ```html
  <ion-badge [color]="t.direction === 'lend' ? 'success' : 'warning'">
    {{ formatAmount(t.amount) }}
  </ion-badge>
  ```
  *(debtus `debtus-contact-details-page.component.html`,
  `transfer-details-page.component.html`, `debtus-home-page.component.html`.)*
- **One pure formatting function per shape, kept beside the model, not
  duplicated per component**: `formatAmount({ value, currency })` for a plain
  amount, `formatSignedBalance(balance)` for the user-perspective sentence
  ("owes you 30.00 USD" / "you owe 30.00 USD" / "settled up" when the net is
  zero) — both unit-tested independently of any component. *(debtus
  `extension-debtus-contract/src/lib/models/balance-utils.ts` +
  `balance-utils.spec.ts`.)*
- **Treat net-zero as its own state**, not a `0.00` row — debtus's
  `isZeroBalance()` filters settled contacts out of the home page's "by
  contact" list entirely, and `summarizeBalances()` drops a currency from the
  aggregate the moment its net crosses back through zero, rather than showing
  "owes you 0.00 USD".

This complements [money amount inputs](#money-amount-inputs--value--currency-as-a-pair)
above, which covers *entering* money; this covers *displaying* a value whose
sign carries domain meaning (debt direction), not just magnitude.

## Multi-entity role pickers (named contact/entity roles)

Some entities relate to more than one of *another* entity, each in a distinct,
named capacity — a marriage certificate has exactly 2 spouses; a birth
certificate has a child plus 1-2 parents; an employment contract has an
employee and an employer. Don't model this as one multi-select picker with a
bare max count (that loses "which one is the child vs. the parent"); model it
as **one single-select picker per named role**, driven by a small schema:

```ts
interface IRoleDef {
  readonly id: string;
  readonly label: string;
  readonly min: number; // 0 = optional
  readonly max: number; // usually 1 per role
}
```

```html
<ion-card>
  <ion-item-divider color="light"><ion-label>People</ion-label></ion-item-divider>
  @for (role of schema.contactRoles; track role.id) {
    @if (isRoleVisible(role)) {
      <ion-item lines="none">
        <ion-label color="medium">
          {{ role.label }}{{ role.min === 0 ? " (optional)" : "" }}
        </ion-label>
      </ion-item>
      <sneat-contacts-selector-input [$max]="1"
        [$selectedContacts]="roleContactsFor(role.id)"
        (selectedContactsChange)="onRoleContactsChange(role.id, $event)" />
    } @else {
      <ion-item button="true" lines="none" (click)="revealOptionalRole(role)">
        <ion-label color="primary">+ Add {{ role.label }}</ion-label>
      </ion-item>
    }
  }
</ion-card>
```
*(docus `new-document-page.component.html` / `doc-contact-roles.ts` —
`IDocContactRoleDef` + `validateDocContactRoles()`; marriage = 2×`{min:1,max:1}`
"Spouse" roles, birth = `child` + `parent1` (`min:1`) + `parent2` (`min:0`).)*

- **One role, one picker**, reusing the app's existing single-entity selector
  (here, contactus's `sneat-contacts-selector-input` with `$max="1"`) — don't
  build a bespoke multi-select for this; compose the existing single-select
  per role instead.
- **Optional roles (`min: 0`) start collapsed** behind a "+ Add {role}" row
  (a real `button="true"` `ion-item`, not a visible empty picker) and reveal
  their picker on click — this keeps a 2-required/1-optional set (birth
  certificate) from showing an empty, unlabelled third picker by default.
  Once revealed, a role stays revealed for the rest of the session.
- **Validate with a pure function** over the schema (`validateXRoles(schema,
  refs): string[]`, one message per violated role — "Spouse is required" /
  "Parent: choose at most 1") rather than hand-rolled per-field checks; keep
  it side-effect-free and unit-test it directly (docus's
  `doc-contact-roles.spec.ts`).
- **Gate the rest of the form behind "required roles filled"** rather than
  rendering inline per-role error text: hide the typed-fields section and the
  submit button until every `min`-required role has a selection. This is
  progressive disclosure, not the touched-field inline-error pattern earlier
  in this doc — appropriate here because there's no single "the form" to mark
  touched; each role is materially a separate, addable sub-form.

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
- Money: amount + currency as an adjacent pair, mapped to one `{ value,
  currency }` object, omitted (not zeroed) when empty.
- Multi-entity roles: one single-select picker per named role, optional roles
  collapsed behind "+ Add {role}", validated by a pure schema function, rest
  of the form gated behind required roles being filled.
- Submit: disabled-until-valid + in-flight spinner; footer in modals, card on pages.
