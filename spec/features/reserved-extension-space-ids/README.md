---
format: https://specscore.md/feature-specification
status: Draft
---

# Feature: System Namespace for Global Extension Records

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=explore) | [Edit](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=edit) | [Ask question](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=ask) | [Request change](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=request-change) |
**Status:** Draft
**Date:** 2026-06-29
**Owner:** alex
**Source Ideas:** reserved-extension-space-ids
**Supersedes:** —
**Grade:** B

> **Note:** This Feature originally specified per-extension `$`-prefixed reserved
> *spaces* (`$invitus`/`$gameboard`). [Decision 0002](../../decisions/0002-reserved-extension-space-ids.md)
> reversed that in favour of a spaceless system *namespace* at `/ext/`; the content
> below is retargeted accordingly. The slug `reserved-extension-space-ids` is
> retained for stable links.

## Summary

A spaceless **system namespace** for an extension's global, cross-user records:
they live at `/ext/{ext-id}/{collection}/{doc-id}` — **not** wrapped in any space.
A related-ref with no `@{space-id}` suffix (`{ext-id}/{collection}/{doc-id}`,
ext-id mandatory) resolves there; the linkage validator and resolver gain a
spaceless branch; and the `SpaceTypeSystem` access semantics are lifted from a
system *space* to this namespace. Resolves the addressing open question left by
[`system-space-type`](../system-space-type/README.md).

## Problem

The `system-space-type` Feature gave us records any authenticated user may write
and anyone may read, with per-record authorization owned by the extension — the
home for shared, cross-user records. But it left **addressing** unspecified: *where
do those records live, and how is that home referenced?* Three consumers need the
answer now — invitus (invites + `InviteResponse` records, per ADR-0001), GameBoard
(games), and ToGethered (spots/intents/subscriptions). All are platform-owned,
cross-user, and **belong to no space**, yet they break the `related`/linkage
validator `WithRelated.ValidateWithKey`, which derives a `spaceID` from key ancestry
and so currently requires every linked record to have a space ancestor.

## Behavior

### Storage

#### REQ: spaceless-system-storage

Global / system extension records MUST live **spaceless** at the root path
`/ext/{ext-id}/{collection}/{doc-id}` (nested subcollections allowed, e.g.
`/ext/{ext-id}/spots/{spotID}/days/{date}`). They MUST NOT be wrapped in any space.
Space-owned records are unchanged: `/spaces/{space-id}/ext/{ext-id}/...`. The
`{ext-id}` sublevel names the module **storing** the record, which is usually — but
need not be — the records' owning extension: the namespace MAY host overlay records
from other modules. For example, when a ToGethered formal event lazily upgrades to
the Eventus engine, its Eventus overlay lands at `/ext/eventus/events/{happeningID}`.
There is **no data migration**: invites, games and intents already live at
`/ext/{ext-id}/...` today.

### Related-ref format

#### REQ: spaceless-ref-format

A system-namespace related-ref MUST be `{ext-id}/{collection}/{doc-id}` — the
**ext-id is mandatory** (it is the namespace root) and there is **no `@{space-id}`
suffix**. A space-bound ref MUST keep its suffix: `{ext-id}/{collection}/{doc-id}@{space-id}`.
The presence or absence of `@{space-id}` is the **sole discriminator**: absent ⇒
resolve under `/ext/{ext-id}/...`; present ⇒ resolve under
`/spaces/{space-id}/ext/{ext-id}/...`. No `$` sigil and no reserved spaces are
introduced (`$invitus`/`$gameboard`/`$togethered` are not created).

### Validator & resolver

#### REQ: spaceless-validator-resolver-branch

The linkage validator MUST recognise a spaceless storage key (`/ext/{ext-id}/...`,
no space ancestor) as the system namespace instead of walking off the top of the
key: `spaceID` becomes optional (empty ⇒ system namespace) in
`RelatedItemKey.Validate`, and `WithRelated.ValidateWithKey` accepts the spaceless
shape. The ref→path resolver MUST, when a ref has no `@{space-id}`, build
`/ext/{ext-id}/...` directly (a spaceless key-builder that does **not** call
`NewSpaceKey`, which panics on empty). The TypeScript mirror (`with-related.ts`)
MUST agree: `spaceID` optional, serialization omits `@` when there is no space.

