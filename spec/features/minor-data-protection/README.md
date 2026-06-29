---
format: https://specscore.md/feature-specification
status: In Review
---

# Feature: Minor Data Protection

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/minor-data-protection?op=explore) | [Edit](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/minor-data-protection?op=edit) | [Ask question](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/minor-data-protection?op=ask) | [Request change](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/minor-data-protection?op=request-change) |
**Status:** In Review
**Date:** 2026-06-29
**Owner:** alex
**Source Ideas:** coppa-gdpr
**Supersedes:** —

## Summary

A central **minor-data policy guard** — `minorDataPolicy(subject, regime) →
constraints` — that **every** collection / share / advertising / profiling
decision point across products (gameboard, eventus, togethered) consults to
enforce, for a known **minor**, two protections: **data-minimization defaults**
and a binding **no-behavioral-ads / no-profiling** constraint. It is the
**enforcement** half of the compliance track, distinct from
[guardian-consent](../guardian-consent/README.md) (the *authorization* gate): even
within a consented scope, a minor's data gets minimized handling and is never
subject to behavioral advertising. The guard **consumes** the minor determination
([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md)) and the
applicable regime ([jurisdiction-resolver](../jurisdiction-resolver/README.md)) —
it applies protections *given* "this is a minor under regime X", and does not
re-derive either. For the MVP it returns the constraint set with **named defaults**
(coarsen DOB to an age-band, optional fields off-by-default) and the binding ad
constraint; the **exhaustive per-field / per-product minimization matrix** is
deferred to its own data-classification spec.

## Problem

A minor's data is spread across every product (gameboard participation, eventus
RSVPs, the family graph, the calendar — per the [`coppa-gdpr`](../../ideas/coppa-gdpr.md)
Idea). Without a single enforcement point, each product would independently
re-decide what to collect about a minor and whether to profile them — inviting
drift, over-collection, and leakage. COPPA restricts behavioral advertising and
profiling for under-13s; GDPR and the UK Children's Code push data-minimization and
no-profiling for minors generally. And authorization is not enough on its own:
[guardian-consent](../guardian-consent/README.md) establishes that we *may* process
within a scope, but a minor still needs **stricter handling within that scope**
(defense in depth) and a **hard, non-negotiable** exclusion from behavioral ads.
This Feature centralizes both as a policy guard every decision point consults.

## Behavior

### Central policy guard

#### REQ: central-policy-guard

A single guard `minorDataPolicy(subject, regime) → constraints` MUST be the **one
source of truth** for minor-data protections; every collection / share /
advertising / profiling decision point across products MUST consult it for a
subject determined to be a minor and MUST enforce the returned constraints. For an
**adult** subject the guard returns **no minor-constraints** (this Feature imposes
nothing). The guard is **driven by** the minor determination
([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md)) and the
regime ([jurisdiction-resolver](../jurisdiction-resolver/README.md)) supplied to
it — it MUST NOT re-derive minor status or resolve jurisdiction itself. When minor
status or regime cannot be determined, the guard MUST **fail safe** to the
most-protective (minor, strictest) constraint set.

### Data minimization

#### REQ: minimization-defaults

For a minor, the guard MUST return **data-minimization** constraints: optional
(non-essential) fields default **off / uncollected**, and collected fields are
exposed in the **least precise form** sufficient for the purpose. Named MVP
defaults: **DOB is coarsened to an age-band** for non-essential uses; precise
location/venue is not retained beyond operational need. These constraints MUST
apply **even within a consented scope** — consent does not waive minimization. The
exhaustive per-field / per-product matrix is out of scope (deferred — see Not
Doing).

### No behavioral ads / profiling

#### REQ: no-behavioral-ads

The guard MUST return a binding **`noBehavioralAds` / `noProfiling`** constraint for
every minor, and any advertising, profiling, or ads-analytics feature — present or
future — MUST consult the guard and honor it: a minor MUST NOT be subject to
behavioral advertising or profiling-for-ads. This is a **forward contract** — no ad
system exists today; the constraint is the binding guarantee future features check,
not advisory guidance.

### Lifecycle boundary

#### REQ: applies-while-minor

The constraints MUST apply for **as long as the subject is a minor** and MUST lapse
once they **age out** (the `re-consent` age-out lifecycle), after which the
now-adult is no longer minor-constrained by this guard. The guard reflects
**current** minor status (read from the determination); it does not itself run the
age-out transition.

## Acceptance Criteria

### AC: products-consult-guard-for-minor (verifies REQ:central-policy-guard)

**Scenario:** a collection point enforces the minor constraint set
**Given** a subject determined to be a minor under regime `R` and a product about to collect or share their data
**When** the product reaches the collection/share decision point
**Then** it consults `minorDataPolicy` and receives — and enforces — the minor constraint set.

### AC: adult-unconstrained-by-guard (verifies REQ:central-policy-guard)

**Scenario:** adults are not constrained by this Feature
**Given** a subject determined to be an adult
**When** the guard is consulted
**Then** it returns no minor-constraints.

### AC: unknown-status-fails-safe-to-minor (verifies REQ:central-policy-guard)

**Scenario:** indeterminate status is fail-safe
**Given** the guard is consulted for a subject whose minor status or regime cannot be determined
**When** it produces constraints
**Then** it returns the most-protective (minor, strictest) constraint set.

### AC: optional-fields-off-by-default (verifies REQ:minimization-defaults)

**Scenario:** non-essential fields are not collected for minors by default
**Given** a minor and an optional, non-essential data field that is not explicitly needed
**When** the guard's constraints are applied at collection
**Then** the field is left uncollected (off by default).

