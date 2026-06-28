# Frontend UX Standards

House UX conventions for Sneat extensions, codified from the live frontends of
**calendarius**, **contactus**, and **listus**. Each rule cites a real example so
you can see it in context.

These are conventions, not a framework: extensions use **native Ionic** building
blocks (`ion-card`, `ion-list`, `ion-item`, `ion-button`, `ion-modal`, …)
directly — there is no custom card/button/modal wrapper component. Consistency
comes from following these patterns, not from a shared widget.

## Principles

1. **Wrap content in cards.** A page is a stack of `ion-card`s, each grouping a
   coherent block of content.
2. **Actions live in the header, end-aligned.** Put action buttons in
   `<ion-buttons slot="end">` of the card/section/page header.
3. **Icon-only by default, label only when it matters.** Header buttons are
   icon-only (`color="medium"`); add a text label only for a primary affordance
   like "Add".
4. **Colour carries meaning.** Use Ionic colours semantically
   (`primary`/`danger`/`success`/`warning`/`medium`) — never decoratively.
5. **Lists belong in cards.** Scrollable collections are `ion-list` inside a card,
   with sliding/grouping patterns as needed.
6. **Modern Angular control flow + signals.** Use `@if` / `@for` / `@empty` and
   `signal()`-backed state (`$data()`, `$isLoading()`), not `*ngIf` / `*ngFor` or
   RxJS in templates.
7. **Always handle the three transient states.** Every data-bound view shows a
   **loading**, **empty**, and **error** state — not just the happy path.

## Contents

### Building blocks

- [`cards.md`](./cards.md) — when and how to use `ion-card`, header styles.
- [`card-title-buttons.md`](./card-title-buttons.md) — action buttons in card /
  section headers.
- [`buttons.md`](./buttons.md) — colour, fill, expand, and icon conventions.
- [`lists.md`](./lists.md) — `ion-list`, sliding items, grouped/collapsible lists.

### Composition

- [`page-layout.md`](./page-layout.md) — page scaffolding: header/toolbar,
  `ion-content`, footers, segments, back/menu buttons.
- [`forms.md`](./forms.md) — inputs, labels, reactive forms, validation, submit.
- [`modals.md`](./modals.md) — modals, alerts, popovers, toasts, dialog structure.
- [`states.md`](./states.md) — loading, empty, and error states.

## How to read the citations

Each rule cites a real component using **repo-relative shorthand**:
`listus/.../pages/list/list-item.component.html`. The leading segment names the
sibling extension repo under `github.com/sneat-co/` (`calendarius`, `contactus`,
`listus`); `/.../` elides the intermediate
`frontend/libs/extensions/<id>/<tier>/src/lib/` path. Open the repo and search
for the trailing filename to find the live example.

## Where divergences exist

The three surveyed apps mostly agree. Where they differ, these docs **recommend
one convention and note the alternative** — they don't pretend a single style is
universal. Look for the **"Recommended"** callouts.

## Relationship to extension standards

These UX standards are one of the three pillars of the broader
[Sneat extension standards](https://github.com/sneat-co/sneat-libs/blob/main/docs/extension-standards/README.md)
(backend wiring · frontend apps · **UX**), which live in `sneat-libs`. The UX
pillar is maintained here because it spans every extension repo, not just
`sneat-libs`.
