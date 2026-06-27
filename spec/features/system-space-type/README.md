---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: System Space Type

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/system-space-type?op=explore) | [Edit](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/system-space-type?op=edit) | [Ask question](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/system-space-type?op=ask) | [Request change](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/system-space-type?op=request-change) |
**Status:** Approved
**Date:** 2026-06-27
**Owner:** alex
**Source Ideas:** system-space-type
**Supersedes:** —
**Grade:** B

## Summary

A SpaceTypeSystem space whose module-record writes are open to any authenticated user (public reads), branched in the spaceus access check, with per-record mutation authorization delegated to the owning extension.

## Problem

spaceus gates every module-record write by **space membership** — the
`dal4spaceus` module-space worker, which Calendarius's `RunHappeningSpaceWorker`
(and every other module's writes) runs through. That makes it impossible for an
extension to keep **shared, cross-user records** — e.g. a gameboard game's
backing Calendarius happening — in one reserved space that any authenticated
user can write to without being made a member. The available workarounds are
both bad: a **system/service identity** owning the space does not exist in the
platform today (every happening is created as the real, member user), and making
every user a **member of a shared space** would leak every record to every member.
We need a space whose write access is open to authenticated users, with per-record
authorization owned by the extension rather than by membership.

## Behavior

### Space type

#### REQ: system-space-type

The platform MUST define a **System** space type as a value of
`coretypes.SpaceType` (e.g. `SpaceTypeSystem`) that `IsValidSpaceType` accepts. A
System space is **platform-owned** and **not tied to per-user membership**; it is
the home for an extension's shared, cross-user records.

### Access control

#### REQ: open-write-for-authenticated

In a space of type System, the spaceus access check (the `dal4spaceus`
module-space worker, branched on `space.Type`) MUST allow module-record writes by
**any authenticated user**, without requiring a space-membership row.
Unauthenticated write requests MUST be rejected.

#### REQ: public-read

In a space of type System, module-record reads MUST succeed for any caller
**without authentication** (public read).

### Authorization delegation

#### REQ: per-record-authz-delegated

For System spaces, spaceus MUST NOT enforce **per-record** mutation
authorization. spaceus permits the write to the space; deciding whether the
caller may mutate a **specific record** is the responsibility of the **owning
extension**, which MUST authorize the caller before issuing the write. (spaceus
supplies the open-write space; the extension supplies record-level authorization —
see the `generic-record-authorization` idea in `sneat-libs`.)

### Provisioning

#### REQ: platform-only-provisioning

End users MUST NOT be able to create spaces of type System through the public
create-space path; System spaces are **provisioned by the platform only**. A
create-space request with `Type=System` originating from an end user MUST be
rejected.

## Acceptance Criteria

### AC: type-accepted-by-validator (verifies REQ:system-space-type)

**Scenario:** the System space type is valid
**Given** the spaceus space-type validator `IsValidSpaceType`
**When** it is called with the System space type value
**Then** it returns true, and a space record may carry that type.

### AC: non-member-authenticated-write-succeeds (verifies REQ:open-write-for-authenticated)

**Scenario:** a non-member authenticated user writes into a System space
**Given** a space of type System and an authenticated user with no membership row in that space
**When** the user's backend call writes a module record (for example a Calendarius happening) into the space
**Then** the write succeeds because the access check does not require membership for System spaces.

### AC: unauthenticated-write-rejected (verifies REQ:open-write-for-authenticated)

**Scenario:** an anonymous write is refused
**Given** a space of type System
**When** an unauthenticated request attempts to write a module record into it
**Then** the write is rejected.

### AC: public-read-succeeds (verifies REQ:public-read)

**Scenario:** anyone can read a System-space record
**Given** a space of type System containing a module record
**When** an unauthenticated caller reads that record
**Then** the read succeeds.

### AC: spaceus-does-not-gate-record-by-membership (verifies REQ:per-record-authz-delegated)

**Scenario:** spaceus delegates per-record authorization to the extension
**Given** a space of type System and an authenticated caller whom the owning extension has authorized for a record
**When** the extension backend writes that record
**Then** spaceus permits the write without applying a membership or per-record authorization check of its own.

### AC: user-cannot-create-system-space (verifies REQ:platform-only-provisioning)

**Scenario:** end users cannot mint System spaces
**Given** the public create-space path
**When** an end user requests creation of a space with `Type=System`
**Then** the request is rejected.

## Rehearse Integration

All six ACs are testable Go-level behaviors against the spaceus module
(`IsValidSpaceType`, the `dal4spaceus` access check, and the create-space path).
Rehearse stubs are deferred to plan/implement time, when the spaceus changes land
and the test seams exist; the ACs are written to be directly executable as table
tests at that point.

## Open Questions

- **Delegation boundary.** Does spaceus call back into the owning extension to
  authorize a write, or does it simply permit the space write and rely on the
  extension having authorized *before* calling? This Feature assumes the latter
  (extension authorizes first, then writes); a future callback hook may be wanted.
- **Reserved-space addressing & provisioning.** How a System space (e.g. the
  `games` space) is created and addressed (well-known id vs. lookup), and which
  platform path is allowed to create System spaces.
- **Naming.** `SpaceTypeSystem` vs `SpaceTypeShared`/`SpaceTypePublic`.
- **Single write chokepoint.** `open-write-for-authenticated` assumes the
  `dal4spaceus` module-space worker is the *only* gate every module write passes
  through. This is load-bearing and unverified — a plan-time spike must confirm it
  (trace a Calendarius write through `RunHappeningSpaceWorker`) before relying on a
  single branch.

---
*This document follows the https://specscore.md/feature-specification*
