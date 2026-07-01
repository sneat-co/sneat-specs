# Screen Flows & the UI Component Checklist

The other docs cover **how to build one component**. This one covers **how
components connect** — the flow between forms, pages, and screens. It exists
because the most common UI defect is an *orphan*: a "Create X" page that works in
isolation but, on success, doesn't take the user anywhere — or a wizard whose
steps exist but aren't wired to each other.

> **Rule of thumb:** a screen is not "done" until both its **entry** (who links
> here) and its **exit** (where it sends the user) are wired to real screens.
> Boxes without arrows are a bug.

## Do this first: draw the flow map

Before writing any component, sketch the screens and the **arrows between them** —
predecessors → this screen → successors. For a multi-screen task (a wizard, a
create→detail pair), write the map down and confirm it before implementing:

```
contacts (list) ──"Add"──▶ new-contact ──submit──▶ contact/:id (details)
       ▲                        │
       └────────"Cancel"────────┘
```

If you can't name where an arrow lands, that's the gap to resolve — not a detail
to defer. The arrows are the deliverable that prevents orphans.

## The checklist — answer these for every UI component

Work through the screen's **lifecycle**. The ordering is deliberate: it surfaces
missing connections at both ends.

### 1. Entry — how does the user get here?

- What route/inputs are **required** (route params, IDs, query params, passed
  router `state`)? List them explicitly.
