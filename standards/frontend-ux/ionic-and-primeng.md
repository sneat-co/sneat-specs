# Ionic & PrimeNG in One App

**Ionic is primary. PrimeNG is a guest.** Every Sneat app surface is built from
native Ionic building blocks (`ion-card`, `ion-list`, `ion-modal`, …) exactly as
the rest of this folder describes. PrimeNG is added only where Ionic has **no**
component at all — org charts / play-off brackets, chat surfaces, trees, rich
data tables — and only inside a declared, bounded **island**.

*Founder decision, 2026-08-19. This document records how the two coexist and how
they stay visually in unison; it does not reopen the choice.*

> **State of the fleet (verified 2026-08-19):** no Sneat app runs Ionic and
> PrimeNG in the same Angular application today. `competios/frontend` is the only
> PrimeNG code we have (v21.1.9, unstyled, Angular 21.2.19 SSR) and it contains
> **zero** Ionic — the whole repo has no `@ionic/*` dependency. Everything in
> §3 (overlays) and §8 (focus) below is therefore derived from reading both
> libraries' shipped source — `@ionic/core` 8.8.6 and `primeng` 21.1.9 — not from
> a running mixed app. **The first app to mix them is a pilot**; treat the pilot
> as the thing that confirms this document, and correct it here when it doesn't.
>
> The Ionic gaps claimed in §1.3 were re-checked against Ionic's live components
> page on 2026-08-19: Ionic has no first-class component for an org chart, tree,
> data grid, chat surface, bracket, colour picker, file upload, rich-text editor
> or chart. `ion-reorder` is list reordering, not general drag-and-drop.

---

## 1. The decision rule

Apply in order. Stop at the first rule that fires.

1. **Does Ionic have a component for it?** → **Use Ionic.** Not "does Ionic have
   a *better* one" — does it have one at all. `ion-datetime` beats `p-datepicker`
   by rule, not by merit.
2. **Does a Sneat library already have it?** → **Use that.** `@sneat/grid`,
   `@sneat/datagrid`, `@sneat/wizard`, `sneat-contacts-selector-input`, and the
   `@sneat/extension-*-ui` packages exist precisely so we don't re-buy controls.
   Reuse through the published package, don't reimplement.
3. **Is it on the founder's named list** — org chart / play-off bracket, chat
   surface — **or does it clear the bar in §1.1?** → **PrimeNG is allowed**,
   inside an island (§2), subject to the never-owns list (§1.2).
4. **Otherwise** → build it, in the owning extension's `-ui` package or in
   `@sneat/components`. **Never add a third UI library.**

### 1.1 The bar, for controls not on the founder's list

All four must hold:

- Ionic ships **nothing** for it (check the current Ionic components list, not
  your memory);
- it is a **data or visualisation surface** — a tree, a table, a bracket, a
  timeline, a chart, a virtualised log — not chrome, not navigation, not a
  dialog, not a text input;
- the realistic alternative is **writing it ourselves**, and that is more than
  about two days of work;
- it fits inside an island (§2) without the island having to own an overlay,
  a route, or the page header.

If you'd have written a bespoke component anyway, PrimeNG's is a better bespoke
component than yours. If you were going to use an Ionic component and PrimeNG's
merely looks nicer, the bar is not met.

### 1.2 What PrimeNG never owns — no exceptions

- **Page chrome**: header, toolbar, back button, menu button, footer. Always
  `ion-header` / `ion-toolbar` / `ion-footer` (see [`page-layout.md`](./page-layout.md)).
- **Navigation**: menus, tabs, segments, breadcrumbs. Always `ion-menu`,
  `ion-tabs`, `ion-segment`.
- **Overlays**: modals, popovers, alerts, confirms, toasts, loading. Always the
  Ionic controllers (see §3 and [`modals.md`](./modals.md)).
- **The submit affordance** of a form. Always `ion-button` (see [`buttons.md`](./buttons.md)).
- **Routing.** A PrimeNG component is never a routed page component.

### 1.3 Worked cases

| Need | Use | Why |
| --- | --- | --- |
| Play-off bracket / org chart | **PrimeNG** `p-organizationchart` | Founder-named. Ionic has nothing. See the CSP caveat in §11. |
| Chat surface | **Ionic shell + PrimeNG parts** | Founder-named — but read the honesty note below: **PrimeNG has no chat component.** The message list is `ion-list`/`ion-content`; PrimeNG contributes `p-scroller` (virtual scroll over long history), `p-autocomplete` (@-mentions), `p-editor` (rich compose). The composer's send button is `ion-button`. |
| Rich data grid (sort / filter / freeze / group) | `@sneat/datagrid` or `@sneat/grid` first; **PrimeNG** `p-table` only if those genuinely can't | Rule 2 before rule 3. Ionic has no grid. |
| Tree / hierarchy browser | **PrimeNG** `p-tree` / `p-treetable` | Ionic has nothing (`ion-accordion` is not a tree). |
| Date / time picker | **Ionic** `ion-datetime` | Ionic has one. Never `p-datepicker`. |
| Modal / dialog | **Ionic** `ModalController` + `SneatBaseModalComponent` | §3. Never `p-dialog`. |
| Confirm | **Ionic** — native `confirm()` per [`modals.md`](./modals.md), or `AlertController` | Never `p-confirmdialog` / `p-confirmpopup`. |
| Toast / transient feedback | **Ionic** `ToastController` | Never `p-toast`. |
| Context menu | **Ionic** `PopoverController` | Never `p-contextmenu` / `p-menu` / `p-popover`. |
| Text / number / select / checkbox input | **Ionic** `ion-input` / `ion-select` / `ion-checkbox` | §6. Never `pInputText` / `p-select` in an Ionic app. |
| Side menu / drawer | **Ionic** `ion-menu` | Never `p-drawer`. |
| Tabs / in-page segments | **Ionic** `ion-tabs` / `ion-segment` | Never `p-tabs`. |
| Autocomplete over remote data | **Ionic** `ion-searchbar` + list, unless the island genuinely needs typeahead-with-templating → **PrimeNG** `p-autocomplete` | Prefer Ionic; this is the one input-shaped exception, and only inside an island. |
| Chart | **PrimeNG** `p-chart`, or Chart.js directly | Ionic has nothing. Pick one and only one charting stack fleet-wide. |
| Rich-text editor | **PrimeNG** `p-editor` (Quill) | Ionic has nothing. Island only. |
| File upload | **PrimeNG** `p-fileupload` | Ionic has nothing beyond a bare `<input type="file">`. Island only; the surrounding form stays Ionic. |
| Colour picker, OTP input, knob, rating, terminal, galleria | **Stop and ask.** | Ionic has nothing, but none of these is a data surface — they fail the §1.1 bar. If you need one, it is a founder call, not an agent call. |

