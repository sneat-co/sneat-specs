---
format: https://specscore.md/plan-specification
status: In Review
---

# Plan: Firestore root audit, prevention and destructive cutover

**Status:** In Review
**Source:** none
**Date:** 2026-10-25
**Owner:** alex
**Supersedes:** —

## Summary

Govern Firestore collection placement in `sneat-eur3-1`, using one reviewed registry for AI-agent guidance, mandatory DALgo path checks, CI, and a daily DataTug Check/Watch. The initial inventory contains 54 roots: 6 approved baseline roots, 43 relocation proposals and 5 candidates for founder root-level review. This is a proposed plan; runtime controls, monitoring, data cutover and deletion are not yet executed.

The founder explicitly says backward compatibility is unnecessary and existing records may be dropped. The later clarification controls execution: **the founder must review and approve the exact deletion list before any deletion**. No approval of this plan, registry, root candidate or source-code change substitutes for that approval.

## Approach

Deliver prevention before cleanup: measure the live tree, review ownership, land an agent rule and one policy registry, cover the runtime write boundary and CI, establish the daily DataTug dogfood check, then switch all consumers and reset approved legacy roots. Reuse DALgo's existing `access.SecureDB` and mandatory database policies rather than change its exported API. Global extension coordination uses `ext/{extension}/{collection}/{id}`; business entities default to `spaces/{space}/ext/{extension}/{collection}/{id}`. Source provenance and every supplied root are in [inventory.md](inventory.md); the reviewable classification is in [root-collections.json](root-collections.json). That JSON is currently a proposal, not a deployed policy.

### End-to-end journey and observable results

1. An agent adds extension storage. The registry and key-factory test show its owning extension and full path before landing.
2. A backend attempts an unapproved root write, directly or inside a transaction/batch. The mandatory policy rejects it before storage mutation; a permissive caller policy cannot widen it.
3. An out-of-band admin write creates a forbidden root. At the next daily run, DataTug enumerates the entire root set, computes the unexpected-root set and emits one actionable alert without a manual refresh.
4. Nobody acts and the violation persists. Evidence remains visible; duplicate notifications follow a documented reminder policy rather than flooding the recipient. A changed unexpected-root set produces a new alert.
5. The root is removed or explicitly approved. DataTug records recovery and the next run passes. Authentication failure, incomplete pagination and missed runs never look like a healthy empty database.
6. The founder reviews a timestamped deletion manifest with direct records and all descendant counts. Only that exact approved scope can be deleted, after all old writers are disabled.
7. Users, bots, jobs, checkout callbacks and DataTug operate on the new paths. Legacy roots stay absent through an active workload and a subsequent daily check.

### Audit verification

A complete read-only, name-only scan at one fixed snapshot enumerated every page and recursively visited missing parent references. Private evidence and the full per-root/per-subcollection count tables are in `.wb/runtime/firestore-root-governance/root-review.md`; concrete paths stay outside Git. Independent server-side COUNT queries matched the three largest collection counts. Independent review round 3 recomputed recursive totals, checked retained versus proposed-deletion classifications, and confirmed every deletion approval flag remains false. Plan and rule review rounds 1–2 resolved sequencing and registry-authority findings; no findings remain. SpecScore lint passes. These receipts validate this audit and proposed plan, not deployed runtime controls.

### Ownership and root candidates

Approved baseline: `_smoke`, `users`, `spaces`, `health_checks`, `ext`, `invites` (founder approved retaining `invites` on 2026-10-25). Root candidates requiring founder decisions: `botPlatforms`, `moneyEvents`, `oauthDeviceAuthorizations`, `oauthDeviceUserCodes`, `referrers`. The default recommendation is to relocate these too unless the founder identifies a shared platform-infrastructure need. `ext` is a root collection; `ext/checkout` is its extension document; `ext/checkout/customers` is the customer collection.

All `checkout*`, `datatug*`, OVDB, AI, Commītius and game roots must leave root level. Proposed targets do not settle user/space ownership: inspect the existing domain contract before choosing global extension versus space extension paths. Commītius explicitly documents personal global ownership; move it beneath its extension without inventing space ownership. Reconcile the Debtus/Anybot `referrers` collision. Keep cross-account payment/global uniqueness coordinators global within their extension. Checkout keeps the founder's requested `ext/checkout` name although Paymentus owns the code. Verify the game's registered extension ID before using the proposed `bidding-tictactoe` segment.

### Reviewed placement refinements — 2026-10-25

`invites` is approved to remain at root. This is a placement decision only; no data deletion or runtime deployment is authorized by it.

