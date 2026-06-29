---
format: https://specscore.md/idea-specification
status: Approved
---

# Idea: COPPA / GDPR child-privacy compliance

**Status:** Approved
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

A geo-adaptive child-privacy compliance layer that leans on the existing at-least-one-guardian rule as the consent attach point. On guardian-link, write a consent record (who consented, when, method, and the jurisdiction + consent-age resolved at that moment, plus scope). The consent record is **platform/legal infrastructure**, so its **system-of-record is a reserved `$`-space** ([Decision 0002] — enumerable, auditable, lifecycle-independent of any family space), referencing the guardian↔child link rather than living on it; the family space gets a **read-through projection** for the parent transparency view (the data-delegation pattern), since parents care about the view, not where it lives. Crucially the consent record is **retained for legal purposes independent of the data it governs** — it survives family-space deletion and even the child-data deletion itself, serving as the **proof-of-consent + deletion ledger** for a regulator, then is purged only after its jurisdiction's retention window. A jurisdiction resolver maps the user to the applicable rule set and consent age (13 US; up to 16 EU per member state; UK), resolved from **declared country** (authoritative) or, absent that, **deduced from context** — most strongly the **game/event venue** (a game in Ireland ⟹ Irish jurisdiction), plus team/division region and locale; **never IP-geo** (a VPN or a parent registering while travelling must not flip the rule set). What ultimately governs is the child's residence, for which the venue is a strong proxy but not identical; so when signals conflict or are absent the resolver **falls back to the strictest applicable rule set**. For the MVP, verification is lightweight: the guardian is a **verified adult user** plus a confirmed contact channel — **SMS by default/preferred, email as fallback** where a number genuinely can't be used. SMS is favoured because possession-of-SIM is a stronger assurance signal than email **and** the guardian's phone is the strongest cross-channel identity anchor we have (the Telegram bot is already phone-based), so it bridges Telegram ↔ web identity and feeds space-merge ([Decision 0004], duplicate-account reconciliation). The phone sits on the **guardian** record and is **never required of the minor** — its child-protection/consent purpose is the GDPR-data-minimization justification for asking. The consent record carries first-class `method` and `assuranceLevel` fields so a market that demands COPPA-grade **verifiable parental consent (VPC)** can harden per-jurisdiction later with no data migration, and the strictest-jurisdiction fallback (jurisdiction resolution above) can *require* the harder tier where needed. The harder tier is a **per-jurisdiction, counsel-selected method** (government-ID, knowledge-based, video, or a credit-card-*transaction*-with-statement-notice) — **not** a fixed one, because **no method proves adulthood outright**: the legal standard is a *reasonable effort to confirm the consenter is the **parent***, not certainty. In particular **card possession alone is insufficient** — minors hold debit/prepaid cards (Revolut, GoHenry) — so card, if ever used, must be the credit-transaction-with-notice variant, never a standalone "they paid ⟹ adult" proxy. Until consent is recorded, a minor profile is flagged incomplete and their data is not fully processed. Guardians can review and delete the child data via a **platform deletion orchestrator** keyed on the child userID that fans out across gameboard/eventus/the family graph/the calendar and runs **soft-delete → grace window → hard-purge**. Deletion is **global to the child user** (any guardian may trigger it; under shared custody co-guardians are notified and may confirm or dispute — direction only, specifics deferred). For two-ended graph edges it **severs the child's side and tombstones** rather than rewriting bystanders' records: a contact another family authored themselves (their kid's teammate) is *their* data — it is **delinked from the deleted user and stripped of child-sourced synced PII, but the stub they wrote is kept**. The governing line is *contributed-by-the-child/guardian ⟹ erased; authored-by-others ⟹ kept-but-delinked; de-identified aggregates ⟹ untouched; the `$`-space consent/deletion ledger ⟹ retained per retention window* — load-bearing and example-driven enough to warrant its **own child-data-erasure specification**. Minors get data-minimization defaults and are excluded from behavioral advertising. Consent is **not perpetual**: a re-evaluation sweep plus a scope-change hook enforce three re-consent triggers against the snapshot stored on the record — (1) **jurisdiction tightened** ⟹ re-consent (loosened ⟹ silent record update); (2) **scope expanded** ⟹ re-consent for the delta only; (3) **age-out** (the minor reaches the jurisdiction's consent age, per Decision 0003's turns-of-age lifecycle) ⟹ the guardian-consent requirement **ends** and control transfers to the now-adult user. Age-out asks the now-adult for **explicit re-affirmation**, but if not actioned within a configurable **N days** it lapses to **continue-by-default**: processing continues under the prior lawful basis, with the ledger recording an honest **non-objection** (*"notified D, no action by D+N, continued pending affirmation"*) — never a fabricated affirmative consent. Default-continue **degrades to pause/restrict** where a jurisdiction requires affirmative adult consent or for higher-risk processing (e.g. behavioral ads), where silence ≠ yes. A parent-facing transparency surface (what we hold about your child, who can see it, one-tap delete) serves the trust half of the success bar.

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

Age-gating reuses Decision 0003 minor-determination; a jurisdiction resolver (declared country, else deduced from context — game venue, team/division region, locale; never IP-geo; strictest applicable rule set when signals conflict or are absent) yields the applicable consent age and rules. A consent record (with `method` + `assuranceLevel`) is written at guardian-link with lightweight verification — verified adult user + confirmed channel, SMS-preferred / email-fallback, phone on the guardian only; a minors data is not fully processed until it exists. Guardians get review + delete of the child data; minors get data-minimization defaults and ad-suppression; a minimal parent transparency view ships. Prove end to end: a minor invited into an Under-12 division is gated, a guardian links and consents, the consent record captures the resolved jurisdiction/age, and the guardian can view and delete the child data.

## Not Doing (and Why)

- Hard verifiable parental consent (payment-card or government-ID grade) in v1 — deferred; architecture must support hardening to it later without migration
- Exhaustive per-jurisdiction legal coverage — start with US/EU/UK majors behind an extensible geo-adaptive resolver
- Verifying a minors own real age or identity — we trust the DOB/division signals from Decision 0003
- Legal sign-off, DPA, and ToS/policy drafting — needs counsel, outside engineering scope
- Building the family graph or minor-determination — they exist in Decision 0003; this consumes them
- The exhaustive **erase-vs-retain matrix** and the **shared-custody dispute flow** — directional here (sever-and-tombstone, contributed-vs-authored, global-with-co-guardian-notify); their detailed semantics + worked examples are deferred to a dedicated child-data-erasure specification

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | The Decision 0003 ≥1-guardian rule + minor-determination are the consent attach point this layer hooks into. | Confirm 0003 is accepted/implemented; trace the guardian-link point where a consent record would be written. |
| Must-be-true | A user's jurisdiction can be resolved reliably enough to pick the rule set (declared country, else deduced from context — game venue, team/division region, locale; not IP-geo), with a safe fallback. | Spike jurisdiction resolution; confirm venue-as-proxy quality and fallback-to-strictest behaviour when signals conflict or are absent. |
| Should-be-true | Lightweight verification (guardian = verified adult user + confirmed channel, SMS-preferred / email-fallback) clears the "reasonable effort to confirm the consenter is the parent" bar for the MVP in target markets (no method proves adulthood outright; card-possession alone is insufficient). | Legal review per launch jurisdiction (flagged: not legal advice); confirm which counsel-selected method counts as "enough" per market. |
| Should-be-true | A verified guardian phone is an effective cross-channel identity anchor (Telegram ↔ web) for space-merge, and SMS deliverability/cost is acceptable in launch markets with email as a real fallback. | Spike SMS delivery + cost per market; trace phone → space-merge ([Decision 0004]) join key. |
| Should-be-true | Data-minimization defaults and behavioral-ad suppression for minors are feasible against current data flows across products. | Audit what minor data each product (gameboard, eventus, togethered) collects and where ads/profiling apply. |
| Might-be-true | A parent-facing transparency surface measurably increases trust/conversion. | Defer; revisit with the trust-surface design. |


## SpecScore Integration

- **New Features this would create:** `coppa-gdpr` (the child-privacy compliance
  layer: jurisdiction resolver, consent record, parental access/deletion,
  data-minimization + ad-suppression for minors, parent transparency surface);
  a **platform deletion orchestrator** (child-userID-keyed, cross-product
  soft→grace→hard fan-out); and a separate **child-data-erasure specification**
  (the erase-vs-retain matrix + shared-custody dispute flow, with worked examples).
- **Existing Features affected:** the invitus accept-flow (capture consent at
  guardian-link); minor-determination (Decision 0003, consumed); the family space
  (scope of parental access/deletion over the child's data).
- **Dependencies:** Decision 0003 (≥1-guardian rule + minor determination); the
  family graph; a new platform-level **jurisdiction resolver** (geo/locale/declared
  → applicable rule set + consent age).

## Open Questions

None at this time.
