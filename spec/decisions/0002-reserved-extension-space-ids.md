---
format: https://specscore.md/decision-specification
status: Draft
---

# Decision: Reserved $-prefixed space IDs for extensions

**Status:** Draft
**Date:** 2026-06-29
**Owner:** alex
**Tags:** —
**Source Idea:** reserved-extension-space-ids
**Supersedes:** —
**Superseded By:** —

## Context

The Approved [`system-space-type`](../features/system-space-type/README.md)
Feature introduced `SpaceTypeSystem` — a platform-owned space whose module records
accept writes from **any authenticated user** (no membership), are **public-read**,
and whose **per-record** authorization is delegated to the owning extension. It is
the home for an extension's shared, cross-user records. But it explicitly left one
thing open: *how a System space is **addressed** (well-known id vs lookup)*.

Two concrete needs now force the question. [Decision
0001](0001-unified-invite-and-rsvp-model.md) places invitus's invites and
`InviteResponse` records — platform-owned, cross-user, spaceless — in a reserved
system space. And **GameBoard games** are global today
(`/ext/gameboard/games/{id}`, spaceless), which breaks the linkage validator
`WithRelated.ValidateWithKey`: it derives a `spaceID` from a record's key ancestry,
so a record carrying a `related` link (a game → its happening) **must** have a
space ancestor. Both want the same thing — a predictable, collision-proof way to
name and address an extension's one reserved system space — and giving spaceless
globals a real space ancestor resolves the `related` problem for free.

## Decision

A reserved system space is addressed by a **well-known `$`-prefixed id equal to the
owning extension's id**: `$invitus`, `$gameboard`, `$eventus`. The `$` sigil reads
as "**S**pace" — a platform-reserved space namespace. (1) The id is `$` + the
extension id; one reserved space per extension, with its module-global records at
the standard path `/spaces/$<ext>/ext/<ext>/{collection}/{id}`. (2) The `$` prefix
is reserved to the platform — the public create-space path rejects any id beginning
with `$`. (3) The id is **well-known**, derivable from the extension id with no
lookup. (4) A `$`-prefixed id **denotes `SpaceTypeSystem`**, so a bare id is
self-identifying.

## Rationale

The `$` convention answers `system-space-type`'s open addressing question with a
*well-known id* rather than a *lookup*, and it does so by reusing existing
machinery instead of inventing a spaceless special case. Crucially it dissolves the
`related`/linkage tension: previously-spaceless globals (invites, games) gain a
real **space ancestor**, so the validator works unchanged and *everything* is
uniformly space-scoped — one storage and access-control path (the existing
`dal4spaceus` system-space branch). The paths are human-readable and collision-proof
once `$` is reserved in id validation.

## Declined Alternatives

### Registry / lookup of reserved space ids

A platform doc mapping extension → reserved spaceID, read at runtime. Lost: extra
indirection and a read for an id that is trivially derivable — convention beats
configuration.

### Rely only on the existing `SpaceRef` type-prefix (`system!<id>`), no `$`

The type already encodes "system" in a `SpaceRef`. Lost: a *bare* space id — used
as a Firestore doc id and in collection paths — is not self-identifying without
carrying the type, and there is nothing for id-validation to reserve. The `$`
prefix makes a bare id recognizable and reservable.

### Opaque / random reserved ids

Lost: not predictable, forces a lookup, and is not human-readable in paths or logs.

## Consequences at Decision Time

Positive: resolves `system-space-type`'s addressing open question; gives invites
and games a real space ancestor so the `related` validator needs no special-casing;
uniform space-scoping on one code path; readable, predictable paths. Net-new work:
reserve the `$` prefix in space-id validation, provision the reserved spaces
(`$invitus`, `$gameboard`, …), and add `coretypes` helpers (`ReservedSpaceID`,
`IsReserved`, `$`-implies-System derivation). This **supersedes** GameBoard's
current spaceless `/ext/gameboard/games` root — greenfield, so migrate. Neutral:
the `/spaces/$invitus/ext/invitus/…` path carries an apparent `$invitus` +
`ext/invitus` redundancy, accepted deliberately to keep one uniform space-module
convention. Follow-ups settled in review (2026-06-29): a reserved space is
referenced by its **bare `$<ext>` id** (no `system!` type-prefix — `$` carries the
type); there is **exactly one** reserved space per extension (hard 1:1, no
`$<ext>-<purpose>` scheme); and the `$` sigil is confirmed (greenfield, no existing
`$`-prefixed ids, no backward-compatibility constraint).

## Observed Consequences

None observed yet.

## Affected Features

None at this time.

---
*This document follows the https://specscore.md/decision-specification*
