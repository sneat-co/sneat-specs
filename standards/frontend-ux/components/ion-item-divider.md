# `ion-item-divider`

Use `ion-item-divider` to introduce a named group of related items inside an
`ion-list`. It separates list sections and gives each section a visible label.
Use it for group headings, including the first group when the list already has
an overall title in its card header. Do not substitute it for that overall title.

## Choose the right element

- Use [`ion-list-header`](https://ionicframework.com/docs/api/list-header) once
  at the top of a list to describe that list.
- Use `ion-item-divider` to introduce groups in an `ion-item-group` when each
  group needs its own heading. It may introduce the first group as well.
- Use a plain `ion-item` for a row of list content or an action. For a purely
  visual break outside a list, use the layout's border or an HTML `<hr>` where
  the break is meaningful; a visual line alone does not name a list group.

Ionic describes item dividers as list elements placed between groups of items;
see its [component reference](https://ionicframework.com/docs/api/item-divider).
The shared list patterns, including empty states and collapsible groups, live in
[`lists.md`](../lists.md).

## Shared appearance and heading

Use `color="light"` for the divider's neutral surface. Ionic applies the
`--ion-color-light` background and `--ion-color-light-contrast` foreground.
This gives every group a clear boundary and keeps list-section headings
visually consistent across Ionic apps in the Sneat platform. Apply it to all
grouped lists, including connected-store and demo-project sections. Do not
leave a divider to pick a platform-dependent default or add a one-off
background color.

Put the section name in an `ion-label` with an `h3`. The visible heading names
the section for everyone and establishes its place in the page's heading
hierarchy. Use the appropriate heading level if the surrounding page structure
requires a different level.

```html
<ion-list>
  <ion-item-group>
    <ion-item-divider color="light">
      <ion-label><h3>Connected stores</h3></ion-label>
    </ion-item-divider>

    @for (store of stores(); track store.id) {
      <ion-item>
        <ion-label>{{ store.title }}</ion-label>
      </ion-item>
    } @empty {
      <ion-item lines="none">
        <ion-label color="medium">No connected stores yet.</ion-label>
      </ion-item>
    }
  </ion-item-group>
</ion-list>
```

Use the same structure for a “Demo projects” group or any other named section.
Keep the list inside its containing card as described in [`lists.md`](../lists.md);
never put a card inside another card ([`cards.md`](../cards.md)).

## Optional counts and trailing actions

A count is secondary information, so show it as a compact `ion-badge` after the
heading. Omit it when the count does not help people scan or compare groups.

```html
<ion-item-divider color="light">
  <ion-label><h3>{{ group.title }}</h3></ion-label>
  <ion-badge color="medium">{{ group.items.length }}</ion-badge>
</ion-item-divider>
```

Put a group-level action in `slot="end"` only when it acts on that group. Keep
the divider itself static; an action button inside a parent `role="button"`
creates nested interactive controls.

## Collapsible groups

For a group that expands and collapses, prefer Ionic's
[`ion-accordion-group`](https://ionicframework.com/docs/api/accordion-group)
and `ion-accordion`; its header provides the interaction and expanded state.
Keep the divider static when the group is only visually separated.

If a custom collapse control is needed, keep the divider as a heading and put a
real `ion-button` in it. The button must expose its state and point to the
controlled content. Do not make the whole divider clickable or add
`role="button"` to it when it contains other controls.

```html
<ion-list>
  <ion-item-group>
    <ion-item-divider color="light">
      <ion-label>
        <h3 [id]="'heading-' + group.id">{{ group.title }}</h3>
      </ion-label>
      <ion-button
        slot="end"
        fill="clear"
        [attr.aria-expanded]="!isCollapsed(group)"
        [attr.aria-controls]="'group-' + group.id"
        [attr.aria-label]="
          (isCollapsed(group) ? 'Expand ' : 'Collapse ') + group.title
        "
        (click)="toggleGroup(group)"
      >
        <ion-icon
          [name]="
            isCollapsed(group) ? 'chevron-down-outline' : 'chevron-up-outline'
          "
          aria-hidden="true"
        />
      </ion-button>
    </ion-item-divider>
    <div
      role="region"
      [id]="'group-' + group.id"
      [attr.aria-labelledby]="'heading-' + group.id"
      [hidden]="isCollapsed(group)"
    >
      <!-- the group's items -->
    </div>
  </ion-item-group>
</ion-list>
```

The native `ion-button` supports keyboard activation. Keep `aria-expanded`
synchronized with the visible state and make `aria-controls` identify the content
being shown or hidden. For the simpler static and list-level patterns, see
[`lists.md`](../lists.md).
