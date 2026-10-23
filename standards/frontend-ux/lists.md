# Lists

Scrollable collections use `ion-list` inside a card, with Ionic's sliding and
grouping patterns layered on as needed.

## Lists live inside cards

Wrap a scrollable collection in `<ion-list>` inside the section's `<ion-card>`,
and iterate with `@for` (always with `track`):

```html
<ion-card>
  <ion-item-divider color="light"><ion-label>Items</ion-label></ion-item-divider>
  <ion-list>
    @for (item of items; track item.id) {
      <ion-item>…</ion-item>
    } @empty {
      <ion-item lines="none">
        <ion-label color="medium">No items yet.</ion-label>
      </ion-item>
    }
  </ion-list>
</ion-card>
```

Always provide the `@empty` branch — see [`states.md`](./states.md) for the
loading-vs-empty distinction. This is **not optional**: listus's own
`list-page.component.html` and `lists-page.component.html` render nothing at all
when their collections are empty (no `@empty`, no "nothing here yet" copy), which
leaves a blank card with no orientation for the user. A list without an `@empty`
branch is incomplete.

Examples: listus `list-page.component.html` (reorderable items),
contactus `contacts-component.html` (contact list with a trailing "Add" item).

## Swipe actions — `ion-item-sliding`

For per-item confirm/destructive actions, use `ion-item-sliding` with
`ion-item-options` on either side, and keep an inline button for the common
action:

```html
<ion-item-sliding #ionSliding>
  <ion-item>
    <ion-checkbox … />
    <ion-label>{{ listItem.title }}</ion-label>
    <ion-buttons slot="end">
      <ion-button (click)="deleteFromList(listItem)">
        <ion-icon name="close-outline" />
      </ion-button>
    </ion-buttons>
  </ion-item>
  <ion-item-options side="start">
    <ion-item-option color="success">Done</ion-item-option>
  </ion-item-options>
  <ion-item-options side="end">
    <ion-item-option color="danger">Remove</ion-item-option>
  </ion-item-options>
</ion-item-sliding>
```
*(listus `list-item.component.html`)*

- `side="start"` → positive/confirm (`color="success"`).
- `side="end"` → destructive (`color="danger"`).

## Grouped / collapsible lists — `ion-item-group` + `ion-item-divider`

Group related items under a static divider. Use `ion-accordion-group` when the
group must collapse; for the structure, styling, heading, count, and accessible
custom collapse rules, see the dedicated [`ion-item-divider` reference](./components/ion-item-divider.md).

Keep group-level actions in `slot="end"`. A divider is not an action control by
default; keep any interaction on a real button inside it.

## Clickable rows need a real interactive element

`tappable` is **not an `ion-item` / `ion-item-divider` property** — it isn't in
Ionic's `IonItem`/`IonItemDivider` component interfaces (checked against
`@ionic/core`'s `components.d.ts`). Setting `tappable="true"` on a row compiles
and does nothing: the row still renders as a plain, non-focusable `<div>`, so
keyboard and screen-reader users can't reach it even though `(click)` fires
for a mouse. This is a recurring copy-paste mistake — it shows up in
calendarius, contactus, `space` components, and `sneat-ui`'s own
`select-from-list.component.html`.

