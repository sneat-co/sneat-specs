---
format: https://specscore.md/plan-specification
status: Implemented
---
# Plan: System Space Type

**Status:** Implemented
**Source Feature:** system-space-type
**Date:** 2026-06-27
**Owner:** alexandertrakhimenok
**Supersedes:** —

## Summary

Introduce a platform-owned `SpaceTypeSystem` space type in spaceus whose
module-record writes are open to any authenticated user (membership not
required) with public reads, while per-record mutation authorization is left to
the owning extension. The work spans `coretypes` (the new enum value +
validator), the `dal4spaceus` module-space worker (the write access-check
branch and the public-read path), and the create-space path (platform-only
provisioning).

## Approach

Strictly linear. Task 1 lands the type itself, which every later task depends
on. Task 2 is a de-risking spike that confirms the load-bearing assumption
behind the whole Feature — that the `dal4spaceus` module-space worker is the
single gate every module write passes through — before Task 3 writes the
access-check branch against it. Task 3 implements the open-write behavior; Task
4 the public read; Task 5 the platform-only provisioning guard. Tasks 3–5 each
depend only on the type (Task 1) and, for Task 3, on the spike's confirmation
(Task 2).

## Tasks

### Task 1: Add SpaceTypeSystem to coretypes and IsValidSpaceType

**Verifies:** system-space-type#ac:type-accepted-by-validator
**Depends-On:** —
**Status:** complete

Add a `SpaceTypeSystem` value to the `coretypes.SpaceType` enum (alongside
Family/Private/Group) and make `IsValidSpaceType` accept it. This is the
foundation every other task builds on.

### Task 2: Spike — confirm dal4spaceus is the single write chokepoint

**Verifies:** system-space-type#ac:non-member-authenticated-write-succeeds, system-space-type#ac:spaceus-does-not-gate-record-by-membership
**Depends-On:** 1
**Status:** complete

De-risk the Feature's load-bearing Open Question: trace module writes through
the `dal4spaceus` workers and confirm where write access is enforced.
**Outcome: the single-chokepoint assumption is FALSE.** Membership is enforced
in two distinct places, and entry points differ in which they use:
`dal4spaceus/module_space_worker.go` `RunModuleSpaceWorkerWithUserCtx`
(pre-transaction `slices.Contains(Space.Data.UserIDs, userID)` check) and
`dal4spaceus/space_worker.go` `SpaceWorkerParams.GetRecords` (load-time
membership check). `RunModuleSpaceWorkerTx` / `RunSpaceWorkerTx` do not
pre-check and rely on `GetRecords`. Task 3 is therefore scoped to branch on
`space.Type == System` in **both** gates and to audit every worker entry point.

### Task 3: Branch both dal4spaceus membership gates on System type

**Verifies:** system-space-type#ac:non-member-authenticated-write-succeeds, system-space-type#ac:unauthenticated-write-rejected, system-space-type#ac:spaceus-does-not-gate-record-by-membership
**Depends-On:** 2
**Status:** complete

Per the Task 2 spike, membership is enforced in **two** places, so branch both
on `space.Type == System`: (a) the pre-transaction check in
`RunModuleSpaceWorkerWithUserCtx` (`module_space_worker.go`) and (b) the
load-time check in `SpaceWorkerParams.GetRecords` (`space_worker.go`). For a
System space, allow a module-record write by any authenticated user without a
membership row, reject an unauthenticated write, and apply no per-record /
membership authorization (delegated to the owning extension). Audit the other
worker entry points (`RunModuleSpaceWorkerTx`, `RunSpaceWorkerTx`,
`RunReadonlyModuleSpaceWorker`) so none re-imposes membership for System spaces.

### Task 4: Public read for System-space records

**Verifies:** system-space-type#ac:public-read-succeeds
**Depends-On:** 1
**Status:** complete

Allow reads of module records in a System space to succeed for any caller
without authentication.

### Task 5: Reject end-user creation of System spaces

**Verifies:** system-space-type#ac:user-cannot-create-system-space
**Depends-On:** 1
**Status:** complete

In the public create-space path, reject a request with `Type=System`; System
spaces are provisioned by the platform only, never minted by end users.

## Open Questions

- **Single-chokepoint assumption resolved (negative).** The Task 2 spike found
  membership is enforced in two gates (`RunModuleSpaceWorkerWithUserCtx` and
  `SpaceWorkerParams.GetRecords`); Task 3 now covers both. A follow-up could
  consolidate the two into one gate, but that refactor is out of this plan's
  scope.
- **Cross-repo release ordering.** Task 1 lands in `sneat-go-core` (`coretypes`)
  and must be released as a new tag before `sneat-core-modules` (Tasks 3–5) can
  bump its dependency and branch on `SpaceTypeSystem`. Implementation is
  therefore sequential and per-repo, not a single batch.
- **Delegation boundary** (callback vs. authorize-before-write) and **reserved
  System-space addressing/provisioning** remain open from the Feature; both are
  out of this plan's task scope and resolved when the first consumer
  (gameboard's reserved `games` space) is implemented.

---
*This document follows the https://specscore.md/plan-specification*
