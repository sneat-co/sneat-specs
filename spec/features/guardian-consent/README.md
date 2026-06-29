---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: Guardian Consent

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/guardian-consent?op=explore) | [Edit](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/guardian-consent?op=edit) | [Ask question](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/guardian-consent?op=ask) | [Request change](https://specscore.studio/app/github.com/sneat-co/sneat-specs/spec/features/guardian-consent?op=request-change) |
**Status:** Approved
**Date:** 2026-06-29
**Owner:** alex
**Source Ideas:** coppa-gdpr
**Supersedes:** —
**Grade:** A

## Summary

Capture and store a **verifiable parental-consent record** — the lawful-basis
artifact for processing a minor's data — at the **guardian-link** step of the
invitus accept flow ([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md)).
The record lives in the spaceless system namespace (`/ext/{ext-id}/...`, per
[Decision 0002](../../decisions/0002-reserved-extension-space-ids.md) /
[reserved-extension-space-ids](../reserved-extension-space-ids/README.md)),
captures the jurisdiction + consent-age resolved at that moment (from the
`jurisdiction-resolver` sibling Feature), and is written with **lightweight
verification**: the guardian is a verified adult user with a confirmed contact
channel — SMS-preferred, email-fallback — recorded as `method` + `assuranceLevel`
so a market needing COPPA-grade verifiable parental consent can harden later with
no migration. Until an active consent record exists, the minor's data is gated
(flagged consent-pending, not fully processed). The record is **retained
independent of the data it governs** — it is the proof-of-consent + deletion
ledger and survives child-data erasure.

## Problem

Minors are first-class Sneat users, so processing their personal data needs a
**lawful basis** — verifiable parental consent under COPPA, GDPR Article 8, and
the UK Children's Code (per the [`coppa-gdpr`](../../ideas/coppa-gdpr.md) Idea).
[Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md) already
requires a minor to have **≥1 guardian linked** and determines minor status
multi-signally, so the guardian-link is the natural attach point — but **nothing
records that consent today**. Without a durable, verifiable, auditable consent
record we cannot prove lawful basis to a regulator, cannot gate processing on it,
and cannot hand a parent a transparency/deletion surface that references it. This
Feature creates that record and the gate; it **consumes** jurisdiction resolution
(sibling Feature) and **defers** the re-consent lifecycle, deletion mechanics,
data-minimization, and the parent-facing surface to their own Features.

## Behavior

### Consent record

#### REQ: consent-record

A consent grant MUST persist a **consent record** in the system namespace
(`/ext/{ext-id}/...`), the lawful-basis artifact for the guardian↔child pair. The record MUST reference the
guardian↔child link rather than living on it, and MUST carry at least:
`guardianUserID`, the child reference (`childUserID` or `childContactID`),
`jurisdiction` and `consentAge` (the snapshot resolved at grant time),
`scope` (what processing the consent covers), `method` and `assuranceLevel`
(the verification tier), the verified channel reference, `grantedAt`, and a
`status` of `active`. There MUST be at most **one active record per
(guardian, child, scope)** — a repeat capture for the same tuple is idempotent
(no duplicate active record). The phone/email used for verification is stored on
the **guardian** record/reference, never required of the minor. The
`jurisdiction`/`consentAge` snapshot is captured at `grantedAt` and is
**immutable** thereafter — re-evaluation **supersedes** the record rather than
editing it in place (the `re-consent` Feature's concern).

### Capture at guardian-link

#### REQ: capture-at-guardian-link

Consent capture MUST be triggered at the **guardian-link step of the invitus
accept flow** ([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md))
when the linked player is determined to be a **minor**. When the linked player is
determined to be an **adult**, no consent record is required or created. Capture
MUST be initiated before the minor's data is fully processed (it sets up the gate
in `REQ:processing-gate`).

### Verification

#### REQ: verified-guardian-channel

The guardian MUST be a **verified adult user with a confirmed contact channel**
before an `active` consent record is written. Channel confirmation is a **consumed
capability**, not reimplemented here: where the guardian already proved control of
a channel by **accepting an invite on that channel** (invitus's claim), that proof
MUST be reused without a redundant challenge. Where a channel the invite did not
establish is needed — notably **phone for the SMS tier** — an explicit OTP confirm
is requested via the platform notifications capability. **SMS is preferred; email
is the fallback** where a number genuinely can't be used. The chosen channel and
its assurance are recorded as `method` + `assuranceLevel`. No method asserts
adulthood outright; the bar is a *reasonable effort to confirm the consenter is the
parent* (card-possession alone is out of scope — see Not Doing).

### Data processing gate

#### REQ: processing-gate

Until an `active` consent record exists for a minor, the minor's profile MUST be
flagged **consent-pending** and the minor's data MUST NOT be fully processed or
shared. Once an `active` record exists, processing MUST proceed **only within the
recorded `scope`**.

### Retention

#### REQ: consent-retention

The consent record MUST be **retained independently of the data it governs**: it
MUST survive family-space deletion and the child-data erasure itself (it is the
proof-of-consent + deletion ledger), and MUST be purged only after the applicable
jurisdiction's **retention window** elapses.

## Acceptance Criteria

### AC: consent-record-persisted-in-system-namespace (verifies REQ:consent-record)

**Scenario:** a grant writes a referencing record in the system namespace
**Given** a guardian linked to a minor child with a resolved jurisdiction and consent age
**When** a consent grant is recorded
**Then** a consent record persists in the system namespace (`/ext/{ext-id}/...`) referencing the guardian↔child link and carrying `guardianUserID`, the child reference, `jurisdiction`, `consentAge`, `scope`, `method`, `assuranceLevel`, the verified-channel reference, `grantedAt`, and `status = active`.

### AC: one-active-record-per-tuple (verifies REQ:consent-record)

**Scenario:** repeat capture is idempotent
**Given** an active consent record already exists for a (guardian, child, scope) tuple
**When** consent capture runs again for the same tuple
**Then** no duplicate active record is created.

### AC: capture-triggered-for-minor (verifies REQ:capture-at-guardian-link)

**Scenario:** linking a guardian to a minor initiates capture
**Given** an invitus accept that links a guardian to a player determined to be a minor
**When** the guardian↔child link is established
**Then** consent capture is initiated for that pair before the minor's data is fully processed.

### AC: no-capture-for-adult (verifies REQ:capture-at-guardian-link)

**Scenario:** adults need no parental consent
**Given** an accept that links to a player determined to be an adult
**When** the link is established
**Then** no consent record is required or created.

### AC: reuse-invite-channel-proof (verifies REQ:verified-guardian-channel)

**Scenario:** invite-accept already proved the channel
**Given** a guardian who established control of a channel by accepting an invite on that channel
**When** consent is recorded
**Then** that channel proof is reused and recorded as the verified channel, with no redundant OTP challenge.

### AC: sms-preferred-tier (verifies REQ:verified-guardian-channel)

**Scenario:** phone confirmed via OTP yields the SMS tier
**Given** a guardian whose phone is confirmed via OTP
**When** consent is recorded
**Then** `method` records the SMS channel, `assuranceLevel` reflects the SMS tier, and the phone is stored on the guardian reference only.

### AC: email-fallback-tier (verifies REQ:verified-guardian-channel)

**Scenario:** email is the fallback when SMS can't be used
**Given** SMS cannot be used and the guardian's email is confirmed
**When** consent is recorded
**Then** `method` records email-fallback and `assuranceLevel` reflects the lower tier.

### AC: unverified-channel-blocks-record (verifies REQ:verified-guardian-channel)

**Scenario:** no confirmed channel ⟹ no active record
**Given** no guardian channel can be confirmed
**When** consent capture runs
**Then** no `active` consent record is written.

### AC: gated-until-consent (verifies REQ:processing-gate)

**Scenario:** a minor without consent is not processed
**Given** a minor with no active consent record
**When** their data would be processed or shared
**Then** processing is withheld and the profile is flagged consent-pending.

### AC: processing-within-scope (verifies REQ:processing-gate)

**Scenario:** processing honors the recorded scope
**Given** a minor with an active consent record
**When** their data is processed
**Then** processing proceeds only within the recorded `scope`.

### AC: unresolved-jurisdiction-stays-gated (verifies REQ:processing-gate)

**Scenario:** an unknown jurisdiction is fail-safe (no record, stay gated)
**Given** a minor whose jurisdiction cannot be resolved
**When** consent capture runs
**Then** no `active` consent record is written and the minor stays consent-pending, with the strictest applicable rule set assumed until resolution.

### AC: record-survives-child-data-deletion (verifies REQ:consent-retention)

**Scenario:** the ledger outlives the data it governs
**Given** a child's data is erased
**When** erasure runs
**Then** the consent record is retained as proof/deletion-ledger and is not erased with the governed data.

### AC: purge-after-retention-window (verifies REQ:consent-retention)

**Scenario:** the record is purged once retention lapses
**Given** a consent record whose jurisdiction retention window has elapsed
**When** the retention purge runs
**Then** the consent record is purged.

## Architecture & Dependencies

- **Consent-record store** — a collection in the system namespace (`/ext/{ext-id}/...`,
  [reserved-extension-space-ids](../reserved-extension-space-ids/README.md)); the
  linkage validator's spaceless branch lets the record carry `related` links to the
  guardian↔child link without a space ancestor, while keeping it platform-owned,
  enumerable, and auditable.
- **Consumes `jurisdiction-resolver`** (sibling Feature) — returns the applicable
  rule set + `consentAge`; this Feature records the returned `{jurisdiction,
  consentAge}` as an immutable snapshot on the record. It does **not** resolve
  jurisdiction itself.
- **Consumes invitus accept/claim** as a source of channel proof, and the
  **platform notifications capability** for an explicit OTP confirm when a new
  channel (e.g. phone) is needed.
- **Invoked by** the invitus accept flow at guardian-link (Decision 0003).
- **Read by** (future siblings) `minor-data-protection`, `parent-transparency-surface`;
  **spared by** `child-data-erasure`; **mutated/superseded by** `re-consent`.

**Data flow:** accept → guardian-link(minor) → resolve jurisdiction (sibling) →
ensure verified channel (reuse invite proof, else OTP via notifications) → write
`active` consent record in the system namespace → clear the consent-pending gate.

**Error & failure modes:** if jurisdiction cannot be resolved, the gate is **kept**
and the **strictest applicable** rule set is assumed until resolution (no record is
written on an unknown jurisdiction). If channel confirmation fails, no `active`
record is written and the minor stays gated. Capture is idempotent per
(guardian, child, scope).

## Not Doing

- **Jurisdiction resolution internals** — owned by the `jurisdiction-resolver`
  sibling Feature; consumed here.
- **Re-consent lifecycle** — the periodic sweep, age-out continue-by-default
  handoff, and jurisdiction/scope-change triggers are the `re-consent` sibling
  Feature. This Feature only writes the initial record + its retention rule.
- **Deletion mechanics** — the deletion orchestrator and erase-vs-retain matrix are
  `child-data-erasure`; here we specify only that the consent record is *spared*.
- **Data-minimization & ad-suppression** — `minor-data-protection`.
- **Parent transparency UI** — `parent-transparency-surface`.
- **Hard VPC tiers (card / government-ID / KBA / video)** — the record's
  `method`/`assuranceLevel` fields are shaped to support hardening later without
  migration, but no hard tier is built now; card-possession is explicitly *not* a
  standalone adulthood proxy.
- **Legal sign-off / DPA / ToS drafting** — needs counsel, outside engineering scope.

## Assumption Carryover

- **Carried (must-be-true):** the Decision 0003 ≥1-guardian rule + minor
  determination are the attach point this Feature hooks into.
- **Delegated:** "jurisdiction can be resolved reliably with a safe fallback" now
  belongs to the `jurisdiction-resolver` sibling; this Feature consumes its result
  and keeps the gate when resolution is absent.
- **Open / flagged (should-be-true):** lightweight verification (verified adult +
  SMS-preferred/email-fallback channel) clears the "reasonable effort to confirm the
  consenter is the parent" bar in launch markets — needs per-jurisdiction legal
  review (not legal advice).

## Rehearse Integration

The ACs are testable contract/data behaviors against the consent-record store and
the capture hook: record shape + system-namespace placement, idempotency per tuple,
minor-vs-adult capture branching, channel-proof reuse vs OTP, the processing gate,
and retention/purge. Rehearse stubs are deferred to plan/implement time, when the
consent-record store and the invitus guardian-link hook land; the ACs are written
to be directly executable as table tests at that point.

## Open Questions

- **Namespace owner for the consent collection.** A dedicated compliance namespace
  (e.g. `/ext/coppa-gdpr/...`) versus hosting the collection as a `coppa-gdpr`
  overlay module under invitus's namespace (`/ext/invitus/...`; the system namespace
  may host overlay records from other modules — see
  [reserved-extension-space-ids](../reserved-extension-space-ids/README.md)). Lean: a
  **dedicated compliance namespace**, because the record is platform/legal
  infrastructure that must be independently auditable and outlive invitus records.

---
*This document follows the https://specscore.md/feature-specification*
