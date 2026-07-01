# Page Layout

Every page follows the same scaffolding: an `ion-header` with one `ion-toolbar`,
an `ion-content` holding a stack of cards, and an **optional** `ion-footer` for
aggregate actions. Detail and list pages differ only in what fills the content.

## Page scaffold

```html
<ion-header>
  <ion-toolbar color="light" class="with-back-button with-end-buttons">
    <ion-buttons slot="start">
      <ion-back-button [defaultHref]="$defaultBackUrl()" />
    </ion-buttons>
    <sneat-space-page-title [space]="$space()" generalTitle="…" />
    <ion-buttons slot="end">
      <!-- primary action(s) … then the menu button last -->
      <ion-menu-button />
    </ion-buttons>
  </ion-toolbar>
</ion-header>

<ion-content class="cardy">
  <!-- a stack of <ion-card>s -->
</ion-content>
```

Examples:
- `calendarius/.../pages/calendar/calendar-page.component.html` — list/browse
  page with navigation + "Add" actions + menu button.
- `contactus/.../pages/contact/contact-page.component.html` — detail page with a
  trash action in `slot="end"`.

## Toolbar

- **One `ion-toolbar`**, `color="light"` by default. Only diverge to
  `color="primary"` when the content genuinely needs the visual distinction (rare).
- **Back button** in `<ion-buttons slot="start">` as
  `<ion-back-button [defaultHref]="$defaultBackUrl()" />`. **Recommended:** use the
  `$defaultBackUrl()` signal rather than hard-coding a route.
- **Menu button** is the **last** element of `<ion-buttons slot="end">`, after any
  action buttons.
- **Action buttons** sit in `slot="end"` before the menu button, following
  [`buttons.md`](./buttons.md) and [`card-title-buttons.md`](./card-title-buttons.md)
  (icon-only by default, `color="primary"` for the primary "Add").
- Signal layout with toolbar classes: `with-back-button` when a back button is
  present, `with-end-buttons` when there are end-slot actions.

### Title — prefer `sneat-space-page-title`

For space-contextual pages use the shared `<sneat-space-page-title>` instead of a
plain `<ion-title>`, so the title reflects the active space (e.g. "Family
calendar" vs "Personal calendar"). Plain `<ion-title>` is fine for pages with no
space context.
*(calendarius / contactus page headers.)*

In practice most contactus pages still hand-roll a plain `<ion-title>` with
bespoke `@switch`/`@if` logic (only ~2 of 12 titled pages use
`sneat-space-page-title`) — so reaching for the shared component is a real
improvement over the current majority, not just codifying what's already there.

## Content

- `<ion-content class="cardy">` for card-based pages (forms, detail views) — this
  is the **Recommended** default.
- Plain `<ion-content>` for dense, list-only pages.
- Inside content, stack `ion-card`s — see [`cards.md`](./cards.md).

> **Reality check:** `cardy` is the *recommended* default but not yet the majority
> in practice — in contactus only ~4 of 15 page-level `ion-content` elements carry
> it, and trackus uses it nowhere. Add it to new/touched card-based pages; the
> inconsistency across existing pages is a known cleanup, not a signal that plain
> `ion-content` is equally fine for card layouts.

## Footer — only when it earns its place

Use an `ion-footer` only for actions or summaries that apply to the **whole
page**, not to a single card. Three established shapes:

| Footer kind | When | Example |
| --- | --- | --- |
| **Conditional selection bar** | Shown only while items are selected; bulk actions (share/archive/delete) in `slot="end"`, a reset in `slot="start"`, count in the title | `contactus/.../pages/contacts/contacts-page.component.html` |
| **Always-present summary** | A persistent total/count plus a primary "Add" | `contactus/.../members/pages/members/members-page.component.html` |
| **Dialog action row** | Confirm / cancel for a dialog page, often in an `ion-grid` | `listus/.../pages/dialogs/copy-list-items/copy-list-items-page.component.html` |
| **Persistent content-page action row** | Whole-page actions on a normal routed page (not a dialog) — e.g. bulk "Delete completed" / "Hide watched" | `listus/.../pages/list/list-page.component.html` |

The footer is **not** dialogs-only: listus's routed `list-page` carries a
persistent footer action row for its whole-list actions. The test is *scope*
(does the action apply to the whole page?), not *whether the page is a dialog*.

**Recommended:** don't add a footer to plain detail/edit pages — put their
submit/cancel in the form's own footer (see [`forms.md`](./forms.md)) or a card.

## Segments (in-page tabs)

`ion-segment` switches content within a page. Placement depends on scope:

- **Filtering a list within a card** → put the `ion-segment` inside the
  `ion-card`, above the list. *(listus `list-page.component.html` — All / Active /
  Completed.)* **Recommended** for filtering.
- **Switching the whole page layout** → place the `ion-segment` between
  `ion-header` and `ion-content`. *(contactus `new-member-page.component.html` —
  Person vs Pet.)*
- **Avoid** putting `ion-segment` inside the toolbar.
