# Loading, Empty & Error States

Every data-bound view handles three transient states, not just the happy path:
**loading** (data not yet arrived), **empty** (data arrived, nothing in it), and
**error** (something failed). State is driven by `signal()`s and Angular control
flow (`@if` / `@for` / `@empty`).

> **State of the codebase:** loading and empty states are common; error states
> are thin and message wording is inconsistent. The conventions below are the
> **target** — apply them to new and touched code, and prefer the wording here
> over inventing new phrasings.

## Distinguish loading from empty

The key rule: only show "empty" once data has actually loaded. Tell the two apart
by whether the backing signal is defined yet.

```html
@for (contact of $contacts() ?? []; track contact.id) {
  <ion-item>…</ion-item>
} @empty {
  @if ($contacts()) {
    <ion-label color="medium">No contacts yet.</ion-label>   <!-- loaded, empty -->
  } @else {
    <ion-label color="medium">Loading…</ion-label>           <!-- not loaded yet -->
  }
}
```
*(contactus `contacts-checklist.component.html`; calendarius
`recurrings-tab.component.html`.)*

## Loading

| Pattern | When | Example |
| --- | --- | --- |
| Spinner + text | Whole list/section not yet loaded | `ion-spinner` + `<ion-text color="medium">Loading…</ion-text>` in a card *(calendarius `single-happenings-list.component.html`)* |
| Skeleton text | Content-shaped placeholder for list rows | `ion-skeleton-text animated="animated"` *(contactus `members-list.component.html`)* |
| Inline action spinner | A row/button action in flight | swap the icon for `<ion-spinner>` while a signal like `isPersisting()` is true *(listus `list-page.component.html`, `list-item.component.html`)* |

```html
@if (!$happenings()) {
  <ion-card><ion-card-content>
    <ion-spinner name="lines" class="ion-margin-end" />
    <ion-text color="medium">Loading…</ion-text>
  </ion-card-content></ion-card>
}
```

**Recommended:** spinner + text for a whole section; skeletons for list rows;
inline spinner (icon swap) for per-row/button actions. Keep the message a plain
`"Loading…"` — don't bake debug context into user-facing text.

## Empty

Use the `@empty` branch of `@for` (or an explicit `!$data()?.length` check) with a
`color="medium"` message, and offer the primary "add" affordance right there when
it makes sense:

```html
@for (recurring of $recurrings() ?? []; track recurring.id) {
  <sneat-happening-card [happening]="recurring" />
} @empty {
  <ion-item>
    <ion-label class="ion-text-wrap">No recurring activities yet.</ion-label>
    <ion-buttons slot="end">
      <ion-button color="primary" routerLink="../new-happening">
        <ion-label>Add first</ion-label>
      </ion-button>
    </ion-buttons>
  </ion-item>
}
```
*(calendarius `recurrings-tab.component.html`,
`calendar-weekday.component.html`.)*

**Recommended wording:** `"No <items> yet."` — consistent across the app, rather
than the current mix of "No records found" / "No items" / "No members".

## Error

Surface errors; don't swallow them. Three established shapes:

| Pattern | When | Example |
| --- | --- | --- |
| Inline `color="danger"` label | Field/section validation | below a form control *(calendarius `happening-form.component.html`)* — see [`forms.md`](./forms.md) |
| Error card | Form-/section-level failure | `ion-card-header color="danger"` *(contactus `new-member-form.component.html`)* |
| Toast `color="danger"` | Transient action failure | `ToastController` *(listus `new-list-item.component.ts`)* — see [`modals.md`](./modals.md) |

**Recommended:** give data-fetching components an `error` signal and render a
`color="danger"` message (with a retry affordance where it helps) when it's set —
not just forms. Use a toast for failures of one-off actions (save, delete).

## Conventions at a glance

- State via `signal()`s; `@if` / `@for … track` / `@empty` in templates (no
  `*ngIf` / `*ngFor`, no RxJS in the template).
- `color="medium"` for loading/empty text; `color="danger"` for errors.
- Loading text is plain `"Loading…"`; empty text is `"No <items> yet."`.
- Distinguish loading from empty by whether the backing signal is defined.
