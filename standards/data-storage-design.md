# Data storage design — principles

**Status:** In force from 2026-11-13. The founder asked for this page ("We need to define
DB design principals and philosophy for sneat ecosystem") and on 2026-11-13 answered "yes"
to building it together with its three controls (below). **How to read it:** only text inside
quotation marks after **(founder)** is his. He often types on a phone; from 2026-11-14 a quote with typos is recorded as a clean version that he confirmed (the three quotes dated 2026-11-14 below: confirmed "All good", 2026-11-14). Every heading, summary and sentence outside
quotation marks is an agent's wording of what he said, and anything marked *(proposal)* is
an agent's own suggestion he has not confirmed. If the wording and his quote disagree, the
quote wins; if a proposal looks wrong for your case, ask him rather than follow it.

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
2. **Design for scale.** **(founder)** "Always design for scale in mind." For the video
   room specifically he gave the number: "No need to test -assume 100K specatators and
   design accordingly". *(proposal)* For any other layout, name its own large case (how
   many records, writers and readers at the biggest plausible use) and say what happens.
3. **Every read and write costs money.** **(founder)** "Each read/write costs money."
   *(proposal)* State document reads and writes for each user action.
4. **Embed by default.** **(founder)** "Can't some of this collections be inside of room
   record as properties?" / "WE have 1mb limit per record". *(agent's wording)* Small, bounded data lives as
   fields on its parent record; the 1 MB document limit is room to use, not a reason to
   split.
5. **A separate collection needs a stated reason.** Accepted reasons *(proposal)*:
   - many people write at once — **(founder)** "contention on join - and it's where we
     deserve a dedicated collection. Especially if we allow a lot of spectators."
   - it grows without limit (append-only history);
   - it is looked up by a different key;
   - it needs different access rules;
   - it would exceed the size budget.
6. **Never put a per-person list on a shared record that many people write at once** (for
   example everyone joining a call). One document per person instead. A small list that
   one actor edits — such as the participants of a happening — is fine on its record.
   *(proposal, follows from 2 and 5)*
7. **Nothing an idle user does may read the database repeatedly.** No per-client polling
   that scales with audience. *(proposal)*
8. **Names say what the thing is, in plain words.** **(founder)** on "epochs": "It says
   nothing to me". (the rule is a proposal)
9. **Space-bounded by default**: a record belongs to a space, so its path starts
   `spaces/{space}/ext/{owning extension}/…` (existing rule `sneat-is-space-bounded`,
   still Draft). When the data is ABOUT another extension's record, principle 12 decides
   the rest of the path: it goes under that record, not directly under your own
   extension's root in the space.

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
    contacts". *(agent's wording of that order)* (a) does the owning extension already support it?
    (b) if not, extend the owning extension; (c) only a special enough case lives outside,
    with the reason written down and approved.

    | Area | Owning extension | Source |
    |---|---|---|
    | Dates, times, calendar, scheduling, recurrence, one occurrence of a repeating event | Calendarius | founder, 2026-11-13 |
    | People and contacts | Contactus | founder, 2026-11-13 |
    | Money: wallet, charge, refund, payout, VAT | Paymentus | existing rule `money-belongs-to-paymentus` |
    | Notifications and reminders | the shared Sneat notification mechanism — the owning extension's name is not confirmed (`notificator` and `remindius` both exist): ask before choosing | founder, 2026-11-13: no product-specific notification or reminder mechanisms |
    | Meetings of every kind (in person, video, call), not only video | Meetingus | founder, 2026-11-13: "Meetingus - all kind of meetings, not just video" |
    | Spaces, membership, roles and permissions | Spaceus | founder, 2026-11-13: "Spaceus - also roles/permissions" |
    | User-authored ad-hoc forms and collecting their answers | Formius (not yet in the extension registry, so its paths are refused until it is registered) | founder, 2026-11-13: "formius is anything if we want to collect user authored ad-hoc forms" |
    | People deciding together on the place and/or time of a happening | ToGethered | founder, 2026-11-13: "togethered - when people need to decide on place and/or time of a happening" |
    | Tournaments, competitions and ranking of participants | Competios | founder, 2026-11-13: "anything for tournaments/competitions/ranking participants" |
    | Invites, including a voucher code | Invitus | founder, 2026-11-14: "A voucher code should be an Invitus invite, and the invite can be extended with extension data so it can hold monetary value and whatever is needed." The money itself stays with Paymentus (the Paymentus row); see principle 13 |

    Related existing rule: `reuse-existing-sneat-code`.

