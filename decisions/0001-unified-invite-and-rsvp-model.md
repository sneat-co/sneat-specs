# ADR-0001: Unified invite & RSVP model

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-06-29 |
| **Deciders** | Alex Trakhimenok |
| **Affected repos** | `sneat-core-modules` (invitus), `eventus`, `gameboard`, `rsvp-express`, `debtus` / `sneat-mod-debtus-go`, `contactus`, `sneat-libs` |

## Context

Sneat needs invites and RSVPs for several verticals — joining a space, RSVPing
to an event (birthday, eventus), confirming attendance at a GameBoard game, and
the legacy debtus "join to track a debt" flow. The trigger for this decision was
designing the **GameBoard ↔ RSVP.express** integration, which surfaced how much
invite/RSVP machinery already exists — and how much of it is **duplicated**.

Today there are at least five overlapping implementations:

| Implementation | What it models | Disposition |
|---|---|---|
| **invitus** — `sneat-core-modules/invitus/dbo4invitus` | Generalized invite: `SpaceID`/`InviteSpace` (team→space rename done), an emerging **`TargetType` + `TargetIDs`** (`""`, `tracker`, `user`), `Roles[]`, coarse `AcceptedByUserIDs`/`DeclinedByUserIDs` + counts, `create_invite_response` facade | **Canonical** — becomes the one invite |
| invitus — `sneat-go-modules/modules/invitus/models4invitus` | Older team-flavored copy (`TeamID`/`InviteTeam`) | **Delete** |
| **eventus** — `contract/.../models/{rsvp,invitation,rsvp-link}.ts` | Richest event-RSVP: `yes`/`no`/`maybe` + adult/child headcounts + dietary/comment, open-vs-invitee `LinkKind`, privacy-safe public `IRsvpContext`; explicitly *"RSVP is pure event-attendance and NEVER joins the host's space"* | **Promote** — its response model becomes shared |
| debtus / `sneat-mod-debtus-go` invite | Telegram-era invite (channels tg/fbm, claims, `Related`) | **Fold in** as `target.type = "tracker"` |
| contactus — `contact_invites.go` | Contact-level invites | **Reconcile** |

Two important properties of the problem:

1. **Membership ≠ attendance.** invitus's center of gravity is "accept = join the
   space as a member." An event RSVP is a different lifecycle: a guest can say
   "going" and show up as social proof **without** joining anything. eventus
   already enforces this boundary; invitus does not.
2. **The invite landing is a conversion surface for logged-out strangers.** A
   recipient taps a link from SMS/WhatsApp with no account, no app, and no
   extension installed. Whatever renders the invite must work from plain data —
   it cannot rely on loading an extension's Angular components. (Confirmed: the
   Sneat frontend has **no** dynamic-component/widget registry today.)

This is a **greenfield** decision: no real users, breaking changes and a full
data wipe are acceptable, so we design the ideal target model directly rather
than a backward-compatible migration.

## Decision

### 1. One canonical invite, in invitus

`sneat-core-modules/invitus` is the single invite record. It owns invite
**transport + identity** (channel, link, pin, expiry, from/to, message, roles).
Generalize it with two fields:

- **`Target`** — *what* the invite is for. Generalizes the existing
  `TargetType`/`TargetIDs`; a space becomes one target type among several, not a
  special case.
- **`Domain`** — *who handles* the invite: the app/server that renders its
  landing and accepts its responses (`sneat.team`, `rsvp.express`,
  `gameboard.live`, or a white-label team's own Sneat server). Drives branding
  and accept-routing. New — nothing models this today.

### 2. Two records: `Invite` (1) → `Response` (N)

The RSVP answer lives in a **separate `Response` record** that references the
invite — not folded onto the invite. An open/shareable invite yields many
responses; responses are editable; open-link responders self-identify with no
invitation row. Headcounts/dietary/comments are attendance data that does not
belong on the invite.

The `Response` shape is **promoted from eventus** (`yes`/`no`/`maybe` +
headcounts + dietary/comment + attribution + public resolve context) and
**replaces** invitus's coarse `AcceptedByUserIDs`/`DeclinedByUserIDs`. With a
data wipe permitted, we keep only one response mechanism.

**Where it lives — in invitus, co-located with `Invite`.** `Response` is the
*universal* answer to any invite: accepting a space invite, RSVPing to an event,
and confirming a game are all a `Response` (status `yes`/`maybe`/`no` +
attribution), with event-only extras (headcounts, dietary, comment) as optional
fields. Because it is universal — not event-specific — it belongs with `Invite`
in invitus, not in a separate `rsvpus` module (illusory separation) and not in
eventus (which would invert the layering, making a vertical a platform
dependency). Co-location keeps the denormalized social-proof counters on
`Invite` **transactional**: a `Response` write and the counter bump happen in one
transaction in one store. `Response` is stored as a subcollection of its invite
(`/invites/{inviteID}/responses/{responseID}`); the landing reads only the
denormalized summary on `Invite`, never a query over responses — prod's
`dalgo2memcachegae` store does not support queries (the same constraint that
shapes the GameBoard single-doc event log).

### 3. Membership join is an optional follow-on, never the RSVP

Submitting a `Response` never joins a space. A `yes` from someone who *should*
become a member (e.g. a player joining the roster) may **trigger** invitus's
existing space-join flow as a separate, explicit step. Spectators, friends, and
party guests respond without joining anything.

### 4. Social proof is denormalized and host-controlled

Aggregate counters (`going`/`maybe`/`declined`, headcount totals) are
denormalized onto the invite/target so the public landing renders "12 going"
without reading the guest list. Exposing *who* is coming is gated by a
host-controlled visibility setting: `counts-only` | `first-names` | `avatars` |
`hidden`. The public resolve context (eventus `IRsvpContext`) never leaks the
full guest list.

### 5. Rendering by data-delegation, not component-delegation

An extension projects `(Target, recipient Role) → IInvitePresentation` — a
generic view-model (title, hero, host, role-specific call-to-action wording,
generic fact sections, deep-link). The host app (RSVP.express / eventus) renders
that view-model with its own generic components. The host app stays ignorant of
extension domain logic; the extension never ships components into the host. This
is the only approach that works for a logged-out stranger on a public landing.

### 6. Cross-entity links use the generic `related` graph

Domain entities link to each other (e.g. a GameBoard game ↔ its happening) via
the platform's existing `related` graph (`IWithRelatedOnly` / `IRelatedModules`
in `sneat-libs`), so neither side hard-codes knowledge of the other. A game
reads its schedule/venue through the linked happening rather than denormalizing
it.