`slugs` does not need a root-level exception. The current generic registry mixes two semantic scopes behind opaque namespace strings: public-space names such as `sneatclub:space` must be resolved before the Space ID is known; Bookius booking-type names are explicitly unique only within a known space (`bookius:bookingType:<spaceID>`). Global lookup does not require a root collection. Proposed paths are `ext/{extension}/slugNamespaces/{kind}/claims/{slug}` for global names and `spaces/{space}/ext/{extension}/slugNamespaces/{kind}/claims/{slug}` for space-local names. Keep the domain space/entity in its existing space-owned location; the global record is a lookup/uniqueness claim. Final owner/path templates remain a design review item. Do not infer authorization or scope by splitting opaque namespace strings; pass validated typed scope and owner through the owning caller/key factory.

Claims hold `targetID`, `targetKind`, `claimedAt`, optional `claimedBy`, and release/rename tombstone metadata (`releasedAt`, `successorSlug`). Namespace and normalized slug are in the document path. The existing public lookup supports space registration, slug claim, rename and public resolution; Bookius uses it to create and resolve booking types. At the audit snapshot, the only populated root slug namespaces were `sneatclub:space` (2 claims) and `schoolus:space` (5 claims). This is snapshot evidence, not a claim that every source consumer has production data.

`transfers` is proposed to become `ext/debtus/transfers/{id}` as one authoritative global extension record. A transfer can involve parties associated with different spaces; do not force ownership under one party's space or create independently editable copies. Optional per-space references/projections can support ledgers and discovery, with an explicit consistency/rebuild policy if introduced. Current Debtus `OwnerSpaceID` is optional and anchors linked-task metadata; source comments explicitly preserve the global transfer rather than imply exclusive space ownership. Preserve participant authorization independently of physical placement. This proposal follows the founder's suggested path; it is not deletion approval.

### Registry and enforcement contract

