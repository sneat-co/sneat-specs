# Cards

`ion-card` is the primary content container. A page is a vertical stack of cards,
each grouping one coherent block (a list, a form, a detail section).

## Use a card for each content block

Wrap each logical section — a list, a filter+segment+list combo, a detail
block — in its own `<ion-card>`. Use **native Ionic** cards; there is no custom
card wrapper component.

Examples:
- `listus/.../pages/lists/lists-page.component.html` — list groups wrapped in a card.
- `contactus/.../components/contacts-component/contacts.component.html` — a card
  wraps a filter, an `ion-segment`, and the contact list together.
- `contactus/.../components/contact-details/contact-details.component.html` — a
  detail block in a card.

## Card headers

There are two header styles in use. Pick by whether the header carries actions or
collapses.

### Recommended: `ion-item-divider` for actionable / collapsible headers

When the header has buttons or toggles a collapsible section, use an
`ion-item-divider` (or an `ion-item`) as the header row, with actions in
`slot="end"`:

```html
<ion-item-divider color="light" class="with-buttons" tappable>
  <ion-label>{{ listGroup.title }}</ion-label>
  <ion-buttons slot="end">
    <ion-button color="medium" (click)="addTo($event, listGroup.type)">
      <ion-icon name="add" />
    </ion-button>
  </ion-buttons>
</ion-item-divider>
```
*(listus `lists-page.component.html`)*

This is the dominant pattern for headers that **do** something (add, edit,
expand/collapse). See [`card-title-buttons.md`](./card-title-buttons.md) for the
button conventions inside these headers.

### Acceptable: `ion-card-header` for static titles

A plain `ion-card-header` + `ion-card-title` is fine for a **purely static**
title with no actions (as some contactus cards do). If you later add a button,
prefer switching to the `ion-item` / `ion-item-divider` form so the action sits
cleanly in `slot="end"`.

### Also seen: `ion-card-title` inside a plain `ion-item`

calendarius uses a third, hybrid header — an `<ion-card-title>` (for the type-scale
text) placed inside a plain `<ion-item>` (not `ion-card-header`), with the action
in `slot="end"` — and even documents the choice inline ("We intentionally do not
use ion-item-divider here"). *(calendarius
`happening-participants.component.html`.)* Acceptable when you want card-title
typography **and** a clean action slot; but prefer the `ion-item-divider` form
unless you specifically need the larger title text, to keep headers uniform.

## Media item card (poster + metadata + actions)

Some items are enriched from an external data source and need a card shaped
around a **thumbnail image + a metadata block + an action row**, rather than a
header/body split — a movie (poster, cast, trailer), a book (cover, author), a
product (photo, price). This shape isn't a list card or a static-title card; it's
its own layout, worth naming so it isn't reinvented per extension.

```html
<ion-card [class.watched]="$isDone()">
  <ion-card-content>
    <div class="movie-card-layout">
      @if (item.posterURL) {
        <img class="poster" [src]="item.posterURL" alt="{{ item.title }} poster"
          width="92" height="138" />
      } @else {
        <div class="poster poster-placeholder">
          <ion-icon name="film-outline" size="large" color="medium" />
        </div>
      }
      <div class="movie-card-body">
        <h2 class="ion-text-wrap">{{ item.title }}
          @if (item.year) { <ion-text color="medium">({{ item.year }})</ion-text> }
        </h2>
        @if ($watchWithLabel(); as watchWithLabel) {
          <ion-badge color="light">{{ watchWithLabel }}</ion-badge>
        }
        <!-- overview snippet, cast line: ion-text color="medium" -->
        <div class="movie-card-actions">
          <!-- trailer / mark-watched / delete ion-buttons -->
        </div>
      </div>
    </div>
  </ion-card-content>
</ion-card>
```
*(listus `pages/list/watch-movie-card/watch-movie-card.component.html`.)*

- **Layout:** a single `<div>` inside `ion-card-content` with `display: flex;
  gap: 0.75em` — thumbnail fixed-size (`flex: 0 0 auto`), metadata block
  flexible (`flex: 1 1 auto; min-width: 0` so long titles/overview text wraps
  and truncates instead of blowing out the card width).
- **No card header at all** — the title lives in the metadata block (an `h2`),
  not `ion-card-header`/`ion-item-divider`. There's no collapsible/actionable
  header row here; the whole card *is* the content.
- **Always give the thumbnail a placeholder branch** (`@if (item.posterURL) {
  img } @else { icon in a sized, `background: var(--ion-color-light)` box)` —
  don't let a missing external-source image collapse the layout or leave a
  broken-image icon.
- **Image `alt` is the item's title** (`alt="{{ item.title }} poster"`), never
  empty or generic — the image is content, not decoration.
- **Secondary metadata** (a provenance/context chip, a truncated description, a
  one-line list of related names) renders as `ion-badge`/`ion-text
  color="medium"` rows below the title, each conditionally rendered only when
  the data is present — don't reserve empty space for absent fields.
- **Actions live in a wrapping row at the bottom of the metadata block**
  (`display: flex; flex-wrap: wrap; gap: 0.25em`), sized `size="small"`, mixing
  a `fill="clear"` link-like action (open the external source), a toggle button
  whose `fill`/`color` reflects state (see [`buttons.md`](./buttons.md)'s
  state-reactive button pattern), and an icon-only destructive action with
  `aria-label` — not a header slot, since this card has no header.
- **State it toggled on** (e.g. "watched") gets a card-level modifier class
  (`[class.watched]="$isDone()"`) that dims the thumbnail (`opacity: 0.6`)
  rather than hiding or moving the card.

**Recommended:** reach for this shape whenever a card's primary content is an
externally-sourced image plus structured metadata — don't force it into the
header/list-item conventions above, which assume the card's content is
internally-authored text.

## Summary

- One `ion-card` per content block; native Ionic only.
- Header with actions/collapse → `ion-item-divider` (or `ion-item`), actions in
  `slot="end"`. **Recommended.**
- Static title, no actions → `ion-card-header` is acceptable.
- Need card-title typography + an action → `ion-card-title` inside an `ion-item`
  is an accepted hybrid (calendarius), but don't reach for it by default.
- Poster/cover/photo + structured metadata + actions, no header → the media
  item card (above): thumbnail + placeholder, flex layout, actions row at the
  bottom.
