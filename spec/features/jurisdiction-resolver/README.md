---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: Jurisdiction Resolver

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/jurisdiction-resolver?op=explore) | [Edit](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/jurisdiction-resolver?op=edit) | [Ask question](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/jurisdiction-resolver?op=ask) | [Request change](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/jurisdiction-resolver?op=request-change) |
**Status:** Approved
**Date:** 2026-06-29
**Owner:** alex
**Source Ideas:** coppa-gdpr
**Supersedes:** —
**Grade:** A

## Summary

A **stateless, near-pure platform resolver** that maps a subject (a minor in
context) to the applicable **child-privacy rule set + consent age**. It reads an
ordered set of **signals** supplied by the caller — **declared country**
(authoritative; the child's/family's stated country), else **deduced**: the
**game/event venue** (strongest deduction), then **team/division region**, then
**locale** — and **never IP-geo**. A resolved country maps, via a **static,
versioned, country-level config** (US ⇒ 13, UK ⇒ 13, each EU member state per its
GDPR Article 8 choice of 13–16), to a regime + consent age. When signals **conflict
or are absent**, the resolver returns the **strictest applicable** rule set (the
most protective — highest consent age). The result is a snapshot the caller (e.g.
[guardian-consent](../guardian-consent/README.md)) freezes onto its own record;
the resolver persists nothing and does **not** itself decide whether a particular
child needs consent (the age comparison is the caller's).

## Problem

Child-privacy compliance turns on **which** regime and consent age apply, and that
varies: COPPA is under-13 (US), GDPR Article 8 is under 13–16 **per EU member
state**, and the UK Children's Code applies in the UK. A single global threshold
both over-shoots (treating US 14-year-olds as needing consent) and under-shoots
(missing an EU state's 16 line) — see the `coppa-gdpr` Idea's rejected
"one fixed global threshold" alternative. Without a shared resolver, every consumer
([guardian-consent](../guardian-consent/README.md), `re-consent`,
`minor-data-protection`) would reinvent jurisdiction logic, risking drift and
unsafe defaults. This Feature centralizes it: ordered signals ⇒ `{regime,
consentAge}`, with a **safe strictest fallback** and an explicit **no-IP-geo** rule
(a VPN or a parent travelling must not flip the regime).

## Behavior

### Signal precedence

#### REQ: signal-precedence

Resolution MUST follow **declared-over-deduced** authority. **(1)** A present
**declared country** (the child's/family's stated country) is **authoritative** and
wins **outright** — it is the resolved country (source `declared`) **even when a
deduced signal would be stricter**. **(2)** With **no** declared country, the
resolver uses the **deduced** signals — **game/event venue** (primary), then
**team/division region**, then **locale**. When the available deduced signals
**agree** on a country, that country is resolved, with the highest-ranked present
signal as the `source` (venue > region > locale). When the deduced signals
**diverge** (resolve to different countries), resolution hands off to
`REQ:strictest-fallback` — because deduced signals are proxies for the child's
residence, divergence is resolved **toward protection**, not by simply trusting the
highest-ranked one. **IP-geo MUST NOT be a signal** and MUST NOT influence the
result.

### Rule-set lookup

#### REQ: rule-set-lookup

A resolved country MUST map to a `{regime, consentAge}` via a **static, versioned,
country-level config** (in-repo; no runtime writes): **US ⇒ COPPA, 13**; **UK ⇒ UK
Children's Code, 13**; **each EU member state ⇒ GDPR Article 8** at that state's
configured age (13–16). A country **absent from the config** MUST be treated as
**ambiguous**, triggering the strictest fallback (`REQ:strictest-fallback`) rather
than a guessed default.

### Strictest fallback

#### REQ: strictest-fallback

When no declared country is available **and** the deduced signals resolve to
**different** countries (a conflict), or when no signal resolves a configured
country at all, the resolver MUST return the **strictest applicable** rule set: the
most protective among the candidate jurisdictions, operationally the **highest
consent age** (tie-break beyond highest consent age is deferred — see Open
Questions). With **no**
candidate jurisdictions, it MUST fall back to a defined **global-strictest default**
(the highest consent age in the config). The result MUST flag that the
strictest-fallback was applied.

### Resolution output

#### REQ: resolution-output

The resolver MUST return a result carrying: the resolved **country** (or
`ambiguous`), the **regime**, the **consentAge**, the **winning signal source**
(declared / venue / region / locale / fallback), and a **fallback-applied** flag.
The resolver MUST be **stateless** — it persists nothing and performs no writes;
the caller snapshots the result. Deciding whether a given child needs consent
(child age < `consentAge`) is the **caller's** responsibility, not the resolver's.
(A consumer that speaks of a "jurisdiction" — e.g. guardian-consent's snapshot —
maps it to this result's `country` + `regime`; the resolver never "fails to
resolve" — absence/ambiguity surfaces as the `fallback-applied`/`ambiguous` result,
on which the consumer gates.)

## Acceptance Criteria

### AC: declared-country-wins (verifies REQ:signal-precedence)

**Scenario:** the authoritative signal beats a deduced one
**Given** a declared country `US` and a game/event venue in `IE` (Ireland)
**When** the resolver runs
**Then** it resolves country `US` (COPPA, 13) with source `declared`, ignoring the venue **even though `IE` (16) would be stricter**.

### AC: venue-deduced-when-no-declared (verifies REQ:signal-precedence)

**Scenario:** venue is the strongest deduction when nothing is declared
**Given** no declared country and a game/event venue in `IE`
**When** the resolver runs
**Then** it resolves country `IE` with source `venue`.

### AC: lower-signal-only-when-higher-absent (verifies REQ:signal-precedence)

**Scenario:** locale is consulted only as a last deduced resort
**Given** no declared country, no venue, no team/division region, and a locale implying `GB`
**When** the resolver runs
**Then** it resolves country `GB` with source `locale`.

### AC: agreeing-deduced-signals-resolve-without-fallback (verifies REQ:signal-precedence)

**Scenario:** deduced signals that agree resolve directly, no fallback
**Given** no declared country, a game/event venue in `IE` and a team/division region also in `IE`
**When** the resolver runs
**Then** it resolves country `IE` with source `venue` (the highest-ranked present deduced signal) and the `fallback-applied` flag is **not** set.

### AC: ip-geo-never-resolves (verifies REQ:signal-precedence)

**Scenario:** IP-geo cannot rescue or alter resolution
**Given** no declared/venue/region/locale signal but an IP-geo hint pointing to `DE`
**When** the resolver runs
**Then** no country is resolved from the IP-geo hint and the strictest fallback applies (IP-geo never affects the outcome).

### AC: us-maps-to-coppa-13 (verifies REQ:rule-set-lookup)

**Scenario:** the US maps to COPPA at 13
**Given** a resolved country `US`
**When** the config is looked up
**Then** the result is `regime = COPPA`, `consentAge = 13`.

### AC: eu-member-state-uses-its-art8-age (verifies REQ:rule-set-lookup)

**Scenario:** an EU state's configured Art.8 age is used
**Given** a resolved country that is an EU member state configured at Article 8 age `16`
**When** the config is looked up
**Then** the result is `regime = GDPR-Art8`, `consentAge = 16`.

### AC: unconfigured-country-is-ambiguous (verifies REQ:rule-set-lookup)

**Scenario:** an unknown country does not get a guessed default
**Given** a resolved country absent from the static config
**When** the config is looked up
**Then** it is treated as `ambiguous` and the strictest fallback applies.

### AC: divergent-deduced-signals-pick-strictest (verifies REQ:strictest-fallback)

**Scenario:** conflicting deductions resolve to the most protective
**Given** no declared country, a venue in a country with consent age `13`, and a team/division region in a country with consent age `16`
**When** the resolver runs
**Then** the result has `consentAge = 16` (the stricter) and the `fallback-applied` flag is set.

### AC: no-signals-global-strictest-default (verifies REQ:strictest-fallback)

**Scenario:** total absence of signal is fail-safe
**Given** no resolvable signals at all
**When** the resolver runs
**Then** the global-strictest default rule set (highest configured consent age) is returned with the `fallback-applied` flag set.

### AC: output-carries-source-and-fallback-flag (verifies REQ:resolution-output)

**Scenario:** the result is fully attributable
**Given** any resolution
**When** it returns
**Then** the result includes the country (or `ambiguous`), regime, `consentAge`, the winning signal source, and whether the strictest-fallback was applied.

### AC: resolver-is-stateless (verifies REQ:resolution-output)

**Scenario:** resolution persists nothing
**Given** a resolution request
**When** the resolver runs
**Then** it performs no writes and persists nothing; the result is returned to the caller to snapshot.

## Architecture & Dependencies

- **Static jurisdiction config** — a versioned, in-repo, **country-level** table:
  `countryCode → {regime, consentAge, protectiveness-rank}`,
  seeded with US (COPPA/13), UK (Children's Code/13), and each **EU member state**
  at its GDPR Article 8 age (13–16). Extensible by PR (devs/counsel own the
  values); the **global-strictest default** is the entry with the highest consent
  age. No runtime writes.
- **Inputs are caller-supplied** — the resolver is pure over `(signals, config)`.
  Callers assemble the signal bundle from their own context: declared country
  (guardian/family profile), venue country (the game/event/happening), team/division
  region (gameboard team/division), locale. The resolver does **not** fetch signals
  and does **not** include IP-geo.
- **Consumed by** [guardian-consent](../guardian-consent/README.md) (snapshots
  `{jurisdiction, consentAge}` onto the consent record), `re-consent` (re-evaluates
  on a jurisdiction-tightened trigger), and `minor-data-protection`.
- **Stateless / no persistence** — owns no collection; this is why it composes
  cleanly with the spaceless storage model ([Decision 0002](../../decisions/0002-reserved-extension-space-ids.md))
  without itself needing a namespace.

**Data flow:** caller assembles signals → resolver takes the **declared** country
if present, else the **agreeing deduced** country (venue > region > locale decides
the `source`) → maps it via the static config → if the country is unconfigured, the
deduced signals **diverge**, or no signal resolves, applies the **strictest
fallback** → returns the `{country|ambiguous, regime, consentAge, source,
fallback-applied}` snapshot.

**Error & failure modes:** an unconfigured country, divergent deduced signals, or
no signal at all all converge on the **strictest applicable** result (never an
optimistic guess) — fail-safe toward the most protective regime. A malformed signal
is ignored as absent.

## Not Doing

- **Sub-national law** — US state-level statutes and other sub-country regimes are
  deferred; the config is country-level for the MVP.
- **Deciding whether a child needs consent** — the age comparison (`childAge <
  consentAge`) belongs to the caller (guardian-consent / re-consent), not here.
- **Fetching or owning the signals** — callers supply the context bundle; the
  resolver is pure over it.
- **IP-geo resolution** — excluded by design, not merely unused.
- **The consent record, verification, gating** — [guardian-consent](../guardian-consent/README.md).
- **Re-evaluation triggers / the periodic sweep** — `re-consent`.
- **Legal sign-off on the actual age/regime values** — counsel owns the config
  contents; this Feature owns the resolution mechanism (not legal advice).
- **Strictest tie-break beyond highest-consent-age** — when candidates share the
  highest consent age, the protectiveness-rank tie-break is deferred (see Open
  Questions); the MVP rule is highest-consent-age only.

## Assumption Carryover

- **Owns (must-be-true from the Idea):** "a user's jurisdiction can be resolved
  reliably enough to pick the rule set, with a safe fallback." Addressed by
  `REQ:signal-precedence` + `REQ:strictest-fallback`: declared-authoritative,
  venue-as-strongest-deduction, never IP-geo, strictest-applicable when ambiguous.
- **Flagged (should-be-true):** venue is a strong **proxy** for the child's
  residence but not identical (an away game abroad); precedence (declared over
  venue) plus the strictest fallback keep this safe — validation is the spike noted
  in the Idea.

## Rehearse Integration

The ACs are pure-function table tests over `(signals, config) → result`: precedence
ordering, no-IP-geo, per-country lookup (US/EU-state/UK), unconfigured ⇒ ambiguous,
divergent-deduced ⇒ strictest, no-signal ⇒ global-strictest default, output shape,
and statelessness. Rehearse stubs are deferred to plan/implement time, when the
config table and resolver entry point land; the ACs are written to run directly as
table tests then.

## Open Questions

- **Strictest tie-break when consent ages are equal.** If two candidate regimes
  share the highest consent age but differ in obligations (e.g. ad-restriction vs
  access-request scope), which is "stricter"? Lean: rank by `consentAge` first, then
  by a **configured regime-protectiveness rank** — pending confirmation of that
  rank's ordering source (counsel-owned, alongside the age values).

---
*This document follows the https://specscore.md/feature-specification*