- One versioned canonical registry must enumerate project/database, approved roots, registered extension IDs, allowed collection-path templates, owner, rationale, sensitivity and review authority. During this draft, `root-collections.json` carries classifications and proposals. Promote the approved policy into `sneat-firebase/firebase/root-collections.json`; update the agent rule to point to that promoted registry in the same change; pin its digest/version in backend, CI and DataTug; fail rather than silently use divergent copies.
- No prefix allow-lists (`checkout*`, `datatug*`, `ovdb*`), no arbitrary `ext/*/**` acceptance. Placement approval never grants access to data. Existing auth, memberships, money fences, TEST/LIVE separation and token secrecy still apply.
- Compose a mandatory storage-placement policy with existing caller/tenant policies using `access.SecureDB(..., access.WithDatabasePolicies(...))`. Read required normalized resources through the current generic policy surface. Keep the Sneat registry in Sneat, not hard-coded inside generic DALgo. No exported DALgo API changes are approved by this plan.
- Cover Insert/Set/Update/Delete, multi-record operations, incomplete/generated keys, write sessions, nested descendants and every read-write transaction handle; reject the whole batch before committing any denied item. Match full hierarchical resources structurally. Keep normal access authorization separate. Before the final cutover, exact existing legacy paths may have temporary read and write placement allowances so deployed consumers keep working; ordinary authorization remains mandatory and the legacy set cannot grow. The final new-path deployment removes both allowances. After cutover, reads of disallowed paths are permitted only through explicitly scoped diagnostic/cutover tooling.
- Audit host startup, Cloud Functions, bot drivers, workers, migration tools, test fixtures and direct Firestore SDK/REST clients. A DAL wrapper cannot guard a raw SDK client or console. Route routine writes through the secured factory, enforce a code/import boundary against bypasses, and restrict production service-account use to reviewed entry points. `sneat-go` remains wiring only; policy implementation belongs in a reusable owning library and each module owns its keys.
- The existing Firestore rules deny client writes. Server libraries use IAM and [bypass those rules](https://firebase.google.com/docs/firestore/security/rules-conditions); a rules-only root check cannot prevent server-created roots. Audit the broad read catch-all when introducing `/ext`: membership-looking fields must never make credential/financial/control subtrees client-readable.

### CI contract

Tests must prove approved examples, arbitrary unknown roots, typo roots, deprecated roots, unknown extension IDs and mis-scoped paths. Parameterize the mutation matrix across direct writes, batches, transactions, incomplete keys and descendants beneath forbidden roots. Include all-or-nothing batch behavior, rollback on transaction failure, policy failure, and caller-allow versus mandatory-deny cases. Exercise real host wiring, not only a standalone policy helper. Inventory roots after representative emulator journeys (account/space setup, checkout TEST callback, bot action, DataTug project/watch and OAuth) and compare with the registry. The test starts with an empty database and must catch the first accidental root. Add a source scan as supplementary lint for direct SDKs and unregistered key factories; literal grep alone is not the runtime proof.

### Daily DataTug Check and alert contract

- Use a real DataTug Check/Watch against a metadata-only Firestore schema source, project `sneat-eur3-1`, database `(default)`. Enumerate collection IDs with pagination; compare exact names against the approved registry. Existing DataTug Firestore schema enumeration and Watch interval validation provide building blocks; a deployed Check that can evaluate collection metadata has **not** been verified and may require an adapter/check implementation.
- Continuous Watch every 86400 seconds; run once at activation so existing unexpected roots alert immediately, not baseline silently. Target the next daily slot at 08:00 Europe/Dublin if the scheduler supports timezone calendar schedules; otherwise document the exact UTC anchor and DST behavior of a 24-hour interval. Persist last successful scan time and registry digest.
- Emit deterministic evidence: unexpected roots, legacy roots awaiting approved cleanup, newly appeared/disappeared names, policy digest, project/database and observation time. Never send document contents, customer IDs, tokens or credential fields. The daily health check needs root metadata only; recursive counting is a separate on-demand audit.
- Unknown roots are actionable immediately. Before cutover, track the existing 48 non-baseline roots as visible legacy inventory rather than flood on each run; keep them separate from approved roots and remove each temporary legacy entry after its approved cleanup. No wildcard legacy exception and no permission to add a new root. This is monitor rollout classification, not old/new data compatibility.
- On first violation or changed unexpected set, notify the chosen existing DataTug channel; deduplicate identical state. On recovery emit one recovery event. Unknown channel/recipient is an open question; do not send external messages until specified. Alert on scan/auth/pagination failure and add an independent freshness check for last successful scan older than 30 hours, so a dead scheduler cannot silently monitor itself.
- Validate healthy → forbidden-root detection → alert receipt → unchanged-state deduplication → changed-set alert → cleanup → recovery, plus auth failure, pagination failure and runner downtime. Use an emulator for deliberate violations; prove one authenticated production root scan and an explicitly labeled test delivery through the real channel. Capture delivery receipt and unattended scheduled-run receipt before claiming monitoring live.

### Destructive cutover and deletion contract

Full replacement, **no copying, dual reads, dual writes or compatibility aliases**. Inventory and switch every reader/writer before deleting anything: backend modules, browser subscriptions, bots, Cloud Functions, cron/task workers, webhooks, triggers, indexes, collection-group queries, TTL policies, fixtures, scripts, serialized document references and stored path strings. [Parent deletion does not delete subcollections](https://firebase.google.com/docs/firestore/manage-data/delete-data); recursive deletion must cover existing records beneath missing parents too. Deleting a root changes neither external Stripe objects nor GitHub/Google grants. Prevent replayed callbacks from silently reconstructing invalid entitlements after local payment ledgers reset; test the reset behavior and fail closed until fresh trusted provider facts are reconciled.

For the founder's review produce an immutable manifest with exact project/database, exact root collection names, each concrete descendant collection path, direct existing-document count, direct and recursive subcollection-instance counts, descendant-document count, total document count, missing-parent count, scan readTime, proposed target and consumer-cutover status. Include a summary grouped by subcollection path template plus a private concrete-path appendix. Mark every entry `deletionApproved: false` initially. Never enumerate sensitive record contents to populate this report.

Deletion is a separate final execution gate. Stop old jobs/writers and establish a fence; recount and seal the manifest. The founder approves its digest and exact scope. Any newly observed collection path, changed counts, stale cutover state or changed manifest requires renewed review rather than widening deletion silently. A separate deletion identity can have scoped cleanup access; application identities cannot bypass placement policy. Delete only explicitly approved legacy roots recursively, never `ext`, `users`, `spaces`, `_smoke`, `health_checks` or `invites` by a broad exclusion/prefix sweep. A reviewed candidate is still excluded until its deletion entry is explicitly approved. Restore writer traffic only on the new version; re-enumerate the entire tree and exercise real workflows. If rollback is needed after deletion, code must continue using approved paths; do not roll back to a legacy writer that recreates roots.

## Tasks

### Task 1: Measure and review collection ownership

**Status:** planning

**Verifies:** none — source-less audit plan; acceptance: all live roots classified with complete recursive counts, no partial scan reported as zero.

Owner: audit runner + founder. Authenticate read access, fix one snapshot readTime, enumerate all roots and descendants, include missing parent documents and pagination, and record metadata-only counts and concrete-path appendix privately. Compare the live roots to the supplied list and additional source risks. Founder decides each root candidate and target ownership. Outputs: reviewed registry and deletion-manifest draft. No delete calls.

### Task 2: Land the agent rule and canonical registry

**Status:** planning

**Verifies:** none — acceptance: AI instructions resolve one rule; policy, tests and monitor resolve one reviewed registry digest.

Owners: Backstage rule, Workbench instruction renderer, Sneat Firebase registry. Land `firestore-approved-root-collections` and add its trigger to shared agent instructions; label it Stated until real controls ship. Promote approved path templates to the runtime registry and document root addition authority. Include the founder's separate deletion-list approval gate. A registry change needs explicit review; scanning the live database cannot automatically approve what exists.

### Task 3: Enforce mandatory placement at every backend write boundary

**Status:** planning

**Verifies:** none — acceptance: a new forbidden root cannot be created via any routine backend mutation surface; ordinary permitted work still passes.

Owners: reusable Sneat policy library, sneat-go-host construction/wiring, module key factories. Reuse existing DALgo access composition. Inventory and close raw-SDK bypasses. Cover host startup and all modules, jobs and bots. Keep any pre-cutover exact legacy read and write placement allowances visible and temporary until Task 6; never allow a new root. They are a deployment sequencing control, not backward-compatible data storage. Fail startup on missing/invalid policy. The final new-path deployment removes all legacy allowances before deletion. Any necessary DALgo gap is a separately reviewed additive proposal subject to its repository's human API-approval rule.

### Task 4: Add CI placement and security regression gates

**Status:** planning

**Verifies:** none — acceptance: unknown/legacy roots and transaction/batch/native-SDK bypass tests fail before deployment.

Owners: policy library tests, Sneat Firebase emulator/rules suite and host CI. Implement the mutation matrix and representative clean-database journeys above. Update rules to explicitly protect the new global extension subtrees and preserve all existing access boundaries; remove stale root grants after consumers switch. Test secrets and financial/control data against malformed membership fields. Required CI becomes blocking when the new implementation is green; do not ship a permanently red or advisory-only gate.

### Task 5: Activate the daily DataTug dogfood Check and freshness alarm

**Status:** planning

**Verifies:** none — acceptance: exact-root set evaluation, one real channel test delivery, an unattended scheduled-run receipt, and failure/freshness coverage.

Owners: DataTug schema-source/check implementation and platform operations. Implement any missing metadata check capability through the existing DataTug source/Check/Watch contracts. Register a read-only source and 24-hour Watch anchored as documented. Confirm recipient/channel and test delivery. Keep source credentials in the established secret binding. Monitoring stores evidence beneath its extension/space extension, creating no new root. Prove the whole detection/recovery journey and out-of-band heartbeat failure detection.

### Task 6: Switch every consumer to approved paths and seal deletion manifest

**Status:** planning

**Verifies:** none — acceptance: zero routine old-path callers/writers, green new-path workflows and immutable fully counted manifest presented to founder.

Owners: Paymentus/checkout, DataTug, core modules, OVDB, Commītius, games, bot drivers and clients, with host wiring only in sneat-go. Replace all paths including root queries, parent-null assumptions and serialized key reconstruction; prepare indexes/TTL/rules/triggers before new traffic. Quiesce old workers/tasks and fence writes. No copy/backfill. Exercise fresh setup, auth, project creation, search, Watch and TEST checkout callbacks; explicitly verify stale event replay cannot recreate entitlements. Do not perform live payments. Recount under the fence and request approval of the exact manifest digest. Wait for that approval.

### Task 7: Execute only the founder-approved deletion scope and verify absence

**Status:** planning

**Verifies:** none — acceptance: approved roots recursively removed, every kept root retained, no old root returns under real workload and the next daily scan.

Owner: one named cleanup operator. Validate project/database/manifest digest and cutover fence, abort on drift, then delete only approved concrete legacy trees including descendants under missing parents. Record per-entry results, failures and post-delete enumeration; resume idempotently within that same scope. No wildcard sweep. Validate new-path workflows, no security regression, no old roots and DataTug recovery. Capture deployed revision, policy digest, CI receipt, scheduled scan and delivery receipts. The effort is complete only when these controls are live and cleanup is verified; writing this plan is not that completion.

## Open Questions

- Founder: which of the five remaining root candidates should remain root-level, and what is the rationale/owner for each exception?
- Founder: confirm the final global-versus-space ownership proposals, especially projects, slug scope/owning extension, the proposed global transfer placement, referrers and the game extension ID.
- Operations: which existing DataTug recipient/channel should receive violation, failure and recovery alerts, and who owns its 30-hour freshness alarm?
- Implementers: does the currently deployed DataTug runner admit schema-metadata Checks? Source building blocks exist; deployed support and live runner health require verification.
- Founder final gate: review and approve the exact counted recursive deletion manifest after cutover and writer fencing. No deletion has been approved or performed.

---
*This document follows the https://specscore.md/plan-specification*