## Model sketch

```go
// invitus — the one invite
type Invite struct {
    From    InviteContact
    Channel string            // email | sms | link
    Pin     string
    Expires time.Time
    Status  InviteStatus
    Roles   []string          // player|coach|parent|judge|friend|member ...
    Message string

    Target  Target            // WHAT (generalizes TargetType/TargetIDs)
    Domain  string            // WHO handles it (app/server/white-label host)

    Kind    InviteKind        // "personal" (bound to one To) | "open" (shareable, Limit)
    To      *InviteContact    // set for personal; nil for open
    Limit   int               // open invites only

    // denormalized social proof
    Going, Maybe, Declined, Adults, Children int
    Visibility string         // counts-only | first-names | avatars | hidden
}

type Target struct {
    Type string   // "space" | "happening" | "game" | "tracker" | "user"
    IDs  []string // e.g. [happeningID]  or  [spaceID, gameID]
}

// shared RSVP response (promoted from eventus; replaces accept/decline)
type Response struct {
    InviteID string
    Status   string  // "yes" | "maybe" | "no"
    Adults, Children int
    Dietary, Comment string
    SelfIdentifiedName, SelfIdentifiedFamilyName string // open-link attribution
    ViaAccount  bool
    SubmittedAt time.Time
}
```

### Target types

| `Target.Type` | `IDs` | Accept semantics |
|---|---|---|
| `space` | `[spaceID]` | join space as member (existing invitus flow) |
| `happening` | `[happeningID]` | RSVP to event; no join |
| `game` | `[spaceID?, gameID]` | RSVP to game (schedule read-through from happening); role-aware |
| `tracker` | `[trackerID]` | legacy debtus join-to-track |
| `user` | `[userID]` | direct user-to-user invite |

### Domain

`Domain` identifies the host that serves the invite's landing and accepts its
responses. It drives (a) which app/branding renders the landing, and (b) which
backend the response is posted to — including a white-label team running its own
Sneat server. Set at invite creation from where the host operates.

## Consequences

**Positive**

- One invite model instead of five; one RSVP model instead of two.
- Membership-vs-attendance boundary is explicit and enforced by structure.
- Public landing works for logged-out strangers; no extension code loaded.
- GameBoard ↔ RSVP collapses to: a `target.type="game"` invite with a role-aware
  view-model, answered via the shared `Response`.
- `Target` + `Domain` make the invite reusable across every vertical and
  white-label deployments.

**Negative / costs**

- Real teardown work: delete the old go-modules invitus, fold debtus invites in,
  reconcile contactus, promote eventus's response model to the shared layer.
- Two records (Invite + Response) to keep consistent; denormalized counters must
  be maintained transactionally on response writes.

**Neutral**

- Each extension must implement the `(Target, Role) → IInvitePresentation`
  projection to get rich invite landings; until it does, a generic fallback card
  renders from the target's denormalized summary.

## Alternatives considered

- **Extend the happening record to hold game/RSVP data.** Rejected: couples
  GameBoard's high-frequency live event log to the happening, and makes
  standalone (no-RSVP) games awkward.
- **One record (invite carries the response).** Rejected: wrong cardinality —
  open links have many responses; responses are editable; attendance fields
  pollute the invite.
- **Component-delegation registry** (extensions contribute Angular components a
  host renders via `NgComponentOutlet`). Deferred, not chosen for the invite
  landing: it cannot run for a logged-out stranger with nothing installed. It
  remains the right tool for the *rich in-app* experience after conversion.
- **A new RSVP model in rsvp-express.** Rejected: eventus already has the richest
  one; duplicating it is the very problem this ADR resolves.

## Open questions / follow-ups

- Whether `Domain` is a bare host string or a small struct (`{ app, host }`).
- First implementation slice and teardown order across the affected repos.