### Access control

#### REQ: lifted-system-namespace-acl

The `SpaceTypeSystem` access semantics — **public-read**, **any-authenticated-write**
(no membership), and **per-record authorization delegated to the owning extension** —
MUST be **lifted from a system *space* to the system *namespace*** at `/ext/`. It is
the same authorization logic relocated, not reinvented: spaceus permits the write to
the namespace; deciding whether the caller may mutate a **specific record** remains
the owning extension's responsibility.

## Acceptance Criteria

### AC: system-records-stored-spaceless (verifies REQ:spaceless-system-storage)

**Scenario:** a global extension record lives at the spaceless root path
**Given** an invitus invite and a GameBoard game
**When** they are stored as system records
**Then** they live at `/ext/invitus/invites/{id}` and `/ext/gameboard/games/{id}`
respectively, with no `/spaces/...` ancestor.

### AC: spaceless-ref-has-no-space-suffix (verifies REQ:spaceless-ref-format)

**Scenario:** a system-namespace ref omits the space suffix
**Given** a related-ref to a system-namespace record
**When** the ref is serialized
**Then** it is `{ext-id}/{collection}/{doc-id}` (ext-id present, no `@{space-id}`),
and a space-bound ref retains its `@{space-id}` suffix.

### AC: missing-suffix-resolves-to-namespace (verifies REQ:spaceless-ref-format)

**Scenario:** the suffix is the discriminator
**Given** a ref with no `@{space-id}`
**When** the resolver builds its storage path
**Then** it resolves under `/ext/{ext-id}/...`; a ref carrying `@{space-id}` resolves
under `/spaces/{space-id}/ext/{ext-id}/...`.

### AC: spaceless-record-validates-without-space-ancestor (verifies REQ:spaceless-validator-resolver-branch)

**Scenario:** a spaceless record satisfies the linkage validator
**Given** a record stored at `/ext/gameboard/games/{id}` that carries a `related` link to a happening
**When** `WithRelated.ValidateWithKey` inspects the record's key ancestry
**Then** it recognises the spaceless system namespace (empty `spaceID`) and validation succeeds without requiring a space ancestor.

### AC: namespace-write-open-to-authenticated (verifies REQ:lifted-system-namespace-acl)

**Scenario:** the lifted ACL governs `/ext/` records
**Given** a system-namespace record under `/ext/{ext-id}/...`
**When** an authenticated, non-member user (authorized by the owning extension) writes it
**Then** the write succeeds (public-read, any-authenticated-write), and an unauthenticated write is rejected.

## Rehearse Integration

The ACs are testable Go-level behaviors against the `sneat-libs`/`sneat-core-modules`
linkage package (`RelatedItemKey.Validate`, ref serialization, and
`WithRelated.ValidateWithKey` over a spaceless key), the ref→path resolver
(`dbo4spaceus.NewSpaceModuleItemKeyFromItemRef`), and the `dal4spaceus`
`SpaceTypeSystem` authorization hook lifted to `/ext/`; with a parallel check on the
TypeScript mirror (`with-related.ts`). Rehearse stubs are deferred to plan/implement
time, when the spaceless linkage branch and the lifted ACL land; the ACs are written
to be directly executable as table tests at that point.

## Open Questions

Settled in review (2026-06-29), folded into Behavior above:

- **Space vs namespace** → a spaceless **namespace** at `/ext/`; no `$<ext>` reserved
  spaces and no `$sneat` shared space (see
  [Decision 0002](../../decisions/0002-reserved-extension-space-ids.md)). See
  `REQ:spaceless-system-storage`.
- **Ref format for system records** → `{ext-id}/{collection}/{doc-id}` with the
  ext-id mandatory and **no** `@{space-id}` suffix; absence of the suffix is the
  discriminator. See `REQ:spaceless-ref-format`.
- **Migration** → none; current `/ext/{ext-id}/...` storage is blessed as-is. See
  `REQ:spaceless-system-storage`.

None remaining.

---
*This document follows the https://specscore.md/feature-specification*
