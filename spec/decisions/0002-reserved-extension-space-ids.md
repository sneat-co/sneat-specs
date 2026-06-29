---
format: https://specscore.md/decision-specification
status: In Review
---

# Decision: Spaceless system namespace for global extension records

**Status:** In Review
**Date:** 2026-06-29
**Owner:** alex
**Tags:** —
**Source Idea:** —
**Supersedes:** —
**Superseded By:** —

> **Note:** This Decision reverses its own earlier draft (which proposed
> per-extension `$`-prefixed reserved *spaces*, e.g. `$invitus`/`$gameboard`).
> The filename slug `reserved-extension-space-ids` is retained for stable links
> and is now historical. **This Decision is the single authoritative record for
> the spaceless system namespace**: the earlier `reserved-extension-space-ids`
> Idea/Feature was removed as redundant with it, and the `system-space-type`
> Feature is superseded (see *Consequences*).

## Context

The Approved [`system-space-type`](../features/system-space-type/README.md) Feature
introduced `SpaceTypeSystem` — a platform-owned space whose module records accept
writes from **any authenticated user** (no membership), are **public-read**, and
whose **per-record** authorization is delegated to the owning extension. It is the
home for an extension's shared, cross-user records, and it left one thing open:
*how that home is addressed.*

Three concrete cases need that home: invitus's invites + `InviteResponse`
([Decision 0001](0001-unified-invite-and-rsvp-model.md)), **GameBoard games**
(`/ext/gameboard/games/{id}` today), and **ToGethered**'s spots / intents /
subscriptions (`/ext/togethered/...` today). All three are platform-owned,
cross-user, and **belong to no space**.

The earlier draft of this Decision proposed giving each a per-extension reserved
*space* (`$<ext>`) so the records gained a space ancestor — which the linkage
validator `WithRelated.ValidateWithKey` currently requires (it derives a `spaceID`
from a record's key ancestry, so a record carrying a `related` link must sit under
a space). On review that was rejected: it models inherently spaceless data as a
member of a space that no one belongs to, and yields a redundant
`/spaces/$<ext>/ext/<ext>/…` path whose only job is to satisfy machinery. This
Decision takes the opposite, simpler route — make spaceless a first-class shape
rather than disguise it as a space.

## Decision

Global / system extension records live **spaceless** at the root path
`/ext/{ext-id}/{collection}/{doc-id}` (nested subcollections allowed, e.g.
`/ext/{ext-id}/spots/{spotID}/days/{date}`). They are **not** wrapped in any space.

1. **Storage.** System-level records: `/ext/{ext-id}/...`. Space-owned records are
   unchanged: `/spaces/{space-id}/ext/{ext-id}/...`.
2. **Related-ref format.** A system-namespace ref is `{ext-id}/{collection}/{doc-id}`
   — **the ext-id is mandatory** (it is the namespace root) and there is **no
   suffix**. A space-bound ref keeps its suffix: `{ext-id}/{collection}/{doc-id}@{space-id}`.
   The presence or absence of `@{space-id}` is the sole discriminator: absent ⇒
   resolve under `/ext/{ext-id}/...`; present ⇒ resolve under
   `/spaces/{space-id}/ext/{ext-id}/...`.
3. **Access control is per-record.** `/ext/` is a **storage location, not an
   authorization scope**: there is **no namespace- or extension-level access
   policy** — no blanket public-read, no blanket any-authenticated-write. Each
   record carries/owns its own access-control, and the authorization gate evaluates
   **that record's** policy on every read and write. (This replaces the earlier
   `SpaceTypeSystem` space-/namespace-level model — see *Declined Alternatives*.)
4. **No `$` sigil, no reserved spaces.** `$invitus` / `$gameboard` / `$togethered`
   are not created; the `$`-prefix reservation in space-id validation is not needed.

## Rationale

These records belong to no space, and the model should say so. Spaceless storage is
**semantically honest**, the paths are **shorter** (`/ext/togethered/...` vs
`/spaces/$togethered/ext/togethered/...`), the **ext-id is the natural first-class
namespace**, and — decisively — there is **no migration**: invites, games and
intents already live at `/ext/{ext-id}/...` today, so this Decision *blesses the
current location* instead of moving data into a synthetic space.

The cost is a one-time, bounded change to the linkage core: the validator and
resolver gain a single spaceless branch. Authorization is enforced **per-record** —
each record owns its read/write policy — rather than by a space type or a
namespace-wide rule; that also removes the standing bypass a blanket
"any-authenticated-write at `/ext/`" would create. This is cheaper, and
conceptually cleaner, than permanently disguising spaceless data as a fake space on
every read, write and path.

## Declined Alternatives

### Per-extension reserved `$<ext>` spaces (this Decision's own earlier draft)

