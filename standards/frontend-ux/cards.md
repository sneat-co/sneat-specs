# Cards

`ion-card` is the primary content container. A page is a vertical stack of cards,
each grouping one coherent block (a list, a form, a detail section).

## NEVER a card inside a card

**MUST NOT.** A card never contains another card — no `ion-card` inside an
`ion-card`, and no element styled like a card (border + shadow/raised
background, e.g. a result/"action committed" panel) inside a card. Nested cards
double the borders, shadows and padding, blur which box owns the content, and
waste scarce mobile width.

**Exception — only with both of these:**

1. the user (founder/product owner) **explicitly asked** for the nested card, and
2. a **written justification** sits next to the code and in the PR, saying why
   none of the flat alternatives below can do the job:

   ```html
   <!-- ui-bible: nested-card — asked for by <who/when>; justification: <why a
        list/item/divider/section cannot do this> -->
   ```

A missing or vague justification ("looks nicer", "grouping") is a defect, not an
exception. Reviewers reject it.

**Use instead, inside the card:**

| Need | Use |
|---|---|
| A sub-group of related rows | `ion-item-divider` / a labelled `ion-list` section |
| A row with status or result | `ion-item` with `ion-icon slot="start"`, `ion-label`, `ion-note slot="end"` |
| A highlighted message/result | `ion-item` with a `color`, or an `ion-text color="…"` line |
| Key–value details | `ion-list` of `ion-item`s (`ion-label` + `ion-note slot="end"`) |
| Visual separation only | a divider line (`lines="full"`) or spacing — not a box |

**Wrong:**

```html
<ion-card>            <!-- "Done!" -->
  <ion-item-divider color="light"><ion-label>Done!</ion-label></ion-item-divider>
  <ion-card-content>
    <ion-card>…Action committed…</ion-card>   <!-- NEVER -->
  </ion-card-content>
</ion-card>
```

**Right:**

```html
<ion-card>
  <ion-item-divider color="light">
    <ion-icon name="checkmark-circle" color="success" slot="start" />
    <ion-label>Done!</ion-label>
  </ion-item-divider>
  <ion-list>… one ion-item per result row …</ion-list>
</ion-card>
```

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

**MUST.** Every `ion-card` starts with exactly one header row:
`<ion-item-divider color="light">` (or `warning` / `danger` for status cards, see
below), the title in an `ion-label`, any actions in `slot="end"`. The light
background is what makes cards read as one family.

```html
<ion-card>
  <ion-item-divider color="light">
    <ion-label>{{ title }}</ion-label>
    <ion-buttons slot="end">
      <ion-button color="medium" (click)="add($event)"><ion-icon name="add" /></ion-button>
    </ion-buttons>
  </ion-item-divider>
  <ion-card-content>…</ion-card-content>
</ion-card>
```

**MUST NOT:** a bare `ion-card-header` / `ion-card-title`, an `ion-card-title`
inside a plain `ion-item`, or a card with no header row. This applies to static
titles too, so adding a button later never changes the header's look. For
collapsible headers add `tappable`; see
[`card-title-buttons.md`](./card-title-buttons.md) for the button conventions.

**Header colour.** The divider is `color="light"` by default. Status, warning
and error cards use the same divider with `color="warning"` or `color="danger"`,
for example a "GitHub access expired" card:

```html
<ion-card>
  <ion-item-divider color="warning">
    <ion-label>GitHub access expired</ion-label>
    <ion-button slot="end" size="small">Reconnect</ion-button>
  </ion-item-divider>
  <ion-card-content>…</ion-card-content>
</ion-card>
```

The card itself takes no `color`. Any other divider colour is not allowed. The
media item card below is the only card with no header by design.

**Enforcement.** datatug-apps enforces this mechanically with
`pnpm run check:card-headers`; other repos can adopt the same checker.

## Media item card (poster + metadata + actions)

Some items are enriched from an external data source and need a card shaped
around a **thumbnail image + a metadata block + an action row**, rather than a
header/body split (the one card without a header row) — a movie (poster,
cast, trailer), a book (cover, author), a product (photo, price). This shape isn't a list card or a static-title card; it's
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
  not a header row. This is the documented exception to the
  [card header rule](#card-headers). There's no collapsible/actionable
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
- Every card: one `ion-item-divider` header, actions in `slot="end"`. Colour is
  `light` by default, or `warning` / `danger` for status and error cards. Any
  other colour is not allowed. (Media item card: no header, see above.)
