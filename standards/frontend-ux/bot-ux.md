# Bot UX (Telegram)

The other docs cover the Ionic web frontends. This one covers **conversational UX
for the Telegram bots** (debtus / splitus), which are a different medium — no
cards, no spinners, no routing, just messages and inline keyboards. The
conventions below are codified from the debtus/splitus bot handlers under
`debtus/backend/bots/`. (The bot is mid-migration to Cloud Run; these are UX
conventions, independent of deployment.)

> Like the web docs, these are conventions, not a framework. Consistency comes
> from following the message/keyboard patterns, not from a shared widget.

## Message construction

- **Go through the `botmsg` abstraction.** Live handlers build a
  `botmsg.MessageFromBot{Text, Format, Keyboard, IsEdit}` and let the framework
  decide send-vs-edit. Reach for raw `tgbotapi.*Config` only in background/queue
  jobs that have no webhook context *(debtus
  `dtb_general/debtus_home_command.go`; raw path only in
  `delayed4debtus/reminder_delays.go`)*.
- **Format is HTML.** Use `botmsg.FormatHTML` (~90 call sites); avoid Markdown /
  MarkdownV2 entirely. Reserve `FormatText` for the rare plain-text reply.

## Navigation

- **Edit-in-place on a callback, new message on a typed command.** This is the
  dominant rule — it keeps menu navigation from flooding the scrollback:

  ```go
  if whc.Input().InputType() == botinput.TypeCallbackQuery {
      m, err = whc.NewEditMessage(buffer.String(), botmsg.FormatHTML)
  } else {
      m = whc.NewMessage(buffer.String())
  }
  ```
  *(debtus `dtb_transfer/transfer_balance_cmd.go`.)*

- **Callback data is a query string**, parsed with `net/url` — never
  colon/positional: `command + "?bill=" + billID`, `"%s?id=%s&in=%s"`
  *(splitus `bill_card_2_cmd.go`; debtus `callback_reminder_return.go`)*.

- **"Back" points at the parent screen's own command**, prefixed with the 🔙
  glyph — there is no generic stack-based back. Use the shared
  `emoji.RETURN_BACK_ICON` constant rather than a literal emoji so the two bots
  stay uniform *(splitus `bill_card_2_cmd.go`; debtus `transfer_balance_cmd.go`)*.

## Destructive actions — pick one shape

- **Cheaply reversible → act now, offer Undo.** Delete immediately, then edit the
  message to show a single "Restore" button *(splitus `bill_delete_cmd.go`)*.
- **Not reversible → Yes/Cancel confirm screen**, emoji-prefixed labels, Cancel
  routing back to the originating card *(splitus `bill_finalize_cmd.go`)*. The
  "Yes" button must route to a **distinct confirmed-action command**, not back
  into the confirm screen itself (the current finalize flow loops on itself — a
  bug, not a pattern to copy).

## Pending / long-running writes

There is no spinner and no typing indicator. The sanctioned affordance is a
**"⏳ …ing" placeholder message, later edited into the result** by targeting that
same message ID *(debtus `dtb_transfer/transfer_common.go` — placeholder then
edit-into-receipt)*. Use `ShowAlert=true` on a callback answer to block a
duplicate action *(splitus `bill_join_cmd.go`)*.

## Multi-step flows

Reuse the per-chat step machinery the framework already provides — don't invent a
new state machine:

- `PushStepToAwaitingReplyTo(...)` / `IsAwaitingReplyTo(...)` /
  `SetAwaitingReplyTo("")` to advance and clear steps.
- `AddWizardParam(...)` / `GetWizardParam(...)` to carry data between steps.

*(debtus `dtb_transfer/transfer_comment_cmd.go`; the 3-step settle flow in
`start_settle_group_cmd.go`.)*

## Errors

Domain errors bubble to a shared framework fallback that sends a **brand-new 🚨
message** (never an edit) *(`bots-fw/botswebhook/driver.go`)*. Translate a domain
error into inline text only when it's actionable for the user *(splitus
`bill_delete_cmd.go` uses the error string as an i18n key)*.

## Keyboard layout

- **One item per row** for menus/action lists *(splitus `split_mode_list_cmd.go`)*.
- **Multi-column grid** for dense pickers — e.g. a 4-column currency picker
  *(`currencies.go`)*.
- For a "choose current value" picker, **omit the current selection** from the
  list rather than marking it *(`anybot/inlinekeyboards/choose_lang_kb.go`)*.

## Summary

| Need | Convention |
| --- | --- |
| Build a message | `botmsg.MessageFromBot`, `FormatHTML` |
| Menu navigation | Edit-in-place on callback, new message on command |
| Encode an action | `<command>?key=value&key2=value2`, parsed via `net/url` |
| Back | 🔙 + parent screen's own command (`RETURN_BACK_ICON`) |
| Reversible delete | Act now + single Undo/Restore |
| Irreversible action | Yes/Cancel confirm → distinct confirmed command |
| Long write | "⏳ …ing" placeholder, then edit that message into the result |
| Multi-step | `AwaitingReplyTo` step-stack + `WizardParam` bag |
| Error | Shared 🚨 fallback (new message); inline text only if actionable |