### AC: dob-coarsened-for-non-essential-use (verifies REQ:minimization-defaults)

**Scenario:** a non-essential consumer sees an age-band, not the DOB
**Given** a minor whose DOB is held and a non-essential use that needs only age
**When** that use reads the field via the guard
**Then** it receives an **age-band**, not the precise DOB.

### AC: minimization-applies-within-consent (verifies REQ:minimization-defaults)

**Scenario:** consent does not waive minimization
**Given** a minor with an active consent record covering a processing scope
**When** their data is processed within that scope
**Then** the minimization constraints still apply.

### AC: minor-excluded-from-behavioral-ads (verifies REQ:no-behavioral-ads)

**Scenario:** a minor is never behaviorally targeted
**Given** a minor and any advertising or profiling decision point
**When** it consults the guard
**Then** it receives `noBehavioralAds`/`noProfiling = true` and serves no behavioral ad and builds no ad profile.

### AC: ad-constraint-binding-on-future-features (verifies REQ:no-behavioral-ads)

**Scenario:** the constraint is a hard contract, not advice
**Given** a newly added advertising / profiling / ads-analytics feature
**When** it processes any subject
**Then** it MUST consult the guard and, for minors, honor `noBehavioralAds` (it cannot opt out).

### AC: constraints-lapse-at-age-out (verifies REQ:applies-while-minor)

**Scenario:** protections end when the minor becomes an adult
**Given** a subject who was a minor and has aged out (re-consent age-out completed)
**When** the guard is consulted
**Then** it returns no minor-constraints.

## Architecture & Dependencies

- **The guard** — `minorDataPolicy(subject, regime) → ConstraintSet`, a
  **stateless** policy function (owns no storage). `ConstraintSet` =
  `{ minimization rules (off-by-default / coarsen), noBehavioralAds, noProfiling }`.
  The named defaults + constraint vocabulary live in config; for the MVP the
  minimization defaults are **regime-uniform (strictest)**, with room for
  per-regime tightening later (see Open Questions).
- **Consumes** the minor determination ([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md)),
  the regime ([jurisdiction-resolver](../jurisdiction-resolver/README.md)), and —
  for the within-consent check — the [guardian-consent](../guardian-consent/README.md)
  scope. It re-derives none of them.
- **Consumed by** every collection/share decision point in gameboard / eventus /
  togethered, and by **any future** advertising / profiling / ads-analytics
  feature (which the forward contract binds).
- **Composes with the spaceless model** ([Decision 0002](../../decisions/0002-reserved-extension-space-ids.md))
  trivially — being stateless, it owns no namespace.

**Data flow:** decision point → `minorDataPolicy(subject, regime)` → `ConstraintSet`
→ the caller enforces (skip an optional field, coarsen DOB to an age-band, suppress
a behavioral ad).

**Error & failure modes:** indeterminate minor-status or regime ⇒ **fail safe** to
the most-protective constraint set (treat as minor under the strictest regime),
consistent with the track's fail-safe posture.

## Not Doing

- **The exhaustive per-field / per-product minimization matrix** — which exact
  field in gameboard/eventus/togethered is minimized how — is deferred to its own
  **data-classification specification**; this Feature sets the principle + named
  defaults + the guard contract.
- **Minor determination** ([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md))
  and **jurisdiction resolution** ([jurisdiction-resolver](../jurisdiction-resolver/README.md))
  — consumed, not built.
- **The consent gate** — [guardian-consent](../guardian-consent/README.md).
- **Building an advertising / profiling / analytics system** — none exists; this
  Feature only sets the binding constraint future systems must honor.
- **Team-roster visibility** (brief `First L.` vs full name) — owned by
  [Decision 0003 §4](../../decisions/0003-invite-acceptance-graph-edges.md); this
  Feature does not redefine it.
- **Adult data-minimization** — good practice but a separate concern; this Feature
  is minor-specific.
- **The age-out transition mechanics** — owned by `re-consent`; the guard only
  reflects current minor status.
- **Legal sign-off on which fields are "essential" per regime** — counsel-owned
  (not legal advice).

## Assumption Carryover

- **Owns (should-be-true from the Idea):** "data-minimization defaults and
  behavioral-ad suppression for minors are feasible against current data flows
  across products." The central-guard + forward-contract design makes this feasible
  without an existing ad system; the cross-product field audit is the validation
  (it produces the deferred minimization matrix).
- **Carried (must-be-true):** minor status (Decision 0003) and regime
  (jurisdiction-resolver) are available inputs — addressed by `REQ:central-policy-guard`
  (consumes both; fails safe when absent).

## Rehearse Integration

The ACs are testable contract behaviors over the guard: minor-vs-adult constraint
sets, fail-safe on indeterminate status, off-by-default + DOB-coarsening
minimization, minimization-within-consent, the binding ad constraint, and age-out
lapse. Rehearse stubs are deferred to plan/implement time, when the guard entry
point and the product consult-points land; the ACs are written to run as table
tests over `minorDataPolicy(subject, regime) → ConstraintSet` then.

## Open Questions

- **Regime-specific minimization vs uniform-strictest (MVP).** The MVP applies
  regime-uniform strictest minimization defaults. Whether (and which) defaults
  should vary per regime (e.g. a US-vs-EU difference in a specific field's
  treatment) is deferred — it ties to the jurisdiction config's
  regime-obligation data and the deferred per-field matrix. Lean: uniform-strictest
  now, per-regime tightening alongside the data-classification spec.

---
*This document follows the https://specscore.md/feature-specification*