> **Honesty note on "chat interfaces".** The founder's decision names chat as a
> PrimeNG case, and it stands — but PrimeNG **21.1.9 ships no chat component**
> (verified by enumerating every module in the published package: there is no
> `chat`, no `messages` thread widget, no `mention`). What PrimeNG actually gives
> a chat surface is the three parts in the table above. Budget the chat screen as
> *"an Ionic screen with three PrimeNG parts"*, not as *"PrimeNG has a chat
> widget"*, or the estimate will be wrong.

---

## 2. Scoping — islands, not interleaving

**The two libraries are scoped by route or feature area, never mixed
widget-by-widget.**

### What an island is

A **PrimeNG island** is a single Angular component that:

- is imported by exactly one Ionic host component;
- renders **inside** an `ion-content` (usually inside one `ion-card`) — never
  outside the Ionic page scaffold;
- imports PrimeNG modules in its own `imports:` array, and **is the only file in
  its feature that does so**;
- takes plain data in via `input()` and emits plain domain events out via
  `output()` — no PrimeNG types cross its boundary (no `TreeNode`, no
  `TableLazyLoadEvent`, no `MenuItem` in a service, a store, or a contract
  package);
- owns no overlay, no route, no page chrome, no navigation, no submit (§1.2).

```
ion-content.cardy
└── ion-card
    ├── ion-item-divider  (header + ion-buttons slot="end")   ← Ionic owns this
    └── <sneat-bracket-chart [rounds]="$rounds()"             ← the island
                             (matchSelected)="openMatch($event)" />
                                                    ↑ emits a domain event;
                                                      the HOST opens an ion-modal
```

The island renders the bracket. When a match is clicked it emits
`matchSelected: MatchId`. The **host** — an Ionic component — opens the
`ion-modal`. The island never calls a controller of any kind.

### The view-model boundary

Adapt domain data to PrimeNG's shapes in a **pure, unit-tested adapter function**
beside the island, not in a service and not in the domain model. competios does
exactly this: `single-elimination-tree.adapter.ts` turns a `FormatScenario` into
`TreeNode[]` and is the only file outside the component that touches a PrimeNG
type — and it imports it `import type`, so nothing of PrimeNG survives into the
JS bundle from that file.

Its own approved feature spec states the rule in the same terms:

> "PrimeNG supplies the shared interactive components […] but domain projections
> do not depend on PrimeNG types. A view-model adapter selects a renderer by
> competition format."
> — `competios/spec/features/angular-web-application/README.md`

### Where the code lives

`@sneat/prime-*` libraries live in **`sneat-libs`, under `libs/prime/*`** (founder
decision, 2026-08-19) — not a new repo, and not inside an extension. One library
per island family (`@sneat/prime-bracket`, `@sneat/prime-chat`, …), each
declaring `primeng` as a **peer dependency**, never a hard dependency.

This is consistent with where `sneat-libs` is heading: it is shedding
`libs/extensions/*` into the per-extension repos and keeping platform-level
libraries. `libs/prime/*` is platform-level.

### The rule you will be tempted to break

**One island per screen area.** A screen with a PrimeNG tree in one card and a
PrimeNG table in another card is two islands, and that is fine. A screen with a
`pInputText` inside an `ion-item` next to an `ion-input` is **interleaving**, and
is forbidden — it produces two label systems, two validation renderings, two
focus rings and two disabled states in one form (§6, §8).

---

## 3. Overlays — one owner per flow

**Every overlay in an Ionic app is an Ionic overlay.** This is not a style
preference; the two overlay systems are mutually hostile in four separate,
verifiable ways.

### The four collisions

Values below are read from the shipped source of `@ionic/core` 8.8.6 and
`primeng` 21.1.9.

**1. Z-index — PrimeNG loses by a factor of eighteen.**

| Stack | Overlay | z-index |
| --- | --- | --- |
| Ionic | `ion-modal`, `ion-alert`, `ion-popover`, `ion-action-sheet` | `20000 + overlayIndex` |
| Ionic | `ion-loading` | `40000 + overlayIndex` |
| Ionic | `ion-toast` | `60000 + overlayIndex` |
| PrimeNG | `modal` (`p-dialog`, `p-confirmdialog`) | `1100` |
| PrimeNG | `overlay`, `menu` (`p-select`, `p-popover`, `p-datepicker`) | `1000` |
| PrimeNG | `tooltip` | `1100` |

*(Ionic: each overlay's compiled `render()` puts an inline
`style.zIndex = 20000 + this.overlayIndex` on the Host element; `overlayIndex`
increments from 1 per presented overlay. PrimeNG: the `zIndex` defaults on the
`PrimeNG` config service, applied via
`ZIndexUtils.set('modal', container, this.baseZIndex + this.config.zIndex.modal)`.)*

A `p-dialog` opened while any Ionic overlay is presented renders **behind** it.
The user sees the backdrop darken and nothing else.

> **Don't be misled by Ionic's SCSS.** Ionic's `ionic.globals.scss` declares
> `$z-index-overlay: 1001`, and `alert.scss` / `popover.scss` / `toast.scss` do
> emit `:host { z-index: 1001 }`. That number is a red herring: the **inline**
> style each overlay writes on its own Host at render time overrides it, so the
> effective value is always the 20000/40000/60000 series. Reading only the
> stylesheet leads to the exact opposite conclusion — that PrimeNG's 1100 sits
> *above* Ionic — and it is wrong.

**Ionic has no runtime z-index configuration.** PrimeNG's is a config object;
Ionic's are Sass build-time variables plus those inline styles, and `IonicConfig`
exposes no `zIndex` key. So the only lever available is raising PrimeNG — which
is why the escape hatch below points that way and never the other.

**2. Escape is handled twice, at document level, by both.** Ionic installs a
document `keydown` listener on first overlay creation that dismisses the
last presented Ionic overlay on `Escape` — **with no check of where the event
came from**. PrimeNG's `p-dialog` binds its own document escape listener. With
both open, one `Escape` closes both.

**3. Focus trapping fights.** Ionic installs a capturing document `focus`
listener and places sentinel elements around the presented overlay's wrapper;
tabbing past the end of an `ion-modal` wraps you back to its first focusable
element. PrimeNG's `p-dialog` defaults to `focusTrap = true` and installs its own
sentinels. Two traps, two rings: the user cannot Tab from an `ion-modal` into a
`p-dialog` that was appended outside it, and cannot Tab back.