- What happens on a **cold arrival** — deep link or page refresh — when router
  `state` is gone and only the URL remains? (Don't rely on `state` being present;
  it isn't after a refresh. Re-fetch from the ID in the URL.)
- **Which existing screens link here?** Name them, and make sure those links
  actually exist. A create page with no "Add" button pointing at it is an orphan.

### 2. Validation & guards

- What makes the input valid? When do you validate (on-touch / on-submit)? How
  are errors shown? → see [`forms.md`](./forms.md).
- Is there an **access guard** — should this user/space even reach this screen?

### 3. In-progress — what happens during the action?

- Disable the submit control, show an inline spinner, and block double-submit
  while async work is in flight. → see [`buttons.md`](./buttons.md) and
  [`states.md`](./states.md).

### 4. Output — where does data go, and what comes back?

- Which service / space does the data write to?
- **What does the action return that the *next* screen needs?** Almost always the
  **new entity's `id`** (and often the entity itself, to seed the detail page).
  Capture it — you can't redirect to a detail page without it.

### 5. Exit — what happens after?

- **On success:** close / redirect / stay-and-reset? **To where, exactly?**
- **On cancel:** back to which screen?
- **On error:** stay on the screen and show feedback (toast / inline) — never
  navigate away silently, and never leave the user on a frozen-looking page.

### 6. Wire it back into the map

- Confirm every arrow you drew is implemented: entry links exist, exit
  navigations land on real screens, the wizard's steps advance and retreat.

## Navigation defaults (Sneat-specific)

Use these unless the task explicitly says otherwise. They are grounded in the
platform's navigation helpers.

### After a successful **create** → go to the entity's **details**

Navigate to the detail page of the just-created entity, using the returned `id`,
and **replace** the create page in history so Back doesn't return to a stale form:

```ts
const created = await this.contactService.createContact(space, request);
this.spaceNavService.navigateForwardToSpacePage(
  space,
  `contact/${created.id}`,
  { replaceUrl: true, state: { contact: created } },
);
```
*(contactus `new-person-form.component.ts`; assetus `add-asset-base.component.ts`
follow this exact shape.)*

- Pass the created entity in router `state` so the detail page can render
  immediately — but the detail page **must still be able to load from the `id` in
  the URL** (state is lost on refresh — see checklist #1).
- This is the **default**. Deviate only deliberately: calendarius's
  `happening-form.component.ts` returns to the **calendar** instead of the new
  happening's detail, because the calendar *is* the natural place to see it. If
  you deviate, it should be a conscious product decision, not an omission.

### Helpers — don't hand-build URLs

| Need | Use | Where |
| --- | --- | --- |
| Navigate to a space page | `spaceNavService.navigateForwardToSpacePage(space, page, navOptions)` / `navigateBackToSpacePage(...)` | `SpaceNavService` (`sneat-libs/libs/space/services`) |
| Build a space-scoped URL (for `routerLink`) | `spacePageUrl(space, page)` | `sneat-libs/libs/space/components/.../space-base-component.directive.ts` |
| Default Back target | `$defaultBackUrl()` + set `$defaultBackUrlSpacePath` in the page | `SpaceBaseComponent` |
| Go back (with browser fallback) | `SneatNavService.goBack(url)` | `sneat-libs/libs/core/.../sneat-nav.service.ts` |

Pages should extend `SpaceBaseComponent` / `SpacePageBaseComponent`, which provide
`$space`, `navigateForwardToSpacePage()`, `spacePageUrl()`, and the
`$defaultBackUrl()` signal. Set `$defaultBackUrlSpacePath` so the back button
lands on the right list (e.g. `new-member` sets `'members'`).

### Modal results

A create/edit modal returns its result by **dismissing with data**; the caller
reads it and acts (e.g. then navigates). → see [`modals.md`](./modals.md).

```ts
const modal = await this.modalController.create({ component: EditXModal, componentProps: { x } });
await modal.present();
const { data, role } = await modal.onWillDismiss();
if (role !== 'cancel' && data) { /* use data — e.g. navigate to data.id */ }
```

### Route naming

Keep create/list/detail routes consistently named relative to each other:

| Screen | Route |
| --- | --- |
| List | plural — `contacts`, `lists`, `members` |
| Create | `new-<entity>` — `new-contact`, `new-happening` |
| Detail | `<entity>/:id` — `contact/:contactID`, `happening/:happeningID` |
| Child create | `<entity>/:id/new-<child>` — `contact/:id/new-location` |

## Anti-patterns to avoid

- **Orphan page** — a screen no other screen links to, or that leads nowhere on
  success. Every screen needs an entry arrow and an exit arrow.
- **Create without redirect** — submitting a create form and leaving the user on
  the (now stale) form, or sending them somewhere with no relation to what they
  just made.
- **Stale create page in history** — redirecting after create *without*
  `replaceUrl`, so Back re-opens the filled-in form.
- **Silent failure** — an action that fails and just re-enables the button with no
  toast/inline message. → [`states.md`](./states.md).
- **Disconnected wizard** — steps that render but don't advance/retreat, or that
  lose earlier steps' data. Thread state through a parent signal; see
  contactus `person-wizard.component.ts` (recipe below).
- **Reliance on router `state`** — assuming a passed object is present. After a
  refresh only the URL survives; always be able to load from the `id`.
- **Live dead-end button** — a control wired to a handler that only logs
  `'not implemented yet'` and returns, shipped in visible markup with no user
  feedback. listus's `list-page.component.html` ships several ("Add to groceries",
  "Groceries shopping list", watch-list add/delete) whose handlers only call
  `errorLogger.logError('not implemented yet')`. Either hide the control behind a
  feature flag or don't render it.
- **Inconsistent sibling create flows** — two "create" screens for related
  entities that exit differently for no reason. contactus's `new-person-form`
  correctly redirects to the new contact's details with `replaceUrl`, but its
  sibling `new-member-form` pops/`navigateBackToSpacePage('members')` back to the
  list instead. Pick one exit convention and apply it to both.

## The wizard recipe (contactus `person-wizard`)

The one built-out multi-step wizard in the surveyed apps is contactus's
`person-wizard.component.ts`. Copy its mechanism rather than inventing another:

- **`formOrder`** — a `readonly WizardStepDef[]` naming the steps in order
  (contactType → ageGroup → gender → relatedAs → name → roles →
  communicationChannels → submit).
- **`show: {[stepID]: boolean}`** — a visibility map (one flag per step), **not**
  an index/pointer. Advancing means flipping the next step's flag on.
- **`openNext(currentStepID)`** — finds the current step, applies `skipStep()`
  filters (space-/contact-type conditionals, e.g. hide `roles` in a family space),
  reads external hide/required config from a parent-supplied `$fields()`, and — if
  the next step already `stepHasValue()` (read straight off the entity DBO) —
  auto-advances again so pre-filled/optional steps are skipped silently.
- **Data threads through a single `$contact` input / `contactChange` output** — the
  wizard holds no separate copy of the entity data; only `show` / `$wizardStep` are
  local UI state. This keeps refresh-safety trivial: the entity is the state.
