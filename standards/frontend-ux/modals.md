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

- `duration: 2000` default; `color="danger"` for errors (see
  [`states.md`](./states.md)), `position: 'middle'` for important messages.

## Summary

| Need | Use |
| --- | --- |
| Focused create/edit | `ModalController` + `SneatBaseModalComponent`, header/content/footer |
| Quick picker | inline `ion-modal` with `trigger` |
| Context menu | `PopoverController` |
| Yes/no destructive confirm | native `confirm()` |
| Transient success/error feedback | `ToastController` |
