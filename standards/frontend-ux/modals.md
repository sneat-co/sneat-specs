# Modals, Dialogs & Feedback

Modals are native Ionic, created via `ModalController`, and structured as
header → content → footer. Use a modal for focused edit/create flows, an alert
for confirmation, a popover for context menus, and a toast for transient feedback.

## Prefer `SneatBaseModalComponent`

Extend the shared base
(`sneat-libs/libs/ui/.../components/sneat-base-modal.component.ts`) for modal
components. It provides standard `close()` / `dismissModal(data?, role?)` helpers
and error logging, so dismissal is consistent.

```ts
export class HappeningTitleModalComponent extends SneatBaseModalComponent { … }
```
*(calendarius `happening-title-modal.component.ts`,
`happening-price-modal.component.ts`.)*

**Recommended:** use the base class. A few modals (listus dialogs) roll their own
dismissal without it — acceptable, but new modals should follow the base-class
pattern.

## Structure

```html
<ion-header>
  <ion-toolbar color="light">
    <ion-title>{{ title }}</ion-title>
    <ion-buttons slot="end">
      <ion-button (click)="close()"><ion-icon name="close-outline" /></ion-button>
    </ion-buttons>
  </ion-toolbar>
</ion-header>

<ion-content>
  <!-- form fields -->
</ion-content>

<ion-footer>
  <ion-toolbar>
    <ion-buttons slot="end">
      <ion-button (click)="close()"><ion-label>Cancel</ion-label></ion-button>
      <ion-button color="primary" [disabled]="$isSubmitting() || !valid()"
        (click)="save()">
        <ion-icon name="save-outline" slot="start" />
        <ion-label>Save</ion-label>
      </ion-button>
    </ion-buttons>
  </ion-toolbar>
</ion-footer>
```
*(calendarius `happening-title-modal.component.html`; contactus
`contact-names-modal.component.html`.)*

- **Header:** `ion-toolbar color="light"` + `ion-title`; a close button
  (`close-outline`, never filled) in `slot="end"`. Use `color="primary"` only for
  weightier flows (e.g. invites, payments). **Recommended:** always include the
  header close button — listus's "footer-cancel-only" variant is the exception,
  not the rule.
- **Footer:** Cancel + primary action in `<ion-buttons slot="end">`. Primary is
  `color="primary"` with an icon + label; disable it while submitting (see
  [`buttons.md`](./buttons.md)).
- **Titles** can carry context, e.g. `Invite {{ member | contactTitle }}`.

## Invocation & results

```ts
const modal = await this.modalController.create({
  component: HappeningTitleModalComponent,
  componentProps: { happening },
});
await modal.present();
const { data, role } = await modal.onWillDismiss();
```

Return results by dismissing with data and a `role` (e.g. `'cancel'`). With
`SneatBaseModalComponent`, call `dismissModal(data, role)`.

## Page-shaped modals (routed-looking, but presented as modals)

listus's dialogs are a distinct, sanctioned shape worth naming: components under
`pages/.../dialogs/` named `*-page.component.ts` and built with a **full
`ion-header` / `ion-content` / `ion-footer` page skeleton**, yet **never
registered as routes** — they are presented via `ModalController.create(...)`.
*(listus `pages/dialogs/copy-list-items/copy-list-items-page.component.ts`,
`pages/lists/new-list-dialog.component.ts`.)*

- This is an accepted alternative to a compact `SneatBaseModalComponent` when the
  dialog is content-heavy enough to want a page layout — but the naming is
  misleading (it reads like a route, behaves like a modal), so **keep the
  `*-page` under a `dialogs/` folder** to signal intent.
- These components predate `SneatBaseModalComponent` and roll their own
  `modal?.dismiss(...)`. New page-shaped modals should still extend the base class.
- If you hit "modal.dismiss() can't find the modal", listus's workaround is to
  patch `modal.componentProps['modal'] = modal` right after `create()`
  *(listus `ListDialogs.service.ts`)*.

## Inline modals & popovers

- **Inline `ion-modal` with `trigger`** for quick pickers (date/time): a trigger
  button + `<ion-modal trigger="…"><ng-template>…</ng-template></ion-modal>`.
  *(calendarius `start-end-datetime-form.component.html`.)*
- **`PopoverController`** for context menus — no header, just an `ion-item-group`
  of actions. *(calendarius `slot-context-menu.component.html`.)*

## Destructive confirmation

The apps confirm destructive actions with the native `confirm()` dialog rather
than `AlertController`:

```ts
if (!confirm('Delete this price?')) return;
```
*(contactus `contact-page.component.ts`, `comm-channel-item.component.ts`;
calendarius `happening-prices.component.ts`.)*

This is the current house convention. **Recommended:** keep using `confirm()` for
simple yes/no destructive prompts for consistency; reach for `AlertController`
only when you need styled buttons or inputs.

**Irreversible bulk actions must confirm — swipe is not consent.** A "Clear list"
that deletes every item, or a "Remove" that deletes a whole list, needs a
`confirm()` step. listus's `list-page.component.ts` "Clear list" and
`lists-page.component.ts` list "Remove" currently execute immediately with **no**
confirmation — relying on the swipe gesture as the only friction. Don't ship an
irreversible action whose only guard is the gesture that triggered it.

## Toasts for feedback

Use `ToastController` for non-blocking success/error feedback after an action:

```ts
const toast = await this.toastController.create({
  message: 'Invitation sent',
  duration: 2000,
  buttons: [{ role: 'cancel', text: 'OK' }],
});
await toast.present();
```
*(contactus `invite-modal.component.ts`; listus `new-list-item.component.ts`.)*

- **Duration by weight:** `2000` for full verb-phrase confirmations ("Invitation
  sent", "Contact created"); a shorter `~1500` for lightweight
  copy-to-clipboard acknowledgements. contactus `invite-modal.component.ts`
  demonstrates both (2000 for `sendInvite`, a `1500` default for the copy-link
  toasts) — match the duration to the weight of the message rather than using one
  flat number.
- `color="danger"` for errors (see [`states.md`](./states.md)),
  `position: 'middle'` for important messages.
- **Every mutating action deserves a toast on failure** — not just a silent
  `errorLogger.logError(...)`. See the "Surface failures" rule in
  [`states.md`](./states.md).

## Summary

| Need | Use |
| --- | --- |
| Focused create/edit | `ModalController` + `SneatBaseModalComponent`, header/content/footer |
| Quick picker | inline `ion-modal` with `trigger` |
| Context menu | `PopoverController` |
| Yes/no destructive confirm | native `confirm()` |
| Transient success/error feedback | `ToastController` |
