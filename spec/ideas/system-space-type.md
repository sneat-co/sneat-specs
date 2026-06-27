---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: System space type for shared, cross-user records

**Status:** Draft
**Date:** 2026-06-27
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might we let an extension keep shared, cross-user records in one reserved space that any authenticated user can write to (backend-mediated), while the owning extension — not space membership — decides who may mutate each record?

## Context

Surfaced while designing gameboard's game-settings: a game is backed by a Calendarius happening, but happenings are space-scoped (/spaces/{spaceID}/...) while gameboard games are a root/global extension that must stay spaceless to users. spaceus gates module writes by space membership (dal4spaceus module-space worker, which Calendarius's RunHappeningSpaceWorker runs through), and no system/service-identity mechanism exists. We want the current user's backend call to write a happening into one reserved space without that user being a normal member, with the extension deciding who may mutate each record.

## Recommended Direction

Introduce a new space type coretypes.SpaceTypeSystem (alongside Family/Private/Group; add to IsValidSpaceType). Branch the spaceus access check in the dal4spaceus module-space worker on space.Type: for a System space, allow module-record writes by any authenticated user and public reads, instead of requiring a membership row. Per-record mutation authorization is delegated to the owning extension (the complementary generic-record-authorization seed in sneat-libs): spaceus permits the write to the space; the extension decides whether this user may mutate this record. First consumer: gameboard's reserved games space, where the current user's gameboard backend call creates/edits a happening and gameboard enforces its crew {userID, role} authz.

## Alternatives Considered

- **Trusted-extension write bypass (no new space type).** Let an extension backend
  write to a reserved space while asserting "I have done my own authorization,"
  bypassing the membership check ad hoc. *Lost:* it is an escape hatch rather than a
  model — every extension reinvents the bypass, there is no single place to reason
  about who may write a shared space, and the "reserved space" stays implicit. A
  typed space (`SpaceTypeSystem`) makes the policy explicit and centralizes it in
  the spaceus access check.

- **System/service identity owns the space.** Provision a system user that owns the
  reserved space; the extension backend calls Calendarius as that system identity.
  *Lost:* no system-identity mechanism exists in sneat-go today (the eventus adapter
  runs as the real user into a member space), so this is net-new infra; it also
  obscures the real actor (happening `createdBy` becomes the system user) and adds a
  provisioning/credential surface. Writing as the *current* user against a System
  space avoids all of that.

- **Spaceless / global happenings in Calendarius.** Extend Calendarius so happenings
  need no space, eliminating the membership question entirely. *Lost:* large,
  invasive change to a space-scoped product (every Calendarius read/index/worker
  assumes a `spaceID`), and it special-cases one extension's need into core
  Calendarius. Adding a space *type* is the smaller, more general change.

## MVP Scope

Add SpaceTypeSystem + IsValidSpaceType; branch the dal4spaceus module-space worker access check on the System type (write = any authenticated user, read = public); provision one reserved games space of that type. Prove end-to-end: a current user who is NOT a member creates and edits a Calendarius happening in the games space via the gameboard backend, with gameboard enforcing crew authz before proxying. No new identity, no membership rows.

## Not Doing (and Why)

- The per-record role-authorization mechanism itself — delegated to the owning extension; tracked by the generic-record-authorization seed in sneat-libs
- Per-record private reads — System-type spaces are public-read for now; private games are out of scope
- Spaceless/global records in Calendarius — rejected; keep space-scoping and add a space type instead
- Migrating existing spaces or changing existing space types
- Direct browser writes to System spaces — writes stay backend-mediated through the owning extension

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | The `dal4spaceus` module-space worker is the single chokepoint where write access is enforced, so branching on `space.Type` there covers all module writes (incl. Calendarius happenings). | Read `dal4spaceus` module-space worker; trace one Calendarius write (`UpdateHappeningTexts` → `RunHappeningSpaceWorker`) and confirm it is the only gate. |
| Must-be-true | A space can exist and be written to by a user with no membership row, without violating other spaceus invariants (calendar-day indexes, space briefs on the user record, etc.). | Spike: create a `SpaceTypeSystem` space, write a happening as a non-member, verify no membership/brief writes are required and reads work. |
| Should-be-true | Public reads of System-space records are acceptable for the first consumers (games are public). | Confirm with product that System spaces are public-read for now; private records are a later, separate concern. |
| Should-be-true | One reserved space holding many records (all games' happenings) scales on Firestore as long as access is always by record id, never an unscoped "list all in space". | Review query paths; ensure gameboard resolves a game's happening by stored `happeningID`, never by listing the space. |
| Might-be-true | The same `SpaceTypeSystem` serves other future shared-record needs (public events, shared lists) beyond gameboard. | Defer; revisit when a second consumer appears. |


## SpecScore Integration

- **New Features this would create:** a `SpaceTypeSystem` Feature in spaceus
  (the new space type + the access-check branch + the per-record authz delegation
  contract).
- **Existing Features affected:** spaceus space model and access check; Calendarius
  happening writes (consume the new type transparently); gameboard game-settings
  (first consumer — backs a game's happening in the reserved `games` space).
- **Dependencies:** complements the `generic-record-authorization` idea seed in
  `sneat-libs` (the per-record "who may mutate this record by role" half, which the
  owning extension implements).

## Open Questions

- **Where the delegation boundary sits.** Does spaceus call back into the owning
  extension to authorize a write, or does it simply permit the space write and rely
  on the extension having authorized *before* calling (backend-mediated)? The MVP
  assumes the latter (extension authorizes first, then writes), but a future
  callback hook may be wanted.
- **Provisioning of reserved System spaces.** How a System space (e.g. `games`) is
  created and addressed (well-known id vs. lookup), and who may create System
  spaces (likely platform-only, not end users).
- **Naming.** `SpaceTypeSystem` vs `SpaceTypeShared`/`SpaceTypePublic` — `System`
  chosen to connote platform-owned + not-user-facing; revisit if a user-facing
  shared space is later wanted.