**4. PrimeNG's scroll lock is inert inside Ionic.** `p-dialog`'s `blockScroll`
adds `.p-overflow-hidden` to `document.body`. In an Ionic page the scroller is
**not** the body — it is `.inner-scroll` inside `ion-content`'s shadow root
(`overflow-y: var(--overflow)`). The page keeps scrolling behind the dialog.

### The rules

- **Only Ionic presents overlays.** `ModalController`, `PopoverController`,
  `ToastController`, `AlertController`, native `confirm()`. Never `p-dialog`,
  `p-confirmdialog`, `p-confirmpopup`, `p-toast`, `p-drawer`, `p-popover`,
  `p-contextmenu`, `p-menu`, `DynamicDialog`.
- **An island never presents anything.** It emits; the Ionic host presents.
- **A PrimeNG island may live *inside* an `ion-modal`.** That is the supported
  direction: Ionic outside, PrimeNG inside. The reverse is never allowed.
- **Small attached overlays that PrimeNG components own internally** —
  `p-autocomplete`'s suggestion panel, `p-select`'s panel — are tolerated **only
  because PrimeNG v21 defaults `overlayAppendTo` to `'self'`**, so the panel
  renders in place, inside the Ionic DOM and stacking context, rather than being
  teleported to `document.body`.
  **Never set `appendTo="body"`, and never set `providePrimeNG({ overlayAppendTo: 'body' })`.**
  That single setting reintroduces all four collisions above.
- **Hardware back button on Android: Ionic only.** Ionic registers overlay
  dismissal against the `ionBackButton` event. PrimeNG overlays are invisible to
  that system, so a hardware back press with a `p-dialog` open dismisses or
  navigates away from the Ionic page *behind* it and orphans the dialog. This
  alone makes `p-dialog` unusable in a mobile Sneat app.
- **If you ever have to raise PrimeNG above Ionic** (you shouldn't — it means an
  island is owning an overlay), do it once, globally, and write down why:
  ```ts
  providePrimeNG({ unstyled: true, zIndex: { modal: 30000, overlay: 30000, menu: 30000, tooltip: 30000 } })
  ```
  This still does not fix Escape, focus, scroll lock or the back button.

### Forward-looking hazard: the browser Top Layer

Angular CDK 22 moved its overlays onto the **native Popover API / Top Layer**.
Elements in the Top Layer paint above all normal DOM content **regardless of
z-index** — there is an open PrimeNG issue about exactly this breaking
`appendTo="body"` overlays. Ionic overlays are `position: fixed` + z-index too,
so the same thing would happen to them. If a third library on the page (or a
future Ionic/PrimeNG version) adopts `<dialog>` / `popover`, every number in the
table above stops mattering and the last-opened Top Layer element wins. Nothing
to do today; re-read this section the next time either library has a major bump.

---

## 4. Theming in unison

One vocabulary, two adapters. `@sneat/design-tokens` is the single source of
brand truth (founder decision, 2026-08-19), seeded from the existing Astro
landings vocabulary.

### 4.1 Where we are starting from — five disconnected vocabularies

| Vocabulary | Where | Shape |
| --- | --- | --- |
| `--color-*` (14 tokens) | `sneat-astro/src/styles/contract.css`, ~40 landings | `accent`, `accent-soft`, `accent-strong`, `accent-text`, `bg`, `bg-2`, `bg-3`, `border`, `danger`, `soft`, `surface`, `text`, `text-faint`, `text-muted` — light only, no dark block |
| `--ion-color-*` | `sneat-libs/libs/components/src/theme/variables.scss` | Six variables per semantic colour × 9 colours |
| PrimeNG | competios `styles.scss` | None — hand-written CSS against bespoke classes |
| `--paper-*` | `competios/frontend/src/styles.scss` | `--paper-bg`, `--paper-ink`, `--paper-rule`, `--accent`, … |
| `--bg-* / --accent-* / --text-*` | `gametable/web/src/style.css` | `--accent-amber`, `--accent-cyan`, `--bg-dark`, `--border-subtle`, `--font-sans` — dark only |

> **Reality check, and it is worse than "they disagree":**
> `sneat-libs/libs/components/src/theme/variables.scss` is the **unmodified Ionic
> CLI scaffold palette** — `--ion-color-primary: #3880ff`, stock Ionic blue — and
> its entire `@media (prefers-color-scheme: dark)` block is **commented out**.
> Every Ionic Sneat app today ships default Ionic blue with no dark mode. There
> is no brand theme to preserve; §4.2 is the first one.

### 4.2 The token package

`@sneat/design-tokens` publishes plain CSS (no build step, no SCSS, consumable by
Angular, Astro and a Worker alike) in three files:

- `tokens.css` — the semantic vocabulary, light and dark;
- `ionic.css` — the Ionic adapter;
- `primeng.css` — the PrimeNG layer.

It **keeps the Astro `--color-*` names** so landings and apps share literally the
same custom properties, and adds three things the Astro contract lacks:

1. **`--color-success` and `--color-warning`.** [`buttons.md`](./buttons.md) and
   [`states.md`](./states.md) use `color="success"` / `color="warning"`
   semantically across every app; the 14-token Astro contract has no equivalent,
   so those two must be added rather than improvised per app.
2. **`-rgb` companions** for every colour Ionic needs as a triple. Ionic renders
   hover/activated/focus states as `rgba(var(--ion-color-X-rgb), 0.14)`, and a
   bare `r, g, b` list cannot be derived from a hex in CSS. It must be authored.
3. **A dark block**, keyed to Ionic 8's `.ion-palette-dark` class so one toggle
   flips both stacks (§4.5).

```css
/* @sneat/design-tokens/tokens.css */
:root {
  /* Surfaces */
  --color-bg: #ffffff;
  --color-bg-2: #f6f5f3;
  --color-bg-3: #ecebe7;
  --color-surface: #ffffff;
  --color-soft: #f6f5f3;
  --color-border: #e2e0db;

  /* Ink */
  --color-text: #1a1a1a;            --color-text-rgb: 26, 26, 26;
  --color-text-muted: #55554f;      --color-text-muted-rgb: 85, 85, 79;
  --color-text-faint: #767670;

  /* Accent */
  --color-accent: #2f6f6a;          --color-accent-rgb: 47, 111, 106;
  --color-accent-strong: #21514d;   --color-accent-strong-rgb: 33, 81, 77;
  --color-accent-soft: #d7e6e4;
  --color-accent-text: #ffffff;     --color-accent-text-rgb: 255, 255, 255;

  /* Status */
  --color-danger: #c0392b;          --color-danger-rgb: 192, 57, 43;
  --color-success: #2f8f4e;         --color-success-rgb: 47, 143, 78;
  --color-warning: #b8791a;         --color-warning-rgb: 184, 121, 26;

  /* Type, geometry — unchanged from the Astro contract */
  --font-sans: ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-body: var(--font-sans);
  --font-display: var(--font-sans);
  --radius: 14px;  --radius-sm: 8px;  --radius-pill: 999px;
  --shadow-sm: 0 2px 8px rgba(20, 20, 20, 0.08);
  --shadow: 0 14px 40px -12px rgba(20, 20, 20, 0.22);
  --ease: cubic-bezier(0.22, 0.61, 0.36, 1);
  --space-1: .5rem; --space-2: 1rem; --space-3: 1.5rem; --space-4: 2.5rem;
}
```

