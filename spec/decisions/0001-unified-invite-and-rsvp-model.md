---
format: https://specscore.md/decision-specification
status: In Review
---
# Decision: Unified invite & RSVP model

**Status:** In Review
**Date:** 2026-06-29
**Owner:** alex
**Tags:** —
**Source Idea:** —
**Supersedes:** —
**Superseded By:** —

## Context

Sneat needs invites and RSVPs for several verticals — joining a space, RSVPing to
an event (birthday, eventus), confirming attendance at a GameBoard game, and the
legacy debtus "join to track a debt" flow. The trigger was designing the
**GameBoard ↔ RSVP.express** integration, which surfaced how much invite/RSVP
machinery already exists — and how much is **duplicated**. At least five
overlapping implementations exist:

| Implementation | What it models | Disposition |
|---|---|---|
| **invitus** — `sneat-core-modules/invitus/dbo4invitus` | Generalized invite: `SpaceID`/`InviteSpace` (team→space rename done), an emerging **`TargetType` + `TargetIDs`** (`""`, `tracker`, `user`), `Roles[]`, coarse `AcceptedByUserIDs`/`DeclinedByUserIDs` + counts, `create_invite_response` facade | **Canonical** — becomes the one invite |
| invitus — `sneat-go-modules/modules/invitus/models4invitus` | Older team-flavored copy (`TeamID`/`InviteTeam`) | **Delete** |
| **eventus** — `contract/.../models/{rsvp,invitation,rsvp-link}.ts` | Richest event-RSVP: `yes`/`no`/`maybe` + adult/child headcounts + dietary/comment, open-vs-invitee `LinkKind`, privacy-safe public `IRsvpContext`; explicitly *"RSVP is pure event-attendance and NEVER joins the host's space"* | **Promote** — its response model becomes shared |
| debtus / `sneat-mod-debtus-go` invite | Telegram-era invite (channels tg/fbm, claims, `Related`) | **Fold in** as `target.type = "tracker"` |
| contactus — `contact_invites.go` | Contact-level invites | **Reconcile** |

Two properties shape the choice. **Membership ≠ attendance:** invitus's center of
gravity is "accept = join the space as a member," but an event RSVP is a different
lifecycle — a guest can say "going" and appear as social proof **without** joining
anything (eventus already enforces this; invitus does not). **The invite landing
is a conversion surface for logged-out strangers:** a recipient taps an SMS link
with no account, app, or extension installed, so the landing must render from
plain data, not by loading an extension's Angular components (the Sneat frontend
has no dynamic-component registry). This is a **greenfield** decision — no real
users; breaking changes and a data wipe are acceptable — so we design the ideal
target directly.

## Decision

### 1. One canonical invite, in invitus

