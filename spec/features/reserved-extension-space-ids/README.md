---
format: https://specscore.md/feature-specification
status: Draft
---

# Feature: Reserved Extension Space IDs

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=explore) | [Edit](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=edit) | [Ask question](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=ask) | [Request change](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/reserved-extension-space-ids?op=request-change) |
**Status:** Draft
**Date:** 2026-06-29
**Owner:** alex
**Source Ideas:** reserved-extension-space-ids
**Supersedes:** —
**Grade:** B

## Summary

A `$`-prefixed, well-known space-id convention for an extension's one reserved
`SpaceTypeSystem` space (`$invitus`, `$gameboard`, …): the `$` prefix is reserved
to the platform, a `$`-prefixed id implies the System type, and the id is derived
from the extension id with no lookup. Resolves the addressing open question left
by [`system-space-type`](../system-space-type/README.md).

## Problem

The `system-space-type` Feature gave us a space whose records any authenticated
user may write and anyone may read, with per-record authorization owned by the
extension — the home for shared, cross-user records. But it left **addressing**
unspecified: *how is a reserved System space named and found?* Without a
convention, every consumer would invent its own id or a lookup, and nothing
reserves a namespace, so a user could mint a space that collides with a reserved
one. Two consumers need the answer now — invitus (invites + `InviteResponse`
records, per ADR-0001) and GameBoard (games, today spaceless, which breaks the
`related`/linkage validator that requires a space ancestor).

## Behavior

### Reserved id format

#### REQ: dollar-prefix-sigil

A reserved system space's id MUST be the owning extension's id prefixed with `$`
(e.g. extension `invitus` → space id `$invitus`; `gameboard` → `$gameboard`;
`togethered` → `$togethered`). The `$` sigil denotes a platform-reserved space
("**S**pace"). An extension has **exactly one** reserved space (a hard 1:1 — no
multi-reserved-space or `$<ext>-<purpose>` scheme), holding records under the
standard space-module path `/spaces/$<ext>/ext/<module>/{collection}/{id}`.
The `<module>` sublevel names the module **storing** the record, which is usually —
but need not be — the space's owning extension: a reserved space MAY host overlay
records from other modules. For example, when a ToGethered formal event lazily
upgrades to the Eventus engine, its Eventus overlay lands at
`/spaces/$togethered/ext/eventus/events/{happeningID}` — `togethered` owns the
space, `eventus` is the storing module. (This is why the `/ext/<module>/` sublevel
is retained rather than collapsed into the reserved space id.)

### Reservation

#### REQ: dollar-prefix-reserved

The `$` prefix MUST be reserved to the platform. The public create-space path
MUST reject any request whose space id begins with `$`. (Composes with
`system-space-type`'s platform-only provisioning — reserved spaces are
provisioned by the platform, never by an end user.)

### Addressing

#### REQ: well-known-no-lookup

The reserved space id for an extension MUST be derivable from the extension id
alone (`$` + extension id), requiring **no** registry read or lookup. A helper
(e.g. `coretypes.ReservedSpaceID(extID)`) MUST return the reserved id, and a
predicate (e.g. `IsReserved`) MUST report whether a given space id is reserved.

### Type derivation

#### REQ: dollar-implies-system

A space id beginning with `$` MUST denote a `SpaceTypeSystem` space, and the
platform MUST derive the System type from the `$` prefix. A reserved space is
therefore referenced by its **bare `$<ext>` id** (e.g. `$invitus`) — the legacy
`SpaceRef` type-prefix form (`system!$invitus`) MUST NOT be used for reserved
spaces, because the `$` already carries the type.

## Acceptance Criteria

### AC: reserved-id-is-dollar-ext (verifies REQ:dollar-prefix-sigil)

**Scenario:** the reserved id is the extension id with a `$` prefix
**Given** the helper that computes a reserved space id
**When** it is called with extension id `invitus`
**Then** it returns `$invitus`, and with `gameboard` it returns `$gameboard`.

### AC: user-cannot-create-dollar-id (verifies REQ:dollar-prefix-reserved)

**Scenario:** end users cannot mint `$`-prefixed spaces
**Given** the public create-space path
**When** an end user requests creation of a space whose id begins with `$`
**Then** the request is rejected.

### AC: reserved-id-derivable-without-lookup (verifies REQ:well-known-no-lookup)

**Scenario:** addressing needs no registry
**Given** only an extension id
**When** the reserved space id is computed
**Then** it is produced from the extension id alone, with no datastore read, and
`IsReserved` returns true for the result.

### AC: dollar-id-implies-system-type (verifies REQ:dollar-implies-system)

**Scenario:** a `$`-prefixed id is recognized as a System space
**Given** a space id beginning with `$`
**When** the platform resolves the space's type from the id
**Then** it is `SpaceTypeSystem`.

### AC: record-under-reserved-space-has-space-ancestor (verifies REQ:dollar-prefix-sigil)

**Scenario:** records in a reserved space satisfy the linkage validator
**Given** a record stored at `/spaces/$gameboard/ext/gameboard/games/{id}` that carries a `related` link to a happening
**When** `WithRelated.ValidateWithKey` derives the `spaceID` from the record's key ancestry
**Then** it resolves `spaceID = $gameboard` and validation succeeds (the record has a valid space ancestor).

## Rehearse Integration

The ACs are testable Go-level behaviors against `coretypes` (the reserved-id
helper, `IsReserved`, `$`-implies-System derivation), the `dto4spaceus`
create-space validation (reject `$` ids), and the `sneat-libs` linkage validator
(`WithRelated.ValidateWithKey` over a reserved-space key). Rehearse stubs are
deferred to plan/implement time, when the `coretypes` helpers and the spaceus
validation branch land; the ACs are written to be directly executable as table
tests at that point.

## Open Questions

Settled in review (2026-06-29), folded into Behavior above:

- **Canonical `SpaceRef` form** → the bare `$<ext>` id; the `system!$invitus`
  type-prefix form is not used for reserved spaces (`$` carries the type). See
  `REQ:dollar-implies-system`.
- **One vs many reserved spaces per extension** → **exactly one** per extension
  (hard 1:1); no `$<ext>-<purpose>` scheme. See `REQ:dollar-prefix-sigil`.
- **Sigil choice / scan** → `$` confirmed: greenfield project, no existing
  `$`-prefixed space ids, no backward-compatibility constraint.

None remaining.

---
*This document follows the https://specscore.md/feature-specification*
