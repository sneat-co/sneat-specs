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

### Surface failures — never log-only

The single most common state defect across the surveyed apps is a mutating action
whose `error:` handler only calls `errorLogger.logError(...)` (console/telemetry)
with **no user-visible feedback** — the spinner just resets and nothing appears to
have happened. This is a **silent failure** (see [`flows.md`](./flows.md)) and is
not acceptable for an action the user initiated.

- Every `error:` callback of a user-initiated mutation (create/save/delete/
  archive/invite) must surface a `color="danger"` toast **in addition to**
  `errorLogger.logError(...)` — the logger is for you, the toast is for the user.
- This is a **target**, not the current state: calendarius uses `ToastController`
  **nowhere** (all failures funnel silently through `errorLogger`, e.g.
  `slot-context-menu.component.ts`), and trackus's `new-tracker-form.component.ts`
  create-failure path is log-only. Apply the rule to new and touched code.

### The canonical tri-state signal formula

Derive loading / empty / error from signals so the three are mutually exclusive
and can't flicker. trackus's `trackers.component.ts` is the reference:

```ts
$error = signal<Error | undefined>(undefined);
$loading = computed(() => !!spaceID && this.$trackers() === undefined && !this.$error());
// empty is then simply: $trackers()?.length === 0
```
*(trackus `components/trackers/trackers.component.ts` — `@if ($loading()) …
@else if ($error()) … @else … @empty`.)*

### Skeleton rows — dim the real shape

For list loading, prefer a **skeleton row shaped like the real item** over a bare
spinner: a dimmed placeholder avatar (`opacity: 0.3` + a generic gravatar src) plus
`ion-skeleton-text animated="animated"`, as in contactus
`members/.../members-list.component.html`. This reads as "content arriving" rather
than "blocked".

### Persisting small view-state locally

Collapsed-group state, "show completed/watched" toggles, and similar per-user view
preferences should persist across sessions **without** a backend round-trip. listus
does this with a reusable `LocalAppState<AppState>` base (localStorage read/write +
a `changed` observable) — see listus `services/listus-app-state.service.ts`. Reach
for that pattern instead of re-deriving view-state on every visit.

## Conventions at a glance

- State via `signal()`s; `@if` / `@for … track` / `@empty` in templates (no
  `*ngIf` / `*ngFor`, no RxJS in the template).
- `color="medium"` for loading/empty text; `color="danger"` for errors.
- Loading text is plain `"Loading…"`; empty text is `"No <items> yet."`.
- Distinguish loading from empty by whether the backing signal is defined.
