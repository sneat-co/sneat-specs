---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: COPPA / GDPR child-privacy compliance

**Status:** Draft
**Date:** 2026-06-29
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might we let minors be first-class Sneat users, collecting only what we need with a verified guardian consenting on their behalf, so we are legally compliant across our launch markets and parents trust us with their kids data?

## Context

Minors are first-class Sneat users (own accounts; we collect name, DOB, gender, team/division membership, game locations, a relationship graph, a personal calendar), triggering COPPA (US, under-13: verifiable parental consent + parental access/deletion + ad restrictions), GDPR Article 8 (EU, under-16, state-configurable), and the UK Children's Code. Decision 0003 (invite-acceptance-graph-edges) already requires a minor to have at least one guardian linked, and determines minor status multi-signally (DOB authoritative, else the team/game division). This Idea builds on those rather than re-deriving them. Direction set in review (2026-06-29): geo-adaptive jurisdiction handling, a minimal legal gate for the MVP, and a success bar of BOTH legal compliance AND a parent-facing trust surface. General orientation only, not legal advice; rollout needs jurisdiction-specific counsel.

## Recommended Direction

A geo-adaptive child-privacy compliance layer that leans on the existing at-least-one-guardian rule as the consent attach point. On guardian-link, write a consent record (who consented, when, method, and the jurisdiction + consent-age resolved at that moment, plus scope). A jurisdiction resolver maps the user to the applicable rule set and consent age (13 US; up to 16 EU per member state; UK). For the MVP, verification is lightweight (the guardian is a verified adult user, plus email confirmation), with the record shaped so it can later harden to payment/ID-grade verifiable parental consent without a data migration. Until consent is recorded, a minor profile is flagged incomplete and their data is not fully processed. Guardians can review and delete the child data across the family graph; minors get data-minimization defaults and are excluded from behavioral advertising. A parent-facing transparency surface (what we hold about your child, who can see it, one-tap delete) serves the trust half of the success bar.

## Alternatives Considered

- **Minors are not users (guardian-proxy only).** Only adults hold accounts; a
  guardian manages the child entirely. *Lost:* abandons the decided minor-as-user
  value and the relationship graph that minors-as-users produce — it dodges
  compliance by deleting the product.

- **Full verifiable parental consent (payment/ID) from day one.** Maximally safe.
  *Lost:* high friction on every minor onboarding, slows the graph-building flow,
  and is more than the minimal legal gate requires for launch. We shape the consent
  record so this can be added later without migration instead.

- **One fixed global threshold (e.g. treat everyone under 18 as needing consent
  everywhere).** Simple, no jurisdiction resolver. *Lost:* over-restrictive in the
  US (COPPA is under-13) and misaligned with EU member-state variation (13–16) — it
  both over- and under-shoots. Geo-adaptive is more correct and lower-friction.

## MVP Scope

Age-gating reuses Decision 0003 minor-determination; a jurisdiction resolver (geo/locale/declared, falling back to the strictest rule set when ambiguous) yields the applicable consent age and rules. A consent record is written at guardian-link with lightweight verification; a minors data is not fully processed until it exists. Guardians get review + delete of the child data; minors get data-minimization defaults and ad-suppression; a minimal parent transparency view ships. Prove end to end: a minor invited into an Under-12 division is gated, a guardian links and consents, the consent record captures the resolved jurisdiction/age, and the guardian can view and delete the child data.

## Not Doing (and Why)

- Hard verifiable parental consent (payment-card or government-ID grade) in v1 — deferred; architecture must support hardening to it later without migration
- Exhaustive per-jurisdiction legal coverage — start with US/EU/UK majors behind an extensible geo-adaptive resolver
- Verifying a minors own real age or identity — we trust the DOB/division signals from Decision 0003
- Legal sign-off, DPA, and ToS/policy drafting — needs counsel, outside engineering scope
- Building the family graph or minor-determination — they exist in Decision 0003; this consumes them

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | The Decision 0003 ≥1-guardian rule + minor-determination are the consent attach point this layer hooks into. | Confirm 0003 is accepted/implemented; trace the guardian-link point where a consent record would be written. |
| Must-be-true | A user's jurisdiction can be resolved reliably enough to pick the rule set (geo/IP, locale, or declared country), with a safe fallback. | Spike jurisdiction resolution; confirm fallback-to-strictest behaviour when the signal is absent or ambiguous. |
| Should-be-true | Lightweight verification (guardian = verified adult user + email confirm) is legally defensible for the MVP in target markets. | Legal review per launch jurisdiction (flagged: not legal advice). |
| Should-be-true | Data-minimization defaults and behavioral-ad suppression for minors are feasible against current data flows across products. | Audit what minor data each product (gameboard, eventus, togethered) collects and where ads/profiling apply. |
| Might-be-true | A parent-facing transparency surface measurably increases trust/conversion. | Defer; revisit with the trust-surface design. |


## SpecScore Integration

- **New Features this would create:** `coppa-gdpr` (the child-privacy compliance
  layer: jurisdiction resolver, consent record, parental access/deletion,
  data-minimization + ad-suppression for minors, parent transparency surface).
- **Existing Features affected:** the invitus accept-flow (capture consent at
  guardian-link); minor-determination (Decision 0003, consumed); the family space
  (scope of parental access/deletion over the child's data).
- **Dependencies:** Decision 0003 (≥1-guardian rule + minor determination); the
  family graph; a new platform-level **jurisdiction resolver** (geo/locale/declared
  → applicable rule set + consent age).

## Open Questions

- **Jurisdiction resolution source.** IP-geo vs declared country vs locale — and
  the fallback when ambiguous (default to the strictest rule set?).
- **Acceptable lightweight-verification method per market.** Verified-adult-user vs
  email-plus vs other — needs legal confirmation per launch jurisdiction.
- **Where the consent record lives.** On the guardian↔child link, a dedicated
  consent collection in the family space, or a reserved-space record.
- **Delete-the-child's-data scope.** Cascade semantics across the family graph and
  the other products (gameboard/eventus/togethered) that hold minor data.
- **Re-consent triggers.** Jurisdiction change, crossing the consent-age threshold
  (minor turns of-age), or a material policy/scope change.