Give each extension a `$<ext>` `SpaceTypeSystem` space and store records at
`/spaces/$<ext>/ext/<ext>/…`. Lost: it models spaceless data as a space member, and
the `$<ext>/ext/<ext>` path is redundant — the extension id appears twice and the
space exists only to satisfy the validator. Reuses machinery at the cost of lying
about ownership.

### Single shared `$sneat` system space

One platform space `$sneat` holding every extension's system records at
`/spaces/$sneat/ext/<ext>/…`. Removes the doubling and keeps the validator
unchanged, but still wears the space costume: `$sneat` is a "space" no one belongs
to, and the path still carries a `/spaces/$sneat` prefix whose only purpose is
machinery.

### Spaceless storage with a sentinel ref (`…@$` / `…@$sneat`)

Store spaceless at `/ext/…` but keep a sentinel space token in the ref for format
uniformity. Lost: a ref that *names* a space but resolves to no space is
misleading, and it saves no code (the validator reads the spaceless storage key and
the resolver must special-case the sentinel regardless). Clean absence of the
suffix is more honest than a token that lies.

### Namespace-/space-type-level ACL (blanket `/ext/` policy or `SpaceTypeSystem`)

Govern access by a rule attached to the *namespace* or a space *type* — e.g.
`SpaceTypeSystem`'s blanket public-read + any-authenticated-write, with only
per-record *write* delegated. Lost: a namespace-wide "any authenticated user may
write here" is a **standing authorization bypass** — a request authorized for one
context can reach unrelated `/ext/` records (exactly the cross-tenant write a code
review caught). Access is therefore decided **per-record**, never by the namespace
or a space type.

## Consequences at Decision Time

**Supersedes** the `$`-prefixed reserved-space direction in its entirety — no
`$invitus`/`$gameboard`/`$togethered`, no `$`-prefix reservation in space-id
validation, no per-extension reserved-space provisioning. To reconcile: the
`reserved-extension-space-ids` Idea/Feature was **removed** as redundant with
this Decision (this Decision is now the contract for the spaceless namespace);
the [`system-space-type`](../features/system-space-type/README.md) Idea/Feature is
**superseded** (its space-type access model is replaced by per-record
authorization); and
[Decision 0001](0001-unified-invite-and-rsvp-model.md) and
[Decision 0003](0003-invite-acceptance-graph-edges.md) drop their `$<ext>` space
references in favour of `/ext/{ext-id}/...`.

**Code** (canonical linkage package: `sneat-core-modules/linkage`, with the TS
mirror in `sneat-libs`). A bounded change with the following touch-points:
- `RelatedItemKey.Validate` — `spaceID` becomes optional (empty ⇒ system namespace).
- Ref serialization (`RelatedItemKey.String`, `NewFullItemRef`) — omit the `@{space}`
  suffix when there is no space.
- `WithRelated.ValidateWithKey` — recognise a spaceless storage key
  (`/ext/{ext-id}/...`, no space ancestor) as the system namespace instead of
  walking off the top of the key.
- The ref→path resolver (`dbo4spaceus.NewSpaceModuleItemKeyFromItemRef` /
  `NewSpaceModuleItemKey`, used by `facade4linkage`) — when the ref has no space,
  build `/ext/{ext-id}/...` directly (a spaceless key-builder that does **not** call
  `NewSpaceKey`, which panics on empty).
- Per-record authorization for `/ext/` records — enforce each record's own
  read/write policy (no namespace/space-type blanket); the legacy `dal4spaceus`
  `SpaceTypeSystem` space-type is retired, not extended. Security-sensitive; tracked
  as a separate follow-up (see the code PRs).
- TypeScript mirror (`with-related.ts`) — `ISpaceModuleItemRef.spaceID` optional;
  `getLongRelatedItemID` omits `@` when no space.
- Tests for both shapes (space-bound and system-namespace).

**Positive:** honest semantics; shorter paths; ext-id as a first-class namespace;
**zero data migration** (current `/ext/{ext-id}/...` storage is blessed as-is).
**Costs / neutral:** two storage shapes now coexist (space-bound and
system-namespace), so the linkage layer branches on the `@{space-id}` discriminator;
the ext-id is **mandatory** on every related-ref.

## Observed Consequences

None observed yet.

## Affected Features

- `reserved-extension-space-ids` — **removed** (Idea + Feature): redundant with this Decision, which is now the authoritative record for the spaceless namespace.
- [`system-space-type`](../features/system-space-type/README.md) — **superseded**: its space-type-level ACL is replaced by per-record authorization.
- [`eventus/mini-products/togethered`](https://github.com/sneat-co/backstage) (backstage) — records at `/ext/togethered/...`.
- Decisions [0001](0001-unified-invite-and-rsvp-model.md) and [0003](0003-invite-acceptance-graph-edges.md) — drop `$<ext>` space references.

---
*This document follows the https://specscore.md/decision-specification*
