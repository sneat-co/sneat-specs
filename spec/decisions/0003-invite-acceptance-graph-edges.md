---
format: https://specscore.md/decision-specification
status: In Review
---
# Decision: Invite acceptance — graph edges, membership, and authority

**Status:** In Review
**Date:** 2026-06-29
**Owner:** alex
**Tags:** —
**Source Idea:** —
**Supersedes:** —
**Superseded By:** —

## Context

The end-goal of the invite/RSVP work ([decision
0001](0001-unified-invite-and-rsvp-model.md)) is **building the relationship
graph**, not conversion. Every Sneat user always has a **family space** (virtual
until it holds data, but always present in the UI) — their **personal life hub**,
the convergence point where graph data and events from *all* products (gameboard,
eventus, togethered) and *all* channels (web, Telegram bot) accumulate into one
calendar. The mini-products are feeders; the family space is the destination.

Decision 0001 settled the invite/`InviteResponse`/`Routing`/`Target` model and that
the first vertical is the **role-rich GameBoard participant flow**, but left
unspecified *what an accepted, role-tagged invite actually deposits*. This decision
defines that — the graph edge, the space membership, and the authority each role
grants — using GameBoard's roles as the driving example. It is shared
infrastructure: eventus and ToGethered also produce graph edges and share the same
open relationship-role-label question.

## Decision

### 1. Accept-time effects are driven by the invite role

An accepted, role-tagged invite deposits up to three effects, by role:
**(a) a relationship/membership edge**, **(b) optional space membership**, and
**(c) optional authority**.

| Role | Edge | Space membership | Authority |
|---|---|---|---|
| **player** | player → team (member); player ↔ player (teammate) | team space | — |
| **coach** | coach → team (coaches) | team space | — |
| **team-admin / coordinator** | admin → team (administers) | team space | **team-level admin** (durable): create/schedule fixtures, edit settings, manage roster & invites — = gameboard's existing "organizer/creator", the crew-based replacement of its creator-only check |
| **parent / guardian** | parent ↔ player (in the **family** graph — see §3) | — (team *contact*, not member) | — |
| **scorekeeper** | crew → game | — | **per-game append**: score / team-foul / substitution |
| **timekeeper** | crew → game | — | **per-game append**: clock / period / possession / timeout / status |
| **judge** | crew → game | — | **per-game append**: judge-ruling / correction |

Two authority tiers: **team-admin** = durable, team-scoped admin authority (and is
the primary inviter); **crew** = ephemeral, per-game append authority, already gated
by gameboard's `IsAuthorized`. player/coach/parent carry no authority.

### 2. Parent/guardian accept flow