### 4.3 The Ionic adapter — six variants from one token

Ionic wants six variables per semantic colour. Two are authored (`base`,
`base-rgb`), two are the contrast pair, and **two are derived in CSS**:

Ionic's own defaults are exactly a **12 % black mix** for `shade` and a **10 %
white mix** for `tint` — check it against the stock palette: `#3880ff × 0.88 =
#3171e0` (Ionic's `--ion-color-primary-shade`), and `#3880ff` mixed 90/10 with
white `= #4c8dff` (Ionic's `--ion-color-primary-tint`). Both land on the exact
published hex, so `color-mix()` reproduces Ionic's derivation rather than
approximating it.

```css
/* @sneat/design-tokens/ionic.css */
:root {
  /* primary ← accent */
  --ion-color-primary: var(--color-accent);
  --ion-color-primary-rgb: var(--color-accent-rgb);
  --ion-color-primary-contrast: var(--color-accent-text);
  --ion-color-primary-contrast-rgb: var(--color-accent-text-rgb);
  --ion-color-primary-shade: color-mix(in srgb, var(--color-accent) 88%, black);
  --ion-color-primary-tint:  color-mix(in srgb, var(--color-accent) 90%, white);

  /* danger ← danger */
  --ion-color-danger: var(--color-danger);
  --ion-color-danger-rgb: var(--color-danger-rgb);
  --ion-color-danger-contrast: #ffffff;
  --ion-color-danger-contrast-rgb: 255, 255, 255;
  --ion-color-danger-shade: color-mix(in srgb, var(--color-danger) 88%, black);
  --ion-color-danger-tint:  color-mix(in srgb, var(--color-danger) 90%, white);

  /* …secondary ← accent-strong, success ← success, warning ← warning,
       medium ← text-muted, light ← bg-2, dark ← text — same six-line shape … */

  /* Page-level Ionic surfaces */
  --ion-background-color: var(--color-bg);
  --ion-background-color-rgb: var(--color-bg-rgb);
  --ion-text-color: var(--color-text);
  --ion-text-color-rgb: var(--color-text-rgb);
  --ion-border-color: var(--color-border);
  --ion-font-family: var(--font-body);
}
```

The complete mapping:

| Ionic semantic | Sneat token | Used for |
| --- | --- | --- |
| `primary` | `--color-accent` | affirmative primary actions |
| `secondary` | `--color-accent-strong` | secondary emphasis |
| `tertiary` | `--color-accent-soft` | rare; decorative accents |
| `success` | `--color-success` | positive confirmation |
| `warning` | `--color-warning` | caution / archive |
| `danger` | `--color-danger` | destructive |
| `medium` | `--color-text-muted` | secondary/neutral text and buttons |
| `light` | `--color-bg-2` | de-emphasised toolbars, dividers |
| `dark` | `--color-text` | high-contrast text |

This table is what makes [`buttons.md`](./buttons.md)'s "colour carries meaning"
rule mean the same thing in both libraries.

- **`color-mix()` is Baseline-available** (Chrome 111 / Safari 16.2 / Firefox
  113, all 2023). **Not yet verified by us:** that `color-mix()` resolves
  correctly when substituted into Ionic's Stencil shadow DOM on Safari. Custom
  properties inherit through shadow boundaries and `color-mix()` is a valid
  `<color>`, so it should — **confirm it in the pilot before this ships**, and if
  it does not, fall back to authoring `-shade`/`-tint` hexes in `tokens.css`.

### 4.4 The PrimeNG layer — tokens *are* the styling

This is the fact that makes the whole arrangement work, and it is the most
important sentence in this document:

**PrimeNG runs `unstyled: true`, so it ships no CSS.** There is no PrimeNG theme
to fight the Ionic theme with, because there is no PrimeNG theme.

```ts
// competios/frontend/src/app/app.config.ts — the verified reference
providePrimeNG({ unstyled: true }),
{ provide: BaseStyle, useClass: PrimeNgExternalBaseStyle },
```

> **The flag's own type declaration disclaims it.** In `primeng@21.1.9`,
> `PrimeNGConfigType.unstyled` carries the JSDoc
> `@experimental — This property is not yet implemented. It will be available in
> a future release.` The runtime **does** implement it — `setConfig()` contains
> `if (unstyled) this.unstyled.set(unstyled)`, and `BaseComponent` reads
> `config?.unstyled()` as its fallback — so the comment is stale, not accurate.
> Don't let someone "fix" the config by deleting the flag on the strength of that
> JSDoc, and don't cite the JSDoc as evidence the mode is unsupported. It is the
> load-bearing setting of this entire standard; competios ships on it.

Reading `primeng@21.1.9`'s `BaseComponent` source, unstyled mode gives you and
denies you exactly this:

**Suppressed in unstyled mode:**
- `_loadCoreStyles()` — the component's structural CSS is never injected;
- `_loadThemeStyles()` — returns immediately, so no design-token CSS, no
  `@primeuix/themes` preset, no `--p-*` variables;
- `cx(key)` — returns `undefined`, so **no `p-*` class names are emitted at all**
  (the host binding is `class: "cn(cx('root'), styleClass)"`, and only
  `styleClass` survives).

**Still emitted / still active in unstyled mode:**
- **One three-rule base stylesheet**, injected at runtime by the *first* PrimeNG
  component: `.p-hidden-accessible`, `.p-hidden-accessible input, select`, and
  `.p-overflow-hidden`. `loadBaseCSS()` is called with no unstyled guard. This is
  why competios provides `PrimeNgExternalBaseStyle` — a `BaseStyle` subclass
  whose `loadBaseCSS` is a no-op — and reproduces those three rules by hand in
  its external stylesheet, so the app satisfies a `style-src` CSP with no
  `unsafe-inline`. **Copy that pattern.**
- `data-pc-name="<component>"` on the root and `data-pc-section="<part>"` on
  every internal part — always, unconditionally.
- `data-p="<state tokens>"` on components that expose state (e.g.
  `invalid focus filled fluid`).
- All behaviour: keyboard navigation, ARIA, focus trap, `ZIndexUtils`, portals,
  virtual scroll, filtering, sorting.

**So the styling hooks, in order of preference:**

1. **`styleClass` / `class` on the component root**, styled by your own CSS.
   Most stable — it is a public input.
2. **`pt` (PassThrough)** to attach your own classes to inner sections. A public
   API; explicit; survives PrimeNG internals changing.
3. **The element selector** — `p-organizationchart { … }`. The custom tag is
   always in the DOM.
4. **`[data-pc-section="…"]`** attribute selectors. They work, but the section
   names are PrimeNG-internal and can change on a minor.
5. **`.p-*` class selectors — never.** They are not emitted in unstyled mode, so
   the rule is silently dead. competios has a live example of this mistake: its
   `styles.scss` carries both `.event-form p-inputnumber` (an *element* selector
   — works) and `.event-form .p-inputnumber` (a *class* selector — dead).

Every island stylesheet is written against `@sneat/design-tokens` custom
properties, never against literal colours:

```css
/* @sneat/design-tokens/primeng.css — shared island primitives */
.sneat-prime-surface {
  background: var(--color-surface);
  color: var(--color-text);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  font-family: var(--font-body);
}
.sneat-prime-node {          /* p-organizationchart node, via styleClass */
  background: var(--color-bg-2);
  color: var(--color-text);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  padding: var(--space-1) var(--space-2);
}
.sneat-prime-node[data-p~="selected"] {
  background: var(--color-accent-soft);
  border-color: var(--color-accent);
}
```

Because the Ionic adapter derives `--ion-color-primary` from `--color-accent` and
the island CSS reads `--color-accent` directly, a brand change is one edit in
`tokens.css` and both stacks move together. That is what "in unison" means here.

### 4.5 Light and dark — one switch

Ionic 8 drives dark mode with the **`.ion-palette-dark` class on the root
element** (the old `body.dark` convention is gone; `@ionic/core` ships
`css/palettes/dark.class.css`, `dark.system.css` and `dark.always.css`). Use the
**class** variant so a single toggle is authoritative for both libraries, and
bootstrap it from the system preference:

```css
/* @sneat/design-tokens/tokens.css — dark block */
:root.ion-palette-dark {
  --color-bg: #14161a;
  --color-bg-2: #1b1e24;
  --color-bg-3: #23272f;
  --color-surface: #1b1e24;
  --color-soft: #23272f;
  --color-border: #333944;

  --color-text: #f2f3f5;            --color-text-rgb: 242, 243, 245;
  --color-text-muted: #a8adb6;      --color-text-muted-rgb: 168, 173, 182;
  --color-text-faint: #7d838d;

  --color-accent: #5fb3ac;          --color-accent-rgb: 95, 179, 172;
  --color-accent-strong: #86ccc5;   --color-accent-strong-rgb: 134, 204, 197;
  --color-accent-soft: #21403d;
  --color-accent-text: #0d1a19;     --color-accent-text-rgb: 13, 26, 25;

  --color-danger: #ef7b6f;          --color-danger-rgb: 239, 123, 111;
  --color-success: #58c47c;         --color-success-rgb: 88, 196, 124;
  --color-warning: #e0a94a;         --color-warning-rgb: 224, 169, 74;
}
```

`ionic.css` and every island stylesheet reference the tokens, never the hexes, so
they need **no dark-mode rules of their own**. Do not import
`@ionic/angular/css/palettes/dark.class.css` — it would reimpose stock Ionic
dark blue over the brand tokens. `@sneat/design-tokens` replaces it.

Toggle in one place:

```ts
document.documentElement.classList.toggle('ion-palette-dark', prefersDark);
```

**Anti-pattern:** a `@media (prefers-color-scheme: dark)` block *and* a class
toggle both defining tokens. One of them wins per-property in ways nobody can
predict from reading either file. Pick the class; derive the class from the media
query once, in TypeScript.

---

## 5. Ionic `mode`

Ionic renders two design languages — `ios` and `md` — and picks per device at
runtime. That platform-adaptive chrome is a large part of why Ionic "feels
Apple/Android", and it is a real reason Ionic is primary.

**Verified today:** no Sneat app pins `mode`. The single wiring point is
`sneat-libs/libs/app/src/lib/get-standard-sneat-providers.ts`, which calls a bare
`provideIonicAngular()`. Also verified: the current per-extension apps
(`togethered`, `calendarius`, `contactus`, …) depend on `@capacitor/core` only
for the Firebase auth plugin and have **no native iOS/Android projects** — only
the legacy `sneat-apps` monorepo has an `ios/` directory. Today these ship as web
/ PWA.

### Recommended

**Pin `mode: 'md'` in any app that hosts a PrimeNG island. Leave pure-Ionic apps
on the platform-adaptive default.**

```ts
// only in an app that hosts a PrimeNG island
provideIonicAngular({ mode: 'md' }),
```

**Why:** an unstyled PrimeNG island can only look like *one* thing — it is styled
once, from tokens. If the Ionic chrome around it flips to `ios` on an iPhone
while the island stays fixed, the seam is visible exactly where the user is
looking: rounded-vs-square controls, different type scale, different toolbar
height, a chevron back button instead of an arrow. With `md` pinned you have one
target to design the island against, and deterministic e2e screenshots.

**The trade-off, stated plainly:** on an iPhone the app stops looking like an
iPhone app. `ion-back-button`, `ion-alert` layout, `ion-toggle`, `ion-segment`,
`ion-refresher` and the sheet-modal feel all change. If we later ship a real
native iOS shell for one of these apps, revisit this for that app — a native app
that looks like Android is a worse trade than a visible seam on one card.

**Do not** pin `mode` fleet-wide in `getStandardSneatProviders()` as a side
effect of this document. That is a large visual change to every existing app and
is a founder call, not an agent call.

---

## 6. Forms

Forms are the place interleaving is most tempting and most damaging.
[`forms.md`](./forms.md) is unchanged and remains authoritative: one control per
`ion-item`, reactive forms, `labelPlacement="stacked"`, errors as
`ion-label color="danger"` shown only after `touched`.

**The rule: a form is single-library.** If a screen has a form, the form is
Ionic — labels, inputs, hints, errors, submit. A PrimeNG island may sit *beside*
the form inside the same page, in its own card. It may not sit *inside* it.

Where an island genuinely needs an input of its own (a `p-autocomplete` filter
over a tree, a `p-table` column filter), it is **island-internal state**, not a
control of the page's `FormGroup`. It never receives a `formControlName`.

### If you nonetheless must bind a PrimeNG control to the form

This happens for exactly one shape: an island-scoped picker whose value is part
of the submitted record. Then:

- Bind with `[formControl]` — every PrimeNG input implements
  `ControlValueAccessor`, so reactive forms work identically to Ionic's
  (which use Ionic's own `ValueAccessor`). Do not reach for `ngModel`;
  [`forms.md`](./forms.md) already forbids mixing.
- **Render the label, hint and error yourself**, with the *Ionic* markup, so the
  presentation is identical to every other field on the page:

  ```html
  <ion-item>
    <ion-label position="stacked">Opponent</ion-label>
    <p-autocomplete [formControl]="opponent" [suggestions]="$suggestions()"
                    styleClass="sneat-prime-input" appendTo="self" />
  </ion-item>
  @if (opponent.touched && !opponent.valid && !!opponent.errors) {
    <ion-label color="danger">Opponent is required</ion-label>
  }
  ```

  The `ion-item` + stacked `ion-label` + `color="danger"` message are exactly the
  forms.md pattern; only the control between them differs.
- **Never** use `p-floatlabel`, `p-iftalabel`, `p-message severity="error"`, or
  PrimeNG's `invalid` styling for validation display. Two error presentations on
  one page is the defect this section exists to prevent.
- The same reasoning applies *within* Ionic: `ion-input` has had its own
  `errorText` and `helperText` properties since Ionic 7 (alongside `fill` and
  `labelPlacement`), but [`forms.md`](./forms.md)'s house pattern is the separate
  `@if (touched && !valid) { <ion-label color="danger"> }` block. Keep the house
  pattern. Mixing `errorText` into some fields and the label block into others
  gives you the two-presentations defect without any help from PrimeNG.
- Style the control's focus, invalid and disabled states from the same tokens
  Ionic uses (`--color-accent`, `--color-danger`, `--color-text-faint`) via
  `[data-p~="invalid"]` / `[data-p~="focus"]` / `[data-p~="disabled"]`, which
  PrimeNG emits in unstyled mode.

**Reconciliation with [`forms.md`](./forms.md):** nothing in that document
changes. This section adds one clause — *and the control is `ion-*` unless it is
an island-scoped picker, in which case its label, hint and error are still
`ion-*`*.

---

## 7. SSR and hydration

### The shapes we run

| Surface | Rendering | Libraries |
| --- | --- | --- |
| Public dynamic pages | Worker-rendered plain TypeScript, no framework | Neither. `competios/web/src/views/*.ts` — functions returning escaped HTML strings, composed by `worker.ts`. |
| Public marketing | Astro, static | Neither. `@sneat/astro` + `--color-*`. |
| The app | Angular, client-rendered (CSR) | **Ionic**, plus PrimeNG islands. |
| Angular SSR pilot | Angular + `@angular/ssr` + `provideClientHydration()` | **PrimeNG only.** `competios/frontend`. |

### The rules

- **Public dynamic pages stay out of both libraries.** They are the fastest,
  cheapest, most cacheable thing we render, they must work with JS disabled and
  under a strict CSP, and neither library earns its bytes there.
- **PrimeNG is SSR-safe enough to pilot.** competios ships `provideClientHydration()`
  with PrimeNG v21 unstyled today. PrimeNG components are ordinary Angular
  components; their markup is produced by Angular's own renderer, so Angular's
  hydration can match it.
- **Do not put Ionic behind SSR + hydration.** `@ionic/angular/standalone` is a
  set of Angular proxy directives (`ProxyCmp`) around **Stencil custom elements**
  registered with `defineCustomElement()`. The DOM inside an `ion-*` element is
  built by the custom element's own shadow-root rendering, *after* upgrade, not
  by Angular's renderer.

  This is not a theoretical concern. Ionic ships a separate
  `@ionic/angular-server` package specifically to hydrate its web components
  server-side, and **Ionic issue #30490** documents an 8.6 regression where
  `ion-header`, `ion-toolbar`, `ion-title` and `ion-content` still fail to
  hydrate — the tags are in the DOM with no `hydrated` class, an empty shadow
  root and no interactivity. It was **closed as "not planned"**, pointing
  upstream at Stencil. Until that is resolved and someone has run it against the
  Ionic version we're on and written the result down here, the rule is: **the
  Ionic app is client-rendered.**
- **`unstyled: true` removes the largest SSR styling hazard.** No runtime theme
  CSS is injected, so there is no server/client stylesheet ordering to diverge.
  The one exception is the three-rule base stylesheet, which is injected at
  runtime by the first component — provide `PrimeNgExternalBaseStyle` and put
  those three rules in the external stylesheet so they are present in the
  server-rendered document too.
- **Guard anything that reads `document`/`window`** in island code with
  `isPlatformBrowser(inject(PLATFORM_ID))`. PrimeNG guards its own; your adapter
  and your island are your problem.

---

## 8. Accessibility and focus — two libraries, one focus model

Both libraries implement focus management, competently and independently. The
standard exists so that only one of them is doing it at a time.

- **The Ionic overlay stack owns focus, always.** This follows from §3: since
  Ionic presents every overlay, Ionic's document-level focus trap is the only
  trap that is ever active for a modal flow.
- **An island participates in tab order; it does not trap.** Never enable a
  PrimeNG `focusTrap` inside an Ionic page. An island the user can tab into and
  out of is correct; an island the user cannot tab out of is a bug.
- **One visible focus ring.** Define it once, from tokens, and apply it to both:
  ```css
  :root { --sneat-focus-ring: 0 0 0 2px var(--color-bg), 0 0 0 4px var(--color-accent); }
  .sneat-prime-surface :focus-visible { outline: none; box-shadow: var(--sneat-focus-ring); }
  ```
  Do not let one stack use an outline and the other a box-shadow — on a mixed
  screen the two read as two different applications.
- **Label the island.** A `p-organizationchart` or `p-table` is an unlabelled
  region to a screen reader unless you give the wrapping element an
  `aria-label` (or `aria-labelledby` pointing at the card's `ion-card-title` /
  `ion-item-divider` label). Do this in the island's own template so it cannot be
  forgotten by the host.
- **Keep PrimeNG's ARIA; add the Sneat wording.** PrimeNG emits correct roles and
  `aria-*` for its widgets. Override only the human-readable strings, through
  `providePrimeNG({ translation: … })`, so screen-reader copy matches the app.
- **Escape must do one thing.** With islands never presenting overlays and
  `focusTrap` never enabled, only Ionic's Escape handler is live. If you find
  yourself reasoning about which handler wins, you have already broken §3.
- **`.p-hidden-accessible`** is PrimeNG's visually-hidden helper and is one of
  the three base rules. If you suppress the runtime stylesheet (§4.4) and forget
  to reproduce it, PrimeNG's screen-reader-only text becomes visible on screen.
  That is a real, shipped-looking accessibility regression; the three rules are
  not optional.

---

## 9. Bundle and performance

### What shipping both costs

- **PrimeNG is per-component standalone** — the package ships 263 separate ESM
  files and every component is `isStandalone: true`. `import { Tree } from
  'primeng/tree'` pulls that component's chunk, plus `primeng/base`,
  `primeng/basecomponent`, `primeng/config`, `primeng/api`, and the `@primeuix/*`
  utilities they import. There is a fixed base cost the first island pays and
  later islands don't. *(No production gzip figure has been measured for a Sneat
  island — the shape of the cost is established, the number is not.)*
- **PrimeNG drags in Angular CDK.** `primeng@21.1.9` declares
  `"@angular/cdk": "^21.0.0"` as a **peer dependency**. Adding the first island
  therefore adds CDK to an app that today doesn't have it. Budget for it, and
  don't then start using CDK overlays — that would be a third overlay stack (§3).
- **In unstyled mode PrimeNG's CSS cost is essentially zero** — three rules. The
  styling you write is styling you would have written anyway for a bespoke
  component.
- **Ionic's cost is already paid** by every app, and does not go up.

### The rules

- **Lazy-load the route that owns an island.** Islands are exactly the surfaces
  users visit rarely (a bracket, a chat, an admin table). An island imported into
  an eagerly-loaded route puts PrimeNG's base cost in the initial bundle for
  every user who never opens it. This is the single most important performance
  rule in this section.
- **Never barrel-import, on either side.** `import { Tree } from 'primeng/tree'`,
  never from a package root. And always
  `import { IonContent } from '@ionic/angular/standalone'` — Ionic's own build
  guidance is that importing from `@ionic/angular` "may pull in lazy loaded Ionic
  code which can interfere with treeshaking". One careless import on either side
  can pull most of a library.
- **`primeng` is a peer dependency** of every `@sneat/prime-*` library, so it is
  installed once per app, not once per island library.
- **Where the cost is unacceptable:** the public Worker-rendered pages (§7) and
  the initial app shell. Neither may import PrimeNG at all.
- **Watch the reverse direction too.** Five `sneat-libs` libraries import
  `@ionic/angular/standalone` at runtime **without declaring it as a
  dependency or peer dependency**: `@sneat/app`, `@sneat/core`, `@sneat/datagrid`,
  `@sneat/logging` (`ToastController` in `error-logger.service.ts`),
  `@sneat/wizard`. That means there is currently **no such thing as a
  PrimeNG-only Sneat surface that still uses Sneat logging** — you get Ionic
  either way. Fix the undeclared peer deps before anyone plans a PrimeNG-only
  app; until then, do not promise one.

---

## 10. Testing

Stencil custom elements and Angular components fail in different ways, so the two
need different test treatment in the same suite.

- **Ionic components need the runtime, not a schema escape hatch.** Provide
  `provideIonicAngular()` in `TestBed` — which the fleet's specs already do — so
  the proxy directives and controllers resolve. Reach for
  `CUSTOM_ELEMENTS_SCHEMA` only for a component you are deliberately not
  exercising; it silences real template errors.
- **Custom elements do not upgrade in jsdom.** An `ion-input` in a jsdom test is
  an inert unknown element: no shadow root, no internal `<input>`, no
  `ion-` lifecycle. So **unit-test Ionic components through the Angular form
  model and the component class**, not by querying rendered internals. Assert on
  `control.value` / `control.errors`, not on `input.value` inside the shadow root.
- **PrimeNG islands are the opposite and this is a real advantage.** They are
  plain Angular components with light-DOM output, so a `ComponentFixture` renders
  them properly in jsdom and you can assert on real markup. Query them by
  `[data-pc-name="tree"]` / `[data-pc-section="…"]` — these are emitted in
  unstyled mode, whereas `.p-*` classes are not, so a selector copied from
  PrimeNG's styled documentation will silently match nothing.
- **Test the adapter, not the widget.** The pure view-model adapter (§2) is where
  the logic is; it has no Angular and no DOM, and it is where the assertions
  belong. competios's `single-elimination-tree.adapter.ts` +
  `primeng-organization-chart-csp.spec.ts` are the pattern.
- **E2E: overlays are the assertion.** The failure modes in §3 are all
  runtime-only — z-index order, Escape, focus wrap, back button. Any app that
  mixes the two gets e2e coverage for: open the island → open an `ion-modal` from
  it → the modal is on top → Escape closes exactly one thing → Android back
  closes the modal, not the page.
- **Pin `mode` before you screenshot.** Visual-regression tests on an unpinned
  Ionic app compare different design languages depending on the runner's UA
  (§5).
- **Change detection differs across the boundary.** Ionic's Angular wrappers are
  deliberately kept out of Angular's normal change-detection cycle (the custom
  element owns its own rendering); a PrimeNG island is an ordinary
  `ChangeDetectionStrategy.OnPush` Angular component. If an island's inputs are
  signals — and they should be — this is a non-issue. If you find yourself
  reaching for `__Zone_disable_customElements` or manual `detectChanges()` to
  make an island update, the island is reading mutable state it should be
  receiving as a signal input.

---

## 11. Anti-patterns

| Don't | What it causes |
| --- | --- |
| Open a `p-dialog` / `p-confirmdialog` / `p-toast` in an Ionic app | Renders at z-index 1100 under Ionic's 20000 — invisible. Plus double-Escape, competing focus traps, inert scroll lock, and a hardware back button that dismisses the page behind it. (§3) |
| Set `appendTo="body"` or `providePrimeNG({ overlayAppendTo: 'body' })` | Teleports a panel out of the Ionic stacking context and out of Ionic's focus trap. The panel is unreachable by keyboard from inside an `ion-modal`. v21 defaults to `'self'` for exactly this reason — leave it. |
| Style PrimeNG with `.p-*` class selectors | Dead CSS. `cx()` returns `undefined` in unstyled mode, so those classes are never emitted. The rule looks right in the stylesheet and does nothing. competios ships a live example. (§4.4) |
| Install `@primeuix/themes` or import a PrimeNG theme CSS | Reintroduces the entire problem this standard avoids: a second complete theme system with its own `--p-*` variables, fighting Ionic's for every colour. |
| Put a `pInputText` in an `ion-item` next to an `ion-input` | Two label systems, two error renderings, two focus rings, two disabled states in one form. (§6) |
| Let a PrimeNG type (`TreeNode`, `MenuItem`, `TableLazyLoadEvent`) into a service, store or contract package | The library becomes un-swappable, and the domain model now depends on a UI vendor. Adapt at the island boundary. (§2) |
| Import an island into an eagerly-loaded route | Every user pays PrimeNG's base cost for a screen most of them never open. (§9) |
| Ship PrimeNG without reproducing the three base rules | `.p-hidden-accessible` stops hiding screen-reader-only text, so it renders on screen. (§4.4, §8) |
| Bind `[ngStyle]`-driven PrimeNG components under a strict CSP | Real, shipped bug: PrimeNG 21.1.9's `OrganizationChartNode` binds row visibility through `NgStyle` even when the chart is non-collapsible, emitting inline `style` attributes that a `style-src` CSP without `unsafe-inline` blocks. competios patches `OrganizationChartNode.prototype.getChildStyle` to a no-op (`primeng-organization-chart-csp.ts`). Check for this on every new PrimeNG component you adopt. |
| Define tokens in both a `prefers-color-scheme` block and a `.ion-palette-dark` block | Unpredictable per-property winners. Derive the class from the media query once, in TS. (§4.5) |
| Add a third UI library because neither has the control | Now three theme systems, three focus models, three overlay stacks. Build it in `@sneat/components` or `libs/prime/*`. (§1) |
| Assume "PrimeNG has a chat component" | It does not, in 21.1.9. Budget the screen as an Ionic screen with three PrimeNG parts. (§1.3) |

---

## 12. Supply chain and governance

Adopting a second UI library is a standing commitment, so record what we are
committing to.

- **`primefaces/primeng` on GitHub was archived on 2026-06-28**, as PrimeTek
  consolidates its Angular, React and Vue lines into a new "PrimeUI" foundation;
  `primeng.org` now redirects to `primeng.dev`. Existing MIT releases —
  including the 21.1.9 we are on — are stated to be unaffected. **Verified** by
  fetching both, 2026-08-19.
- **What this changes:** nothing about the code today, but the issue tracker we
  would file against, and the repository we would read source from, have moved.
  Anyone citing a PrimeNG issue URL in a Sneat commit should expect it to be an
  archived one.
- **What it means for the decision rule:** §1 already limits PrimeNG to controls
  Ionic doesn't have, in bounded islands, behind pure adapters, with no PrimeNG
  types in the domain (§2). That containment is now also the exit plan. Keep it
  tight enough that replacing `p-organizationchart` with something else is a
  one-island change, not a migration.
- **Do not upgrade PrimeNG major versions casually.** v21 changed the default
  `overlayAppendTo` to `'self'` — a change this standard depends on (§3). Read
  the migration notes against §3, §4.4 and §11 before bumping.

---

## Not yet verified

Stated here rather than asserted above:

1. **No mixed app exists.** Every overlay, focus and z-index claim in §3 and §8
   is read from `@ionic/core` 8.8.6 and `primeng` 21.1.9 shipped source, not
   observed in a running mixed app. A search for third-party reports of
   Ionic↔PrimeNG z-index, focus-trap or scroll-lock conflicts found **none** —
   this combination appears to be genuinely undocumented in the wild, so absence
   of reports is not reassurance.
2. **`color-mix()` inside Ionic's shadow DOM on Safari** (§4.3). Expected to
   work; confirm in the pilot; fall back to authored `-shade`/`-tint` hexes if not.
3. **Ionic's shade/tint formula is not published.** Ionic's docs defer to their
   colour-generator tool rather than stating the algorithm. The 12 % black / 10 %
   white mixes in §4.3 are **derived arithmetically** and reproduce Ionic's stock
   palette exactly (`#3880ff` → `#3171e0` / `#4c8dff`), which is strong evidence
   but not a vendor guarantee. If Ionic changes it, §4.3 drifts silently.
4. **PrimeNG island bundle cost** (§9). The shape of the cost is established
   (fixed base + per-component + CDK); no production gzip number has been
   measured for a Sneat island.
5. **`ControlValueAccessor` on Ionic's Angular wrappers** (§6) is relied on by
   every existing Sneat form and demonstrably works, but no Ionic-authored page
   states it explicitly. Treat it as established by practice, not by contract.
6. **Whether PrimeNG's per-component `data-p-*` state attributes are themselves
   ungated by `unstyled`** (§4.4). The `data-pc-name`/`data-pc-section` datasets
   are verified unconditional; the `data-p-*` state attributes were confirmed
   present across 112 shipped component files but not individually checked for an
   unstyled guard. If an island's `[data-p~="invalid"]` rule doesn't fire, this
   is the first thing to check.

## Where this document is in tension with the decisions it records

Recorded, not softened — the decisions above stand as written.

1. **"Chat interfaces" names a control PrimeNG does not have.** Verified by
   enumerating every module in `primeng@21.1.9`: no chat, no message thread, no
   mention component. The decision (chat is a PrimeNG case) is still sound —
   `p-scroller`, `p-autocomplete` and `p-editor` are all things Ionic lacks — but
   the deliverable is an Ionic chat screen with PrimeNG parts, not an adopted
   chat widget.
2. **"Public dynamic pages are Worker-rendered plain TypeScript" is being
   replaced inside competios by its own approved spec.**
   `competios/spec/features/angular-web-application/README.md` (Status:
   **Approved**, 2026-08-14) says: *"Replace every Competios.com presentation
   surface with one Angular and PrimeNG application using route-appropriate SSR,
   prerendering, hydration and client rendering on the existing Cloudflare
   Worker."* The Worker already dispatches `/events*` and `/tournament-formats*`
   to the Angular SSR bundle. So `competios/web/src/views/` is cited above as the
   pattern for public pages while competios itself is migrating off it. Someone
   should decide which of the two is the fleet standard.
3. **PrimeNG's own approved home is an SSR app with no Ionic in it.** The
   evidence that unstyled PrimeNG works is real; the evidence that it coexists
   peacefully with Ionic is not, because that has never been run.

---

## Summary

- **Ionic by default.** PrimeNG only where Ionic ships nothing, and only inside
  an island.
- **Islands, not interleaving.** One component, plain data in, domain events out,
  no PrimeNG types beyond its boundary, no overlays, no chrome, no routes.
- **Ionic owns every overlay.** Four independent collisions (z-index 20000 vs
  1100, double Escape, competing focus traps, inert scroll lock) plus the Android
  back button make this non-negotiable.
- **`@sneat/design-tokens` is the one vocabulary.** The Astro `--color-*` names,
  plus `success`/`warning`, plus `-rgb` companions. Ionic's six variants derive
  from it with `color-mix()`; PrimeNG has no theme to derive because it is
  unstyled, so the tokens *are* the styling.
- **Dark mode is one class** — `.ion-palette-dark` on the root — driving both.
- **Pin `mode: 'md'` in apps that host an island**; leave pure-Ionic apps adaptive.
- **Forms are single-library.** Labels, hints and errors are always Ionic markup.
- **Public pages import neither**; the app is client-rendered; only PrimeNG goes
  behind SSR.