12. **Another extension's data about a record lives UNDER that record, at
    `…/ext/{extension}/…`.** *(agent's wording of the pattern; the example is his.)* The
    owner keeps its record; the other extension hangs its data beneath it, addressed the
    same way the owner addresses the thing.

    **(founder)** approved example, 2026-11-13 — the room of one occurrence of a happening.
    His words: "No, it should be under happeningAdjustment - you've lost adjustment part
    `happeningAdjsutments/{happening-id}/ext/meetingus/...`"; and to the layout below, with
    the slot level added: "yes - that what I had in min from the beginning."

    ```
    spaces/{space}/ext/calendarius/days/2026-11-13            ← Calendarius day record, a real document (unchanged)
      field  happeningAdjustments.{happeningID}.slots.{slotID}   ← the occurrence: cancel, move, attendance

    spaces/{space}/ext/calendarius/days/2026-11-13
        /happeningAdjustments/{happeningID}/slots/{slotID}    ← path segments only: NO documents are created here
            /ext/meetingus                                    ← Meetingus document: the room of that call
            /ext/meetingus/attendees/{userID}                 ← Meetingus document: one per person who joined
    ```

    How the path is formed: take the owner's document (`…/days/{date}`), repeat the owner's
    own field path to the thing as collection/document segments
    (`happeningAdjustments/{happeningID}/slots/{slotID}`), then `ext/{extension}`. Firestore
    allows documents below documents that do not exist; do not create the in-between ones —
    the occurrence itself stays a field of the day record.

    One occurrence of a happening is the session. **(founder)** "I am glad we converged on
    using happening adjustment for session and expanding it for attendence is right call."
    *(agent's reading)* So no product keeps its own session record; the attendance result
    is a Calendarius feature on the occurrence field, and the Meetingus attendee documents
    hold the detail it is derived from (who connected, when, on which devices).
    Words *(proposal)*: participants (Calendarius) = who is expected; attendees
    (Meetingus) = who joined.

13. **Records are designed to be extended: enrich the owner's record, or keep an isolated
    record when that is better.** **(founder)** 2026-11-14: "Remember that all our records are
    designed to be extendable: one extension can enrich records of another extension if it
    makes sense, or keep records isolated when that's better.".
    *(agent's reading, to be corrected)* Before creating a collection of its own, an
    extension asks whether its data is really extra data about a record another extension
    already owns. If so it enriches that record instead of keeping a parallel record with
    a copy of the owner's facts.

    **(founder)** 2026-11-14, on what "enrich" means: "I mean records can be enriched where
    that is supported by the host extension's record type, like the invite has an ext field.
    But any record can have an /ext/{id} subcollection entry, for extending it with a
    dedicated record attached under the extendable one.".
    *(agent's reading)* So there are two forms, and the layout shown for approval must say
    which: (a) extension data inside the owner's own document — only where the owner's
    record type supports it, as the Invitus invite does with its `ext` field; (b) a
    dedicated record attached under the owner's record at `…/ext/{extension}`
    (principle 12) — possible for any record. A fully isolated record is the third choice. Principle 5 (size and write frequency) decides between them, plus
    two more reasons to keep data out of the owner's document: it must be readable by a
    different audience, or it outlives the owner's record.

    A separate statement of his the same day gives an example (quoted in the owner table,
    principle 11): a voucher code is an Invitus invite, and the invite carries the monetary
    value as extension data. *(agent's note)* The money itself still follows the rule
    `money-belongs-to-paymentus`; how the value on the invite and the balance in Paymentus
    relate is a layout to show him, not something this page decides.

## What to show for approval

One page: each record kind with path, key and an example record with realistic data; what
is embedded and what is separate, with the reason from principle 5; reads and writes per
user action; which existing records were considered for reuse, and for each whether the data enriches the owner's record (inside its document, or under it at `…/ext/{extension}`) or is kept isolated, with the reason (principle 13); the typical case and the large case (principle 2); what is kept for how long.

## Controls

1. A trigger line in the shared agent-instructions index points here (owner's file).
2. Every feature spec carries a **Storage layout** section, checked by the spec linter:
   example records, existing records considered for reuse, and the owner's approval line —
   or the explicit statement that the feature stores nothing.
3. A list of every known collection per extension, next to the extension registry. A path
   that is not on the list fails CI with a link to this page. An entry records who
   approved it and when.

Controls 2 and 3 are being built (2026-11-13); until they exist this page binds as a
Stated rule only.
