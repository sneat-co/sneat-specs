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
loading-vs-empty distinction.

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

Group related items under a divider that doubles as a collapsible, tappable
header with actions in `slot="end"`:

```html
<ion-item-group>
  <ion-item-divider (click)="clickGroup(listGroup)" tappable>
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
*(listus `lists-page.component.html`)*

Use a chevron icon to signal collapse state, and call
`$event.stopPropagation()` in header buttons so they don't toggle the group.

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
| Grouped / collapsible | `ion-item-group` + tappable `ion-item-divider` |
| Count on a group header | `ion-badge` in the divider |