`sneat-core-modules/invitus` is the single invite record, owning invite transport
+ identity (channel, link, pin, expiry, from/to, message, roles). Generalize it
with **`Target`** (*what* the invite is for — generalizes `TargetType`/`TargetIDs`;
a space becomes one target type, not a special case) and an optional **`Routing`**
(*where the invitee goes* — see [§7](#7-routing-is-optional-override-only-and-resolved-at-open-time)).

### 2. Two records: `Invite` (1) → `InviteResponse` (N)

The answer is a **separate `InviteResponse` record** referencing the invite — not
folded onto it (an open invite yields many responses; responses are editable;
open-link responders self-identify with no invitation row). It is **promoted from
eventus** (`yes`/`no`/`maybe` + headcounts + dietary/comment + attribution +
public resolve context) and **replaces** invitus's coarse
`AcceptedByUserIDs`/`DeclinedByUserIDs`. It is the *universal* answer to any invite
(space-join, event RSVP, game confirm are all an `InviteResponse`), with event
extras optional — so it belongs in **invitus**, not a separate `rsvpus` module and
not eventus (which would invert layering).

> **Naming.** "RSVP" (*répondez s'il vous plaît*) is grammatically the *request* —
> it belongs to the invite (`rsvpBy` deadline). The invitee's structured, recorded
> answer is an **`InviteResponse`** (`InviteResponseDbo`). We avoid bare `Response`
> (un-greppable) and `Reply` (implies free-text conversation). `InviteResponse`
> aligns with invitus's existing `create_invite_response` facade; eventus's
> `IRsvp` is renamed on promotion.

**Storage — in invitus's reserved system space `$invitus`.** Invites and responses
are platform-owned, cross-user, spaceless records, so they live in invitus's
**reserved system space** (`$invitus`, type `SpaceTypeSystem`) under the standard
space-module path — see [decision 0002](0002-reserved-extension-space-ids.md) and
the [`reserved-extension-space-ids`](../features/reserved-extension-space-ids/README.md)
and [`system-space-type`](../features/system-space-type/README.md) Features:

```
/spaces/$invitus/ext/invitus/invites/{inviteID}
/spaces/$invitus/ext/invitus/inviteResponses/{responseID}
```

`InviteResponse` carries denormalized ref fields (`inviteID`, `target {type,
ids}`, `responderUserID`, `status`). Because the reserved space + module prefix is
a **fixed** path, cross-cutting host views ("responses across my happenings" →
`WHERE target.happeningID == …`; "all my responses" → `WHERE responderUserID == …`)
are plain indexed collection queries — not collection-group, not space-variable. A
per-invite subcollection is rejected: a response relates to invite **+ happening +
responder**, not one parent. Social-proof counters live on `Invite`; the counter
bump and the response write are **one transaction** (Firestore transactions span
collections, so co-location is not required). The system-space auth model fits:
any authenticated user may write (no membership), reads are public, and invitus
enforces per-record authorization.

### 3. Membership join is an optional follow-on, never the RSVP

Submitting an `InviteResponse` never joins a space. A `yes` from someone who
*should* become a member (e.g. a player joining the roster) may **trigger**
invitus's existing space-join flow as a separate step. Spectators, friends, and
party guests respond without joining anything.

### 4. Social proof is denormalized and host-controlled

Aggregate counters (`going`/`maybe`/`declined`, headcount totals) are denormalized
onto the invite so the public landing renders "12 going" without reading the guest
list. Showing *who* is coming is gated by a host visibility setting: `counts-only`
| `first-names` | `avatars` | `hidden`. The public resolve context never leaks the
full list.

### 5. Rendering by data-delegation, not component-delegation

An extension projects `(Target, recipient Role) → IInvitePresentation` — a generic
view-model (title, hero, host, role-specific CTA, generic fact sections,
deep-link). The host app renders it with its own generic components, staying
ignorant of extension domain logic. This is the only approach that works for a
logged-out stranger on a public landing.

### 6. Cross-entity links use the generic `related` graph

Domain entities link via the platform's `related` graph (`IWithRelatedOnly` /
`IRelatedModules` in `sneat-libs`), so neither side hard-codes the other; a game
reads schedule/venue through its linked happening. The linkage validator
(`WithRelated.ValidateWithKey`) derives a `spaceID` from key ancestry, so records
carrying `related` need a space ancestor — GameBoard games satisfy this by living
in the reserved system space `$gameboard` per
[decision 0002](0002-reserved-extension-space-ids.md).

### 7. Routing is optional, override-only, and resolved at open-time

Where the invitee goes is **not** a single frozen `domain` string — that conflates
three orthogonal axes (**product** · **channel** · **host**) across two distinct
moments (**review** the invite vs **proceed** after the response), and it would
freeze per-visitor host selection that must stay dynamic. Instead the invite
carries an optional **`Routing`** of three symmetric `Destination`s — `Review`,
`OnAccept`, `OnDecline` — each with optional `Product` / `Channel` / `Host`.
Everything is **derived by default** (product from `target.type`: game →
gameboard.live, space → sneat.team, happening → eventus/rsvp; channel → `web`;
host → resolved from product + channel + visitor locale/white-label). Only
deliberate **overrides** are stored: drive into a bot (`Channel: "bot"`), pin a
white-label or a specific host (`Host`), or send the invitee to a different
product after acceptance than the one that hosted the review (e.g. review in
rsvp.express, `OnAccept.Product = "gameboard"`).

The concrete URL is computed **at open-time**, not stored — so the **sent link is a
stable platform resolver/shortlink** (`https://snt.link/i/{inviteID}`) that applies
the matching `Destination` plus the *current* visitor's locale/tenant and
redirects. Pinning a per-visitor host (locale/geo → `.ie`) onto the invite is
explicitly disallowed; that is the resolver's job, with the `Host` override as the
escape hatch when a specific host is genuinely intended. The open-time resolver
itself is a separate future Feature (it is where locale and white-label routing
live); its `product × locale → host` mapping is a **static platform config**
(reviewed/deployed, read-free on the hot path), with the `Host` override covering
white-label and force-a-host cases — not a convention-derived host or a
runtime-read registry doc.

### Model sketch

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
    Routing *Routing          // WHERE the invitee goes — optional; omit → all derived

    Kind    InviteKind        // "personal" (bound to one To) | "open" (shareable)
    To      *InviteContact    // set for personal; nil for open
    Limit   int               // open invites only
    RsvpBy  *time.Time        // the RSVP *request*: respond-by deadline (optional)

    // denormalized social proof
    Going, Maybe, Declined, Adults, Children int
    Visibility string         // counts-only | first-names | avatars | hidden
}

type Target struct {
    Type string   // "space" | "happening" | "game" | "tracker" | "user"
    IDs  []string // e.g. [happeningID]  or  [spaceID, gameID]
}

// optional per-stage routing overrides; resolved to a concrete URL at open-time.
// Nested structs are pointers so omitempty actually omits them; scalars are plain.
type Routing struct {
    Review    *Destination `firestore:"review,omitempty"`    // entry: review the invite
    OnAccept  *Destination `firestore:"onAccept,omitempty"`  // outcome: after "yes"
    OnDecline *Destination `firestore:"onDecline,omitempty"` // outcome: after "no" (rare)
}

type Destination struct {
    Product string `firestore:"product,omitempty"` // "gameboard"|"sneat-team"|… default: from target.Type
    Channel string `firestore:"channel,omitempty"` // "web"|"bot"|"app"            default: "web"
    Host    string `firestore:"host,omitempty"`    // pin a host (white-label/.ie/bot) default: resolved
}

// the universal invite-answer (promoted from eventus IRsvp; replaces accept/decline)
type InviteResponseDbo struct {
    InviteID        string
    Target          Target   // denormalized for cross-invite/cross-happening queries
    ResponderUserID string
    Status          string   // "yes" | "maybe" | "no"
    Adults, Children int
    Dietary, Comment string
    SelfIdentifiedName, SelfIdentifiedFamilyName string // open-link attribution
    ViaAccount  bool
    SubmittedAt time.Time
}
```

### Target types & routing

| `Target.Type` | `IDs` | Accept semantics | Default product |
|---|---|---|---|
| `space` | `[spaceID]` | join space as member (existing invitus flow) | sneat-team |
| `happening` | `[happeningID]` | RSVP to event; no join | eventus / rsvp |
| `game` | `[spaceID?, gameID]` | RSVP to game (schedule read-through); role-aware | gameboard |
| `tracker` | `[trackerID]` | legacy debtus join-to-track | debtus |
| `user` | `[userID]` | direct user-to-user invite | rsvp |

`Routing` (optional) overrides the derived destinations per stage; the default
product per target is shown above. See
[§7](#7-routing-is-optional-override-only-and-resolved-at-open-time).

## Rationale

The duplication is the problem to solve: invitus and eventus are converging on the
same thing from opposite directions (invitus generalized *who/what* you invite but
has only coarse accept/decline; eventus built the rich *attendance response* but
bound it to its own event model). Unifying on invitus-as-transport +
eventus-response-as-shared collapses five implementations to one, and makes
GameBoard ↔ RSVP a thin projection (`target.type="game"` + a role-aware
view-model). The membership-vs-attendance split — already shipped in eventus —
becomes structural rather than a per-extension convention. Data-delegation
rendering is forced by the logged-out-stranger constraint; a component registry
cannot run where nothing is installed.

## Declined Alternatives

### Extend the happening record to hold game/RSVP data

Couples GameBoard's high-frequency live event log to the happening, and makes
standalone (no-RSVP) games awkward.

### One record (invite carries the response)

Wrong cardinality — open links have many responses, responses are editable, and
attendance fields pollute the invite.

### Component-delegation registry

Extensions contribute Angular components a host renders via `NgComponentOutlet`.
Deferred for the invite landing — it cannot run for a logged-out stranger with
nothing installed. Still the right tool for the *rich in-app* experience after
conversion.

### A new RSVP model in rsvp-express

eventus already has the richest one; duplicating it is the very problem this
decision resolves.

### A separate rsvpus module for the response

Illusory separation now that `InviteResponse` is the universal invite-answer, and
it makes the denormalized counters non-transactional across modules.

## Consequences at Decision Time

Positive: one invite model instead of five and one response model instead of two;
the membership-vs-attendance boundary is structural; the public landing works for
logged-out strangers; `Target` + optional `Routing` make the invite reusable across
verticals, channels (web/bot/app), and white-label/localized deployments while the
common invite stores no routing at all. Costs: real teardown (delete the old
go-modules invitus, fold debtus invites in as `tracker`, reconcile contactus,
promote eventus's response model) and two records (Invite + InviteResponse) to keep
consistent with transactional counters. Each extension must implement the
`(Target, Role) → IInvitePresentation` projection for a rich landing; until then a
generic fallback card renders from the denormalized summary. The open-time
**routing resolver** (product/host derivation, locale + white-label routing, the
`snt.link/i/{inviteID}` shortlink) is deferred to its own future Feature. Open
follow-up: the first implementation slice + teardown order across the affected
repos.

## Observed Consequences

None observed yet.

## Affected Features

None at this time.

---
*This document follows the https://specscore.md/decision-specification*
