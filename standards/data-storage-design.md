# Data storage design — principles (DRAFT for the founder to edit)

**Status:** Draft, 2026-11-13. Nothing here is in force until the founder approves it.
Lines marked **(founder)** quote him. Everything else is a proposal.

Read this before adding or changing any stored record, collection, index or document path.
The process rule is [`storage-layout-needs-human-approval`](https://github.com/sneat-co/backstage/tree/main/spec/rules/storage-layout-needs-human-approval).
Where a collection may live is a separate, already enforced rule:
[`firestore-approved-root-collections`](https://github.com/sneat-co/backstage/tree/main/spec/rules/firestore-approved-root-collections)
and the registry in [`sneat-go-core/firestorepolicy`](https://github.com/sneat-co/sneat-go-core/tree/main/firestorepolicy).

## Why this exists

A video-room backend was designed and built by AI agents with 19 Firestore collections,
reviewed only by other AI agents. The founder first saw the layout when he asked why there
were so many.

## Principles

1. **A human approves every layout before it is built.** **(founder)** "Going forward
   datastorage design/layout should always be presented and get explicit human approval."
2. **Design for scale.** **(founder)** "Always design for scale in mind." Working
   assumption he gave for a room: 100,000 spectators. Say what happens at 100,000 for any
   layout.
3. **Every read and write costs money.** **(founder)** "Each read/write costs money."
   State document reads and writes for each user action.
4. **Embed by default.** **(founder)** "Can't some of this collections be inside of room
   record as properties?" / "WE have 1mb limit per record". Small, bounded data lives as
   fields on its parent record. The 1 MB limit is a budget to use.
5. **A separate collection needs a stated reason.** Accepted reasons (proposal):
   - many people write at once — **(founder)** "contention on join - and it's where we
     deserve a dedicated collection. Especially if we allow a lot of spectators."
   - it grows without limit (append-only history);
   - it is looked up by a different key;
   - it needs different access rules;
   - it would exceed the size budget.
6. **Never put a per-person list on a shared record** that many people write. One document
   per person instead. (proposal, follows from 2 and 5)
7. **Nothing an idle user does may read the database repeatedly.** No per-client polling
   that scales with audience. (proposal)
8. **Names say what the thing is, in plain words.** **(founder)** on "epochs": "It says
   nothing to me". (the rule is a proposal)
9. **Space-bounded by default**: records live under `spaces/{space}/ext/{extension}/…`
   (existing rule `sneat-is-space-bounded`, still Draft).

10. **Reuse an existing record before inventing one.** **(founder)** "We already should
    have single instance happening adjustments/records that we should use to record
    instance of a recurring happening. No need to invent from scratch." / "data storage
    layout is very important and must be done right to make sure we reuse code". Search the
    ecosystem first (Calendarius, Contactus, Spaceus, Linkage…) and say what you searched.
    One occurrence of a recurring happening is named the way Calendarius already names it:
    day record `days/{YYYY-MM-DD}` → happening → slot.

11. **Separation of concerns: each area has one owning extension — check it first.**
    **(founder)** "We have a module/extension for each area. Calendarius for anything
    scheduling related - if any agent needs to do anything related to
    date/times/calendar/scheduling we should first of all check calendarius if it supports
    already or need extension or is it a special enough case to live outside. Same for
    contacts". The order is always: (a) does the owning extension already support it?
    (b) if not, extend the owning extension; (c) only a special enough case lives outside,
    with the reason written down and approved.

    | Area | Owning extension | Source |
    |---|---|---|
    | Dates, times, calendar, scheduling, recurrence, one occurrence of a repeating event | Calendarius | founder, 2026-11-13 |
    | People and contacts | Contactus | founder, 2026-11-13 |
    | Money: wallet, charge, refund, payout, VAT | Paymentus | existing rule `money-belongs-to-paymentus` |
    | Notifications and reminders | shared Sneat notifications | founder, 2026-11-13 (earlier) |
    | Meetings of every kind (in person, video, call), not only video | Meetingus | founder, 2026-11-13: "Meetingus - all kind of meetings, not just video" |
    | Spaces, membership, roles and permissions | Spaceus | founder, 2026-11-13: "Spaceus - also roles/permissions" |
    | User-authored ad-hoc forms and collecting their answers | Formius | founder, 2026-11-13: "formius is anything if we want to collect user authored ad-hoc forms" |
    | People deciding together on the place and/or time of a happening | ToGethered | founder, 2026-11-13: "togethered - when people need to decide on place and/or time of a happening" |

    Related existing rule: `reuse-existing-sneat-code`.

## What to show for approval

One page: each record kind with path, key and an example record with realistic data; what
is embedded and what is separate, with the reason from principle 5; reads and writes per
user action; which existing records were considered for reuse; the typical case and the 100,000 case; what is kept for how long.

## Proposed controls (not built)

- A list of every known collection per extension, in one file, next to the existing
  extension registry. A new path that is not on the list fails CI with a message linking
  this page.
- The entry for a collection records who approved it and when.
