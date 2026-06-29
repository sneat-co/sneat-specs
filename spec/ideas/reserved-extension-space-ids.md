---
format: https://specscore.md/idea-specification
status: Specifying
---

# Idea: Reserved `$`-prefixed space IDs for extensions

**Status:** Specifying
**Date:** 2026-06-29
**Owner:** alex
**Promotes To:** reserved-extension-space-ids
**Supersedes:** —
**Related Ideas:** extends:system-space-type

## Problem Statement

How might we name and address an extension's one reserved system space by a predictable, collision-proof id — so backends and Firestore paths can reference it without a registry lookup, and so a bare space id is self-identifying as platform-reserved?

## Context

The `system-space-type` Feature (Approved) introduced `SpaceTypeSystem`: a platform-owned space whose module records accept writes from any authenticated user (no membership), are public-read, and whose per-record authorization is delegated to the owning extension. But it explicitly left open *how a System space is addressed (well-known id vs lookup)*. Two consumers now need an answer: ADR-0001 (unified invite & RSVP) stores invites and `InviteResponse` records — platform-owned, cross-user, spaceless — in a reserved space; and GameBoard games are global today (`/ext/gameboard/games`, spaceless), which breaks the linkage validator `WithRelated.ValidateWithKey` (it derives a `spaceID` from key ancestry, so any record carrying a `related` link must have a space ancestor). No id convention exists, and user space ids are currently unconstrained — so nothing reserves a namespace for the platform.

## Recommended Direction

Reserve the `$` prefix (read as "**S**pace") for platform-owned spaces. A reserved system space's id is `$` + the owning extension's id — `$invitus`, `$gameboard`, `$eventus`. The id is **well-known** (derived from the extension id, no lookup); a `$`-prefixed id **implies** `SpaceTypeSystem`; and the public create-space path **rejects** any requested id beginning with `$`. An extension's module-global records then live under the standard space-module path `/spaces/$<ext>/ext/<ext>/{collection}/{id}` — giving previously-spaceless globals a real space ancestor, so the `related`/linkage validator works unchanged.

## Alternatives Considered

- **Registry / lookup of reserved space ids.** A platform doc mapping extension →
  reserved spaceID, read at runtime. *Lost:* extra indirection and a read for an id
  that is trivially derivable from the extension id; convention beats configuration.

- **Rely only on the existing `SpaceRef` type-prefix (`system!<id>`), no `$`.** The
  type already encodes "system" in a `SpaceRef`. *Lost:* a *bare* space id — used as
  a Firestore doc id and in collection paths — is not self-identifying without
  carrying the type alongside it, and there is nothing for id-validation to reserve.
  A `$` prefix makes a bare id both recognizable and reservable.

- **Opaque / random reserved ids.** Provision reserved spaces with generated ids.
  *Lost:* not predictable, forces a lookup, not human-readable in paths or logs.

## MVP Scope

Reserve `$` in space-id validation (reject user/extension create-space ids beginning with `$`); add `coretypes` helpers (`ReservedSpaceID(extID) → "$"+extID`, `IsReserved`, and `$`-implies-System derivation); provision `$invitus` and `$gameboard` as `SpaceTypeSystem` spaces. Prove end-to-end: an invitus invite stored at `/spaces/$invitus/ext/invitus/invites/{id}` and a GameBoard game at `/spaces/$gameboard/ext/gameboard/games/{id}`, both with a valid space ancestor (the game's `related` link to its happening validates); and a user create-space request with a `$`-prefixed id is rejected.

## Not Doing (and Why)

- Multiple reserved spaces per extension — one-per-extension (`$<ext>`) for now; a suffix scheme can come later if a second is ever needed
- Private reads of reserved-space records — inherits public-read from `system-space-type`; private records are a separate, later concern
- Migrating or renaming existing (non-reserved) user spaces — out of scope
- A SpaceRef canonicalization overhaul — keep the existing `system!<id>` form; only add `$`-prefix recognition
- Any end-user UI for reserved spaces — they are platform plumbing, not user-facing

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| ~~Must-be-true~~ Confirmed | No existing space id begins with `$`, so reserving the prefix breaks nothing. | **Confirmed by owner (2026-06-29):** greenfield project, no `$`-prefixed space ids exist, no backward-compatibility constraint. |
| Must-be-true | A record under `/spaces/$<ext>/ext/<ext>/…` satisfies the linkage validator's `spaceID`-from-ancestry derivation. | Trace `WithRelated.ValidateWithKey` for a key at `/spaces/$gameboard/ext/gameboard/games/{id}`; confirm it resolves `spaceID = $gameboard`. |
| Should-be-true | A well-known id (no lookup) is sufficient — first consumers need exactly one reserved space each. | Confirm invitus and gameboard each need a single reserved space. |
| Should-be-true | The public create-space path is the single chokepoint where a `$`-prefixed id must be rejected. | Review the create-space validation path (`dto4spaceus`); confirm it is the only user-facing creation entry point. |
| Might-be-true | The same convention serves future shared namespaces (public events, shared catalogs). | Defer; revisit when a third consumer appears. |

## SpecScore Integration

- **New Features this would create:** `reserved-extension-space-ids` (the `$`
  prefix convention + reservation in create-space validation + `coretypes`
  helpers + `$`-implies-System derivation).
- **Existing Features affected:** `system-space-type` (this resolves its
  addressing open question); spaceus create-space validation (reject `$` ids);
  GameBoard game storage (moves from spaceless `/ext/gameboard` into `$gameboard`).
- **Dependencies:** builds on `system-space-type` (the space *type* and its auth
  semantics); consumed by ADR-0001 (invite/`InviteResponse` storage) and the
  GameBoard game↔happening `related` link.

## Open Questions

Settled in review (2026-06-29):

- **Canonical `SpaceRef` form** → the bare `$<ext>` id (no `system!` type-prefix
  for reserved spaces; `$` carries the type).
- **One vs many reserved spaces per extension** → **exactly one** per extension
  (hard 1:1); no `$<ext>-<purpose>` scheme.
- **Sigil choice / scan** → `$` confirmed (greenfield, no existing `$` ids, no
  backward-compatibility constraint).

None remaining.
