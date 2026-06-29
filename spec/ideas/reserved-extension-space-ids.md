---
format: https://specscore.md/idea-specification
status: Specifying
---

# Idea: Spaceless system namespace for global extension records

**Status:** Specifying
**Date:** 2026-06-29
**Owner:** alex
**Promotes To:** reserved-extension-space-ids
**Supersedes:** —
**Related Ideas:** extends:system-space-type

> **Note:** This Idea originally proposed per-extension `$`-prefixed reserved
> *spaces* (`$invitus`/`$gameboard`/`$eventus`). On review
> ([Decision 0002](../decisions/0002-reserved-extension-space-ids.md)) that
> direction was reversed in favour of a spaceless system *namespace* at `/ext/`.
> The slug is retained for stable links; the content below is retargeted to the
> spaceless model.

## Problem Statement

How might we give an extension's global, cross-user records (invites, games, spots) a first-class home — one that backends and Firestore paths can address without a registry lookup, and that satisfies the linkage validator — when those records belong to **no space**?

## Context

The `system-space-type` Feature (Approved) introduced `SpaceTypeSystem`: a platform-owned space whose module records accept writes from any authenticated user (no membership), are public-read, and whose per-record authorization is delegated to the owning extension. But it left one thing open: *how that home is addressed.* Three consumers now need an answer: ADR-0001 (unified invite & RSVP) stores invites and `InviteResponse` records — platform-owned, cross-user, spaceless; GameBoard games are global today (`/ext/gameboard/games`, spaceless); and ToGethered's spots/intents/subscriptions (`/ext/togethered/...`) are likewise spaceless. All three break the linkage validator `WithRelated.ValidateWithKey`, which derives a `spaceID` from key ancestry, so any record carrying a `related` link is currently required to have a space ancestor.

## Recommended Direction

Make **spaceless** a first-class shape rather than disguise it as a space. Global / system extension records live spaceless at the root path `/ext/{ext-id}/{collection}/{doc-id}` (nested subcollections allowed) — they are **not** wrapped in any space. A related-ref to such a record is `{ext-id}/{collection}/{doc-id}` (the ext-id is mandatory — it is the namespace root — and there is **no `@{space-id}` suffix**); a space-bound ref keeps its suffix, `{ext-id}/{collection}/{doc-id}@{space-id}`. Absence of `@{space-id}` is the sole discriminator that means "the system namespace." The `SpaceTypeSystem` access semantics (public-read, any-authenticated-write, per-record authorization delegated to the owning extension) are **lifted from a system *space* to the system *namespace*** at `/ext/`. There is **no data migration**: invites, games and intents already live at `/ext/{ext-id}/...` today.

## Alternatives Considered

- **Per-extension reserved `$<ext>` spaces (this Idea's own earlier draft).** Give
  each extension a `$<ext>` `SpaceTypeSystem` space and store records at
  `/spaces/$<ext>/ext/<ext>/…`. *Lost:* it models inherently spaceless data as a
  member of a space no one belongs to, and the `$<ext>/ext/<ext>` path is redundant
  (the ext id appears twice); the space exists only to satisfy the validator.

- **Single shared `$sneat` system space.** One platform space holding every
  extension's system records at `/spaces/$sneat/ext/<ext>/…`. *Lost:* still wears
  the space costume — `$sneat` is a space no one belongs to and the path still
  carries a `/spaces/$sneat` prefix whose only purpose is machinery.

- **Spaceless storage with a sentinel ref (`…@$` / `…@$sneat`).** Store spaceless at
  `/ext/…` but keep a sentinel space token in the ref for format uniformity.
  *Lost:* a ref that names a space but resolves to none is misleading, and it saves
  no code (the resolver must special-case the sentinel anyway).

## MVP Scope

Make the linkage core accept the spaceless shape: `spaceID` becomes optional (empty ⇒ system namespace) in `RelatedItemKey.Validate`; ref serialization omits the `@{space}` suffix when there is no space; `WithRelated.ValidateWithKey` recognises a spaceless storage key (`/ext/{ext-id}/...`, no space ancestor) as the system namespace; the ref→path resolver builds `/ext/{ext-id}/...` directly for a space-less ref; and the `SpaceTypeSystem` authorization hook is lifted to authorize records under `/ext/`. Prove end-to-end: an invitus invite at `/ext/invitus/invites/{id}` and a GameBoard game at `/ext/gameboard/games/{id}`, both spaceless, with the game's `related` link to its happening validating; and the TS mirror (`with-related.ts`) agreeing on serialization.

## Not Doing (and Why)

- A `$`-prefix space-id reservation or `$<ext>` reserved spaces — reversed by Decision 0002; spaceless `/ext/` is the home, no sigil is introduced
- Private reads of system-namespace records — inherits public-read from `system-space-type`; private records are a separate, later concern
- Migrating or renaming existing user spaces — out of scope (and system records need no migration; they already live at `/ext/{ext-id}/...`)
- A SpaceRef canonicalization overhaul — only the linkage layer gains a spaceless branch
- Any end-user UI for system-namespace records — they are platform plumbing, not user-facing

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| ~~Must-be-true~~ Confirmed | System extension records already live at `/ext/{ext-id}/...`, so blessing that location needs no data migration. | **Confirmed by owner (2026-06-29):** invites, games and ToGethered intents are stored at `/ext/{ext-id}/...` today. |
| Must-be-true | The linkage validator can recognise a spaceless `/ext/{ext-id}/...` key as the system namespace instead of requiring a space ancestor. | Trace `WithRelated.ValidateWithKey` for a key at `/ext/gameboard/games/{id}`; confirm a spaceless branch validates without a `spaceID`. |
| Must-be-true | The ref→path resolver can build `/ext/{ext-id}/...` without calling `NewSpaceKey` (which panics on empty space). | Trace `dbo4spaceus.NewSpaceModuleItemKeyFromItemRef` for a ref with no `@{space-id}`. |
| Should-be-true | The two storage shapes (space-bound and system-namespace) coexist cleanly, branched on the `@{space-id}` discriminator. | Confirm serialization round-trips both shapes in Go and the TS mirror. |
| Might-be-true | The same namespace serves future spaceless globals (public events, shared catalogs). | Defer; revisit when a fourth consumer appears. |

## SpecScore Integration

- **New Features this would create:** `reserved-extension-space-ids` (retargeted —
  the spaceless `/ext/{ext-id}/...` storage + the suffix-less ref format + the
  validator/resolver spaceless branch + the lifted `SpaceTypeSystem` ACL).
- **Existing Features affected:** `system-space-type` (reframed from system *space*
  to system *namespace*); the `linkage` package and its TS mirror (`with-related.ts`)
  gain a spaceless branch.
- **Dependencies:** builds on `system-space-type` (the access semantics, lifted to
  `/ext/`); consumed by ADR-0001 (invite/`InviteResponse` storage) and the GameBoard
  game↔happening `related` link.

## Open Questions

Settled in review (2026-06-29):

- **Space vs namespace** → **spaceless namespace** at `/ext/`; no `$<ext>` reserved
  spaces, no `$sneat` shared space (see Decision 0002).
- **Ref format for system records** → `{ext-id}/{collection}/{doc-id}` with the
  ext-id mandatory and **no** `@{space-id}` suffix; absence of the suffix is the
  discriminator.
- **Migration** → none; current `/ext/{ext-id}/...` storage is blessed as-is.

None remaining.
