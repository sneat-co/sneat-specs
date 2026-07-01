# Lists

Scrollable collections use `ion-list` inside a card, with Ionic's sliding and
grouping patterns layered on as needed.

## Lists live inside cards

Wrap a scrollable collection in `<ion-list>` inside the section's `<ion-card>`,
and iterate with `@for` (always with `track`):

```html
<ion-card>
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

Group related items under a divider that doubles as a collapsible header, with
actions in `slot="end"` — see [below](#clickable-rows-need-a-real-interactive-element)
for how to make that header actually keyboard-operable:

```html
<ion-item-group>
  <ion-item-divider (click)="clickGroup(listGroup)" tabindex="0" role="button"
    (keydown.enter)="clickGroup(listGroup)">
    <ion-icon [name]="isCollapsed(listGroup) ? 'chevron-down-outline' : 'chevron-up-outline'" />
    <ion-label>{{ listGroup.title }}</ion-label>
    <ion-buttons slot="end">
      <ion-button (click)="addTo($event, listGroup.type)">
        <ion-icon name="add" />
      </ion-button>
    </ion-buttons>
  </ion-item-divider>
  <!-- grouped items -->
</ion-item-group>
```
*(listus `lists-page.component.html`, adjusted — see below.)*

Use a chevron icon to signal collapse state, and call
`$event.stopPropagation()` in header buttons so they don't toggle the group.

## Clickable rows need a real interactive element

`tappable` is **not an `ion-item` / `ion-item-divider` property** — it isn't in
Ionic's `IonItem`/`IonItemDivider` component interfaces (checked against
`@ionic/core`'s `components.d.ts`). Setting `tappable="true"` on a row compiles
and does nothing: the row still renders as a plain, non-focusable `<div>`, so
keyboard and screen-reader users can't reach it even though `(click)` fires
for a mouse. This is a recurring copy-paste mistake — it shows up in
calendarius, contactus, `space` components, `sneat-ui`'s own
`select-from-list.component.html`, and (until this fix) in this very doc's
grouped-list example above.

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

- **For `ion-item-divider`** (no `button` prop exists on it at all): add
  `tabindex="0" role="button"` plus a `(keydown.enter)` (and usually
  `(keydown.space)`) handler alongside `(click)`, as in the corrected snippet
  above — there's no native-element shortcut here, so the keyboard path has to
  be added by hand.

Applies to every "grouped/collapsible list" and "picker" pattern in this file
and in [`cards.md`](./cards.md) — audit existing `tappable` usages when
touching that code.

### Alternative: `ion-accordion-group` for collapsible groups

For genuinely collapsible groups (as opposed to a static divider), Ionic's
`ion-accordion-group` / `ion-accordion` is an accepted — and often better —
alternative to hand-rolling `ion-item-group` + chevron + `stopPropagation`: it
provides expand/collapse animation, single-vs-multiple expansion, and a11y
semantics for free. Put the header in `slot="header"` and gate any header action
with `stopPropagation()` as usual.

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

## Counts — `ion-badge` in a divider

Show counts with an `ion-badge` in the divider:

```html
<ion-item-divider>
  <ion-label>{{ group.brief?.title }}</ion-label>
  <ion-badge color="light">{{ group.dbo?.numberOf?.members }} members</ion-badge>
</ion-item-divider>
```
*(contactus `members-page.component.html`)*

## Summary

| Need | Pattern |
| --- | --- |
| Scrollable list | `ion-list` inside `ion-card`, `@for … track` + `@empty` |
| Per-item swipe actions | `ion-item-sliding` + `ion-item-options` (success start / danger end) |
| Grouped / collapsible | `ion-item-group` + `ion-item-divider` (`tabindex`/`role="button"` for keyboard) |
| Clickable row | `ion-item [button]="true"` — **not** `tappable` (not a real prop) |
| Count on a group header | `ion-badge` in the divider |