- **For a clickable `ion-item`** (a row that navigates or selects, like an
  asset-type picker entry): use the real prop, `button="true"`. This renders an
  actual `<button>` internally — focusable, `Enter`/`Space`-activatable, with
  the right ARIA role, for free.

  ```html
  <!-- assetus new-asset-page.component.html — grouped asset-kind picker -->
  <ion-item [lines]="..." button="true" (click)="selectCategory(c)">
    <ion-icon slot="start" [name]="c.iconName" color="medium" />
    <ion-label>{{ c.title }}</ion-label>
  </ion-item>
  ```
  *(Before: `tappable="true"`, inert to keyboard. After: `button="true"`,
  keyboard-operable — fixed in assetus PR #48.)*

- **For `ion-item-divider`**, do not set `tappable` or use the divider as a
  button. Keep it a heading and use a real `ion-button` with `aria-expanded`
  and `aria-controls` when a custom collapse control is needed; see the
  [`ion-item-divider` reference](./components/ion-item-divider.md#collapsible-groups).

Applies to every "grouped/collapsible list" and "picker" pattern in this file
and in [`cards.md`](./cards.md) — audit existing `tappable` usages when
touching that code.

**The same gap exists on `ion-chip`** when it's used as a single-select option
picker (e.g. a going/maybe/out attendance picker) — `ion-chip` has no `button`
prop at all (checked against `@ionic/core`'s `Chip` class: only `color` /
`outline` / `disabled`), so a bare `(click)` leaves it just as non-focusable as
`tappable`. Add the keyboard path by hand, plus `role="radio"`/`aria-checked`
(or `role="button"` for a non-exclusive chip) so the selection state is
announced:

```html
<ion-chip
  [color]="status() === s.id ? 'primary' : 'medium'"
  tabindex="0"
  role="radio"
  [attr.aria-checked]="status() === s.id"
  (click)="status.set(s.id)"
  (keydown.enter)="status.set(s.id)"
  (keydown.space)="$event.preventDefault(); status.set(s.id)"
  >{{ s.label }}</ion-chip
>
```
*(gameboard `rsvp-page.component.ts` — the going/maybe/out attendance picker on
its anon RSVP landing; found because it's the core action on the page, not
because of a lint rule.)*

### Alternative: `ion-accordion-group` for collapsible groups

For genuinely collapsible groups (as opposed to a static divider), prefer
Ionic's `ion-accordion-group` / `ion-accordion` to a custom collapse control: it
provides expand/collapse animation, single-vs-multiple expansion, and
accessibility semantics. Put the header in `slot="header"`; if it has a
separate action button, keep that action from toggling the accordion.

```html
<ion-accordion-group>
  <ion-accordion>
    <ion-item slot="header" color="light">
      <ion-label>{{ category.emoji }} {{ category.title }}</ion-label>
      <ion-buttons slot="end">
        <ion-button (click)="addTo($event, category)"><ion-icon name="add" /></ion-button>
      </ion-buttons>
    </ion-item>
    <div slot="content"><!-- grouped items --></div>
  </ion-accordion>
</ion-accordion-group>
```
*(trackus `components/trackers/trackers.component.html`.)*

For optional counts on group headings, see the [`ion-item-divider` reference](./components/ion-item-divider.md#optional-counts-and-trailing-actions).

## Grouped list with a computed fill/availability summary

A recurring shape: a roster/list where every row has a **status derived from a
separate response map** (RSVP'd going/maybe/out, a signup sheet's filled/open
slots, an approval queue's approved/pending/rejected), plus a **one-line summary**
of how full/complete the whole thing is ("8 of 12 going, need 4 more"). gameboard's
roster console (`game-invites/:gameId`) is the reference:

- **One pure fold function**, not per-template computation: `computeRosterFill(
  roster, responses, target): FillSummary` takes the raw list + the response map
  + the target count and returns everything the UI needs — the grouped rows
  *and* the summary line — in one pass (gameboard `roster-fill.ts`). Keep it
  side-effect-free and unit-test it directly, the same way [`forms.md`](./forms.md)'s
  balance-display convention keeps `formatSignedBalance` separate from any
  component.
- **The "no entry yet" status is derived, never persisted** — a roster member with
  no response is displayed as e.g. `no-reply`, computed as the fallback in the
  fold, not written to storage. Only real responses are persisted.
- **Group by the derived status** (going / maybe / out / no-reply, or your
  domain's equivalent), each group rendered only `@if (group.rows.length > 0)` —
  but see the empty-summary gap below.
- **The summary line is prominent and reads as a sentence**, not a bare
  fraction: `"{{goingCount}} of {{needed}} going, need {{remaining}} more"` /
  `"… — full"` when `isFull`, computed once in the same fold (gameboard
  `RosterFillSummary.fillLabel`).

### Don't let "every group empty" render nothing

If every status group is conditionally hidden when empty (as above), an empty
*overall* collection (no roster members at all yet) renders **nothing at all**
under the section heading — worse than the missing-`@empty` gap flagged at the
top of this file, because there's no single `@for`/`@empty` pair to catch it;
the emptiness is spread across N independently-hidden groups. Add one explicit
check ahead of the grouped loop for the whole-collection-empty case:

```html
@if (fill().total === 0) {
  <ion-note class="hint">No players on the roster yet — add one below.</ion-note>
} @else {
  @for (group of groups(); track group.key) {
    @if (group.rows.length > 0) { … }
  }
}
```
*(gameboard `roster-page.component.ts` — caught by exercising a freshly-created,
still-empty roster, not by reading the per-group template.)*

## Summary

| Need | Pattern |
| --- | --- |
| Scrollable list | `ion-list` inside `ion-card`, `@for … track` + `@empty` |
| Per-item swipe actions | `ion-item-sliding` + `ion-item-options` (success start / danger end) |
| Grouped / collapsible | `ion-item-group` + static `ion-item-divider`; use `ion-accordion` or a separate accessible button for collapse |
| Clickable row | `ion-item [button]="true"` — **not** `tappable` (not a real prop) |
| Count on a group header | `ion-badge` in the divider |
| Grouped-by-derived-status + fill summary | one pure fold fn for rows + summary; guard the whole-collection-empty case, not just each group |