A **player** is a team member/contact that may be a userless contact, a **minor
user**, or an **adult user**. A **parent** ("parent" = any legal guardian or
caretaker, e.g. an older sibling) becomes a **team contact with role `parent`**,
linked to the player (team member, role `player`). The contact may pre-exist (a
personal invite's `ToSpaceContactID`) and gets the accepting user's userID
associated on accept (invitus's existing claim).

- **Acceptor may differ from the invited entity.** An invite can be sent to a
  player and **accepted by a guardian**: the guardian self-creates their `parent`
  contact, claims it with their userID, and **creates/links the player profile(s)**
  (siblings → multiple players in the same family). This departs from invitus's
  current assumption that the invited contact *is* the acceptor — the accept flow
  must support acceptor-creates-a-different-contact-and-children.
- **Minor rule.** The only structural difference between a minor and an adult
  player is that **a minor player MUST have ≥1 guardian linked**; an adult needn't.
  Any user (minor or adult) may self-accept; a guardian accepts on behalf of a
  userless minor (or by choice).
- Parent/child is **many-to-many** (reuse the existing family-space model): many
  guardians → one player (mum/dad/older sibling); one guardian → many players.

### 3. The family graph (private life), with the team as a bridge

Parent↔player and parent↔parent relationships live in **family spaces (private
life)**, **not** the team space. The team is only the *bridge* that introduces two
families; the resulting connection is owned by private life and **persists beyond
the team**. On accept, the user's (always-present, possibly virtual) family space
is materialized and the family graph begins building. A child may belong to
**multiple family spaces** (split/blended households); a signed-in user picks which
family space — or both — the team-seeded connections attach to.

**Inter-family edge — `family ↔ family`, formed as a _suggestion_.** When two
families' children are teammates, a **suggested** connection is seeded by that
shared-team context ("your child and the Smiths' are on the Hawks — connect?") and
promoted to a real edge on opt-in — not a silent auto-edge. Rationale: graph growth
is the goal, but it is private life, so a context-seeded suggestion (the
"people-you-may-know" pattern) grows the graph *with consent*. **(Default — flagged
in Open Questions; switch to auto-formed if consent is not required.)**

### 4. Visibility

Team roster and contacts are visible **only to other team members and their
parents**, by default. Names may be shown **brief** (`First L.`) or **full** by
consent. (Same visibility-setting shape as Decision 0001's social-proof control.)

### 5. Duplicate identities are tolerated, reconciled by merge

A person signing in via different auth providers/channels (e.g. Gmail, then later
accepting a Telegram-bot invite) creates **duplicate** user accounts + family
spaces. We do **not** try to prevent this at creation (cross-provider identity is
undetectable up front); instead a **space-merge** capability reconciles them — link
another auth provider, pick the **main** space (most graph data, oldest as
tiebreaker), and merge the duplicate in. Space-merge is generic platform
infrastructure and is captured **separately** (planned Decision 0004); the GameBoard
vertical ships allowing duplicates and merge cleans up later.

## Rationale

Reusing what exists keeps this tractable: roles map to contactus contacts+roles,
the claim-on-accept is invitus's existing flow, parent/child many-to-many and the
family space already exist, and the crew roles are already gameboard's
`Source`/`IsAuthorized`. The one genuinely new structural choice is putting
parent↔parent in the **family** graph rather than the team — which is correct
because it is private life and must outlive any one team. Suggestion-over-auto for
the inter-family edge trades a little graph-growth speed for consent, which private
life demands. Tolerating duplicates and merging is the only workable answer to
cross-provider identity.

## Declined Alternatives

### Parent↔parent as a team-space edge

Rejected: the connection is private life and must persist after the child leaves
the team; the team is a bridge, not the owner.

### Auto-formed inter-family edge

Rejected as the default: silently linking strangers' private family graphs because
their kids are teammates is invasive. Kept as the override if product decides
consent is unnecessary.

### Prevent duplicate accounts at creation

Rejected: cross-provider identity (Gmail vs Telegram) is undetectable at signup;
post-hoc space-merge is the workable path.

### Family space only for parents/players

Rejected: **every** user always has a family space (the personal life hub); coach,
admin, and crew get one too — it is the calendar/graph convergence point across all
products.

## Consequences at Decision Time

The invitus accept flow grows beyond claim-the-invited-contact: it must support
acceptor ≠ invited, guardian-creates-player(s), and the minor ≥1-guardian rule
(which needs a minor/DOB signal on the profile). Family-space materialization and
the family graph become part of accept. The inter-family suggestion engine and the
space-merge capability are new, separable workstreams (merge → Decision 0004). Crew
acceptance must write a per-game authority record into gameboard's `IsAuthorized`
path; team-admin acceptance supersedes gameboard's creator-only settings check.
Positive: every accepted invite now feeds the relationship graph and the user's
single life-hub calendar — the actual end-goal — while reusing existing
contactus/invitus/family-space machinery.

## Observed Consequences

None observed yet.

## Affected Features

None at this time.

## Open Questions

- **Inter-family edge: suggested (default here) vs auto-formed.** Confirm consent is
  required (suggestion) or not (auto).
- **Minor determination.** What signals "minor" to enforce the ≥1-guardian rule — a
  DOB or `isMinor` flag, and who sets it (player vs guardian)?
- **Minor self-accept → guardian step.** A minor user who self-accepts cannot
  satisfy ≥1 guardian alone: spawn a reverse guardian invite, or allow accept but
  flag the player profile incomplete until a guardian links? (Lean: allow + flag +
  prompt.)
- **Compliance.** Minor *users* raise parental-consent / COPPA-K questions; the
  mandatory-guardian rule is the natural attach point — design later.
- **Relationship-role-label taxonomy.** The exact `linkage` role labels + registered
  opposites (parent-of/child-of, coach-of/coached-by, teammate=symmetric,
  crew/game) — shared with eventus and ToGethered (both have it open); settle once,
  platform-wide.
