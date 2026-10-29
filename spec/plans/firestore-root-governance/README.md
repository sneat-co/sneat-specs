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

Govern Firestore collection placement in `sneat-eur3-1`, using one reviewed registry for AI-agent guidance, mandatory DALgo path checks, CI, and a daily DataTug Check and operator alert runner. The immutable initial 2026-10-25 inventory contains 54 roots: 6 approved baseline roots, 43 relocation proposals and 5 candidates for founder root-level review. Those historical classifications do not describe the later approved root set. The founder authorized prevention and monitoring implementation on 2026-10-25. Audit, agent instructions and the current server policy are verified at the checkpoint below. Daily scanning, the independent freshness observer, current authenticated bot enrollment/provider-acknowledged delivery, shared Telegram client deployment and privacy-rule activation have acceptance receipts. Fleet placement/bypass discovery, CI coverage and final execution-fence checks remain in progress. Universal writer coverage and deletion are not claimed.

The founder explicitly says backward compatibility is unnecessary and existing records may be dropped. The later clarification controls execution: **the founder must review and approve the exact deletion list before any deletion**. No approval of this plan, registry, root candidate or source-code change substitutes for that approval.

## Approach

Deliver prevention before cleanup: measure the live tree, review ownership, land an agent rule and one policy registry, cover the runtime write boundary and CI, establish the daily DataTug dogfood check, then switch all consumers and reset approved legacy roots. Reuse DALgo's existing `access.SecureDB` and mandatory database policies rather than change its exported API. Global extension coordination uses `ext/{extension}/{collection}/{id}`; business entities default to `spaces/{space}/ext/{extension}/{collection}/{id}`. Source provenance and every supplied root are in [inventory.md](inventory.md); the reviewable classification is in [root-collections.json](root-collections.json). That JSON remains the audit proposal; the deployed policy is the versioned public Core registry described below.

### End-to-end journey and observable results

1. An agent adds extension storage. The registry and key-factory test show its owning extension and full path before landing.
2. A backend attempts an unapproved root write, directly or inside a transaction/batch. The mandatory policy rejects it before storage mutation; a permissive caller policy cannot widen it.
3. An out-of-band admin write creates a forbidden root. At the next daily run, DataTug enumerates the entire root set, computes the unexpected-root set and emits one actionable alert without a manual refresh.
4. Nobody acts and the violation persists. Evidence remains visible; duplicate notifications follow a documented reminder policy rather than flooding the recipient. A changed unexpected-root set produces a new alert.
5. The root is removed or explicitly approved. DataTug records recovery and the next run passes. Authentication failure, incomplete pagination and missed runs never look like a healthy empty database.
6. The founder reviews a timestamped deletion manifest with direct records and all descendant counts. Only that exact approved scope can be deleted, after all old writers are disabled.
7. Users, bots, jobs, checkout callbacks and DataTug operate on the new paths. Legacy roots stay absent through an active workload and a subsequent daily check.

### Audit verification

A complete read-only, name-only scan at one fixed snapshot enumerated every page and recursively visited missing parent references. Private operator evidence retains the full per-root/per-subcollection count tables; concrete paths stay outside Git. Independent server-side COUNT queries matched the three largest collection counts. Independent review round 3 recomputed recursive totals, checked retained versus proposed-deletion classifications, and confirmed every deletion approval flag remains false. Plan and rule review rounds 1–2 resolved sequencing and registry-authority findings; no findings remain. SpecScore lint passes. These receipts validate this audit and proposed plan, not deployed runtime controls.

### Verified server checkpoint — 2026-10-27

The verified deployed policy cohort is [Core v0.72.0](https://github.com/sneat-co/sneat-go-core/tree/cc8893075fe41046a363f435ccf094abb9f972d9/firestorepolicy), registry SHA-256 `dbf50f47c7e29251cc016e431e9d0d55e4be3e50f7b53fd42ddf2f110aefd781`: seven approved roots, 47 legacy observation names, an explicitly empty `legacyWritableRoots` list and 64 registered extensions. On secured DALgo paths, the mandatory policy denies routine mutations of all 47 legacy roots. The Firebase registry mirror and sealed monitor configuration use this cohort; the private catalog Check bytes remain unchanged. The earlier six-root/48-writable baseline and intermediate seven-root/63-extension source are historical. Newer [Core source](https://github.com/sneat-co/sneat-go-core/tree/dd62868e19011836d5627aeb8d2a0367ebf8d3fe/firestorepolicy) registers 65 extensions, including space-only `sizeus`; that source change is not a deployment receipt and does not replace the verified 64-extension cohort.

The 2026-10-27 read-only runtime and successful artifact-promotion receipts below bound these eight identified Cloud Run writers to immutable source and image digests. This table is a dated writer baseline; subsequent Host/Telegram rollouts and monitor configuration acceptance are recorded below rather than treating the older images as current. QueryDock is a shared-database writer: its actual DALgo store constructor unconditionally applies Core's root policy; extension registration alone would not prove enforcement.

| Service / region | Source | Serving image SHA-256 |
| --- | --- | --- |
| Sneat / west1 | [b2b5d6a](https://github.com/sneat-co/sneat-go/tree/b2b5d6ad329dbc5b4736ec29250284a4b0930987) | `f70a1e49669a6591e36340f0de6c88f9dd7e11302f6ea384bc8fccd8ab86339a` |
| DataTug / west1 | [c108f25](https://github.com/datatug/datatug-cloud/tree/c108f257ff274e9d571654e1c058f40b5ad0ce04) | `4aaeacbb9633383f982102f55cb242c4527893e115bd067a9967eb9024f98db3` |
| DataTug preview / west1 | [c108f25](https://github.com/datatug/datatug-cloud/tree/c108f257ff274e9d571654e1c058f40b5ad0ce04) | `1659e562f67c1b81bdf9d19fd8dd86baf50f84b58f8faf3239812dcf6768113d` |
| Sneat / west3 | [240d83a](https://github.com/sneat-co/sneat-go/tree/240d83a1481ddb6cccabf86d9ac825b7cdd973c9) | `e5bc8f65b4a46bbe663213bbef4c7d07bae6fd32745f983dc7d4ae5186cfb41a` |
| DataTug / west3 | [c108f25](https://github.com/datatug/datatug-cloud/tree/c108f257ff274e9d571654e1c058f40b5ad0ce04) | `4aaeacbb9633383f982102f55cb242c4527893e115bd067a9967eb9024f98db3` |
| DataTug preview / west3 | [c108f25](https://github.com/datatug/datatug-cloud/tree/c108f257ff274e9d571654e1c058f40b5ad0ce04) | `1659e562f67c1b81bdf9d19fd8dd86baf50f84b58f8faf3239812dcf6768113d` |
| Checkout TEST / west3 | [b2b5d6a](https://github.com/sneat-co/sneat-go/tree/b2b5d6ad329dbc5b4736ec29250284a4b0930987) | `5990a266bbe8e5ae0fa847f7a3ca30f8948de27b9fceb775536b2edcc1be9c83` |
| QueryDock / west1 | [5d6d13f](https://github.com/datatug/querydock-go/tree/5d6d13fd7a301034a936da64fba7693f5f2d8379) | `2af32ab78264a23bc947e3a273d8c6324dc296aa423e6b7d7803fc6312fb1d81` |

At that checkpoint, Host and TEST artifacts passed their normal [production](https://github.com/sneat-co/sneat-go/actions/runs/40511066245) and [TEST](https://github.com/sneat-co/sneat-go/actions/runs/40511065443) workflows. QueryDock's [verified-artifact promotion](https://github.com/datatug/querydock-go/actions/runs/40520503607) binds its source, CI artifact and serving image; its stale service-level source label is not the provenance authority. Regional image-only promotions preserved configuration and routed traffic to the new revisions, with bounded old-request drain checks. These receipts are point-in-time evidence, not permission to assume future deployments remain fenced.

App Engine is verified `USER_DISABLED`: zero instances, the exact 46 version IDs preserved, 514 seconds of bounded drain observation and no completed requests in the bounded post-disable log window. No versions, Firestore records or IAM were deleted. The replacement uptime check passed from six Google checkers after retirement and the existing alert policy points to that exact check; old uptime-check cleanup is separate. Cloud Run Jobs returned zero in the observed all-region listing. Cloud Functions inventory is unknown because its API is disabled. Standalone-game source/build receipts do not establish an additional deployed service. These bounds do not exclude unknown administrative clients or prove universal writer discovery.

### Bounded writer and CI audit — 2026-10-28

The bounded source audit distinguishes secured DALgo paths from native SDK owners inside the same services. IssueNumber public QA and translations use native SDK storage below `spaces/{space}/ext/issuenumber`; DataTug project-search recovery reaches the backend Quick Search owner, whose native transaction writes use `ext/datatug/projectQuickSearch`. These observed paths have approved placement, but bypass mandatory DALgo policy and need owner-specific prevention tests. They do not establish that every SDK mutation is guarded or that an unexpected root was observed. The Sneat `trackus-repair` command also exposed an unwrapped adapter; a focused source repair is under review/landing, without a production-execution claim here.

The next preventive gate is a bounded augmentation of the existing [focused storage-driver CI workflow](https://github.com/sneat-co/cicd/blob/6869be36f4f3d3aa031794331b05a791ed149527/.github/workflows/go-storage-driver-boundary.yml). Its current WB import policy permits SDK imports broadly in hosts and does not certify reviewed constructors. Add repository-wide classified native import sites, including constructor function-value assignments, build-tagged files and nested modules; use exact reviewed owner exceptions and meaningful negative fixtures. Wire the passing gate into each identified product repository's required CI/deploy aggregate. This is proposed work, not an implemented fleet gate. REST/dynamic writes, administrative clients and undiscovered writers remain separate inventory and runtime-test obligations.

### Client, privacy and monitoring acceptance — 2026-10-28

Shared Telegram Quick logins is published in `@sneat/auth-ui` 0.27.46 and uses the scoped status/unlink contract in Host v0.11.7. [Apps PR #615](https://github.com/datatug/datatug-apps/pull/615) landed at `d6bb3c7fd520ceddd92de307c43080af8de18906`; [Main CI 40761876322](https://github.com/datatug/datatug-apps/actions/runs/40761876322) and [normal Cloudflare deploy 40767082946](https://github.com/datatug/datatug-apps/actions/runs/40767082946) succeeded. The official deployment receipt and captured live build information match that source. Live My-page screenshots show Telegram once inside Quick logins, its server-confirmed username and Disconnect action, with the current Google connection retained. Automated tests cover confirmation, cancellation and authenticated unlink. A real-account Disconnect click was blocked by automatic approval review; no live unlink or cancellation-dialog proof is claimed, and the connection was preserved. The independent purchase-recovery banner is outside this acceptance.

Host v0.11.7 reached DataTug and Sneat primary/TEST deployments and the three reviewed regional promotions. These later deployment receipts supersede the old pending Telegram backend claim without widening the bounded eight-writer inventory.

Full privacy rules were activated from Firebase source `8cb3e4219d906673bc3944b003dfec8751e25110`, SHA-256 `3938d8928385531096f7475cb7e962ec4b2ab501f0ad012987f8c932eaf75d42`, by [full-scope deploy 40537201920](https://github.com/sneat-co/sneat-firebase/actions/runs/40537201920). The later reviewed source `14ce816715838b6f4be08f372e94bec1d884a683` has SHA-256 `10ff5789a9af2d4dc3b78a27fdaab12d9a3a8591bb829763ffe59843ec6d6c87`; [deploy 40552180159](https://github.com/sneat-co/sneat-firebase/actions/runs/40552180159) succeeded, with exact live-byte readback at 18:14 UTC on 2026-10-27 and unchanged release metadata at 18:41 UTC. Its rules delta adds an owner-read requirement for personal DataTug favorites; Calendar credential protections, client write denial, wallet admissions/replay and server-private fences remain unchanged. The older digest is historical activation evidence, not the later active-rules cohort. Keep the final execution fence subject to fresh live rules and consumer checks; these receipts do not authorize a reset, legacy fallback, aliases or backfill.

The daily exact-root Check and separate freshness observer are live. Current Google ownership, Telegram link and the approved bot chat were verified before the operator-recipient configuration switch. Independent live acceptance at 20:14 UTC on 2026-10-28 observed one real queued recovery event go from one pending item to zero at 20:11 UTC under the delivery fence. The exact deployed sender removes that item only after successful recipient authorization and a valid provider acknowledgment, supporting provider-acknowledged delivery rather than a human-read claim or persisted raw message receipt. Scanner success and next-run times were unchanged: this delivery did not manufacture a fresh scan. Normal future latest-revision routing was restored without changing the accepted serving revisions, images or runtime configuration. Keep recipient identifiers and credentials private; no identity proof was copied or bot record manufactured. Both jobs still share platform dependencies, so this does not establish total shared-platform outage coverage.

### Preliminary counted refresh — 2026-10-28

A preliminary fixed-snapshot refresh at `2026-10-28T19:41:22.834042Z` counts the 47 legacy roots as 1,708 direct documents plus 296 descendant documents, totaling 2,004 documents across 17 recursive subcollection instances. This refresh is neither approval-ready nor executable and does not revise the immutable historical 47-root / 1,911-record proposal. Seven retained-Space `publicSlug` field resets remain a separate unapproved scope; whole Spaces must not be removed. Fresh final fence/drift checks and the exact counted manifest still require separate founder approval. The immutable audit proposal and its inventory artifacts remain unchanged. No collection, legacy record or retained-Space field deletion has been approved or executed.

### Historical prevention rollout — 2026-10-26

The following records describe the initial baseline only. They are retained as historical receipts and are superseded by the server checkpoint above, not current source/deployment claims.

The deployed baseline registry digest is `7e453f4e97be72eea320ddb0195fbb260e03ec44e82b1880d8f208520cf14f61`: six approved roots, 48 exact temporary legacy allowances and 62 registered extension IDs. Mandatory DALgo policy tests cover direct, transaction, batch and generated-key mutations, including a permissive caller that cannot widen the mandatory policy. Native administrative SDK access can bypass this wrapper; the daily metadata scan provides detection, not prevention, for such writes. Legacy allowances remain visible until their consumers cut over; deployment does not approve new code using those roots.

The private `sneat-co/sneat-datatug-proj` project commit `1aae2bf2e0f5236ed0180b8145cef0c86bb7f08d` owns the sealed Core Check. DataTug Cloud PR #56 landed at `ecf7d282d835ccd71be6ed033357d64acfb3f02e`; Go CI run 40261322393 and automatic deploy 40262196861 succeeded. Production revision `datatug-cloud-00036-8sx` serves 100 percent of traffic with the pinned configuration and existing bot-secret binding.

An unattended authenticated scheduler run completed at `2026-10-26T03:06:02.268180Z`. Core evidence `roots-1` passed with zero unexpected roots: 54 observed roots equal six approved plus 48 frozen legacy names. The next scan is due 24 hours later. The following minute tick left the success time, evidence and state update time unchanged, proving the daily runner did not repeat the scan. Private operator acceptance receipts preserve this historical scan; no collection contents were inspected by this monitor.

The actual Go sender authenticated the approved existing private DataTug Bot destination and delivered one labeled test message, with provider receipt 4 at `2026-10-26T00:46:52Z`. Exact recipient identifiers remain private deployment configuration. Use that bot chat until the founder provides the future Telegram channel ID. The separate sneat-app freshness observer is accepted live as recorded below. Both runtimes depend on Firestore and Cloud Scheduler; their separate jobs detect scanner/job failure, not a total shared-platform outage.

The observer repair [Sneat Go PR #1298](https://github.com/sneat-co/sneat-go/pull/1298) landed at `abaa03c73b0580b3152e403928a11d0aa1260875`. Main CI 40267701436 and production deploy 40267875717 succeeded. Production revision `sneat-app-00016-d6v` serves 100 percent of traffic with the exact pinned operator artifact and read-only existing token binding. A natural five-minute scheduler request at `2026-10-26T04:20:00.177195Z` returned HTTP 200. Scanner success time, next-run time, state update time and complete evidence remained unchanged, so the observer did not manufacture scan freshness. Private operator receipts preserve this historical observer acceptance. A separate checkout TEST deployment failed against a destroyed pre-existing credential version; this production observer acceptance does not assert that TEST deployment succeeded.

The backend repair has a separate unresolved full native Watch CI failure: [run 40257686330](https://github.com/datatug/backend/actions/runs/40257686330) failed initial baselines, then registration-recovery acceptance on one unchanged failed-job retry. Its reviewed source and main merge have identical trees; bounded log inspection found no placement denial or counter-encoding signature, but the underlying deadline cause remains unknown. [Backend issue #117](https://github.com/datatug/backend/issues/117) preserves the evidence and requires retaining the existing authorization, retry-stage proof, race/fencing assertions and deadlines. No further retry or gate waiver is claimed. Scanner-specific CI and production acceptance above do not imply that this full backend gate passed.

### Ownership and root candidates

Approved roots: `ext`, `health_checks`, `spaces` and `users`. Invitations use `ext/invitus/invites`; bot referral attribution uses `ext/bots/referrers`. The 2026-10-29 consolidation places synthetic task probes at `health_checks/cloud-tasks` beside `health_checks/firestore-write`; `_smoke` collection approval is retired, without changing its HTTP routes. OVDB counters belong at `ext/ovdb/state/current` and settings at `ext/ovdb/settings/current`. The prior approvals retaining `invites` and `referrers` at root were superseded by the 2026-10-29 four-root decision. The five initial root-candidate decisions are resolved; this approval does not authorize deletion or establish deployment. The founder approved `botPlatforms → ext/bots/platforms`, `moneyEvents → ext/paymentus/moneyEvents`, both `oauthDevice*` roots under `ext/auth`, and the dedicated `bidding-tictactoe` extension on 2026-10-26. `ext` is a root collection; `ext/checkout` is its extension document; `ext/checkout/customers` is the customer collection.

All `checkout*`, `datatug*`, OVDB, AI, Commītius and game roots must leave root level. Proposed targets do not settle user/space ownership: inspect the existing domain contract before choosing global extension versus space extension paths. Commītius explicitly documents personal global ownership; move it beneath its extension without inventing space ownership. Retain the shared `referrers` root as approved below; do not move it under an extension. Keep cross-account payment/global uniqueness coordinators global within their extension. Checkout keeps the founder's requested `ext/checkout` name although Paymentus owns the code. Verify the game's registered extension ID before using the proposed `bidding-tictactoe` segment.

### Approved referrers retention — 2026-10-26

`referrers` is the seventh approved root. The maintained AnyBot flow records bot acquisition attribution across products: platform (`p`), destination bot (`to`), referrer type/ID (`by`) and creation time (`t`). It has no Space ownership. The native `/start` flow writes it and the bot-level leaderboard reads it; the duplicate Debtus factory has no maintained production caller in the inspected source graph. Retaining the shared root preserves that existing contract without inventing a Space owner. Keep raw attribution server-private and preserve existing authorization; placement approval grants no access. The frozen snapshot contains five direct referrer records and no descendants, all retained by this decision.

The historical deletion proposal remains **47 roots / 1,911 records**, unchanged and unapproved. Retaining five referrer records changes neither that proposal nor the immutable initial audit. Recount and seal a fresh exact manifest only after cutover; no source or root-placement approval substitutes for deletion approval.

### Reviewed placement refinements — 2026-10-25

`invites` is approved to remain at root. This is a placement decision only; no data deletion or runtime deployment is authorized by it.

`slugs` does not need a root-level exception. The current generic registry mixes two semantic scopes behind opaque namespace strings: public-space names such as `sneatclub:space` must be resolved before the Space ID is known; Bookius booking-type names are explicitly unique only within a known space (`bookius:bookingType:<spaceID>`). Global lookup does not require a root collection. Proposed paths are `ext/{extension}/slugNamespaces/{kind}/claims/{slug}` for global names and `spaces/{space}/ext/{extension}/slugNamespaces/{kind}/claims/{slug}` for space-local names. Keep the domain space/entity in its existing space-owned location; the global record is a lookup/uniqueness claim. The founder agreed to this scope split on 2026-10-25; validate typed caller ownership during implementation. Do not infer authorization or scope by splitting opaque namespace strings; pass validated typed scope and owner through the owning caller/key factory.

Claims hold `targetID`, `targetKind`, `claimedAt`, optional `claimedBy`, and release/rename tombstone metadata (`releasedAt`, `successorSlug`). Namespace and normalized slug are in the document path. The existing public lookup supports space registration, slug claim, rename and public resolution; Bookius uses it to create and resolve booking types. At the audit snapshot, the only populated root slug namespaces were `sneatclub:space` (2 claims) and `schoolus:space` (5 claims). This is snapshot evidence, not a claim that every source consumer has production data.

`transfers` is proposed to become `ext/debtus/transfers/{id}` as one authoritative global extension record. A transfer can involve parties associated with different spaces; do not force ownership under one party's space or create independently editable copies. Optional per-space references/projections can support ledgers and discovery, with an explicit consistency/rebuild policy if introduced. Current Debtus `OwnerSpaceID` is optional and anchors linked-task metadata; source comments explicitly preserve the global transfer rather than imply exclusive space ownership. Preserve participant authorization independently of physical placement. The founder approved this target on 2026-10-25; it is not deletion approval.

### Registry and enforcement contract

- One versioned canonical registry must enumerate project/database, approved roots, registered extension IDs, allowed collection-path templates, owner, rationale, sensitivity and review authority. The local `root-collections.json` carries audit classifications and proposals. The promoted runtime policy is public `sneat-go-core/firestorepolicy/root-collections.json` (Core v0.72.0, commit `cc8893075fe41046a363f435ccf094abb9f972d9`), with a digest-verified deployment mirror in private `sneat-firebase/firebase/root-collections.json`. The agent rule points to that canonical registry; backend, CI and DataTug pin its digest/version and fail rather than silently use divergent copies.
- No prefix allow-lists (`checkout*`, `datatug*`, `ovdb*`), no arbitrary `ext/*/**` acceptance. Placement approval never grants access to data. Existing auth, memberships, money fences, TEST/LIVE separation and token secrecy still apply.
- Compose a mandatory storage-placement policy with existing caller/tenant policies using `access.SecureDB(..., access.WithDatabasePolicies(...))`. Read required normalized resources through the current generic policy surface. Keep the Sneat registry in Sneat, not hard-coded inside generic DALgo. No exported DALgo API changes are approved by this plan.
- Cover Insert/Set/Update/Delete, multi-record operations, incomplete/generated keys, write sessions, nested descendants and every read-write transaction handle; reject the whole batch before committing any denied item. Match full hierarchical resources structurally. Keep normal access authorization separate. Before the final cutover, exact existing legacy paths may have temporary read and write placement allowances so deployed consumers keep working; ordinary authorization remains mandatory and the legacy set cannot grow. The verified current policy removes routine legacy write eligibility; frozen observation names remain for detection and are not a client-read grant. Diagnostic reads require separately authorized tooling; normal client access follows the coordinated security rules.
- Audit host startup, Cloud Functions, bot drivers, workers, migration tools, test fixtures and direct Firestore SDK/REST clients. A DAL wrapper cannot guard a raw SDK client or console. Route routine writes through the secured factory, enforce a code/import boundary against bypasses, and restrict production service-account use to reviewed entry points. `sneat-go` remains wiring only; policy implementation belongs in a reusable owning library and each module owns its keys.
- The existing Firestore rules deny client writes. Server libraries use IAM and [bypass those rules](https://firebase.google.com/docs/firestore/security/rules-conditions); a rules-only root check cannot prevent server-created roots. Audit the broad read catch-all when introducing `/ext`: membership-looking fields must never make credential/financial/control subtrees client-readable.

### CI contract

Tests must prove approved examples, arbitrary unknown roots, typo roots, deprecated roots, unknown extension IDs and mis-scoped paths. Parameterize the mutation matrix across direct writes, batches, transactions, incomplete keys and descendants beneath forbidden roots. Include all-or-nothing batch behavior, rollback on transaction failure, policy failure, and caller-allow versus mandatory-deny cases. Exercise real host wiring, not only a standalone policy helper. Inventory roots after representative emulator journeys (account/space setup, checkout TEST callback, bot action, DataTug project/watch and OAuth) and compare with the registry. The test starts with an empty database and must catch the first accidental root. Add a source scan as supplementary lint for direct SDKs and unregistered key factories; literal grep alone is not the runtime proof.

### Daily DataTug Check and alert contract

- Use an operator-owned scheduled DataTug Core Check against a metadata-only Firestore schema source, project `sneat-eur3-1`, database `(default)`. Enumerate collection IDs with pagination; compare exact names against the approved registry. The runner reuses the sealed Core catalog and evaluator; unattended production metadata-source execution passed on 2026-10-26 as recorded above. Independent freshness-observer activation also passed on 2026-10-26 as recorded above.
- Run the operator-owned scheduled Core Check every 86400 seconds; run once at activation so existing unexpected roots alert immediately, not baseline silently. Use a 24-hour interval from the first successful scan and retry source failures after 15 minutes; no fixed local-time or DST promise is made. Persist last successful scan time and registry digest.
- Emit deterministic evidence: unexpected roots, legacy roots awaiting approved cleanup, newly appeared/disappeared names, policy digest, project/database and observation time. Never send document contents, customer IDs, tokens or credential fields. The daily health check needs root metadata only; recursive counting is a separate on-demand audit.
- Every root outside the four approved roots is actionable, including historical legacy observation names. Preserve historical classification in evidence, but never exclude it from the Check or alert payload. Routine write eligibility for legacy names is empty. Alert on changes through the existing durable delivery path, without repeating unchanged findings on every scan. Observation metadata is not a placement or compatibility exception.
- On first violation or changed unexpected set, notify the chosen existing DataTug channel; deduplicate identical state. On recovery emit one recovery event. The founder approved the existing private DataTug Bot destination. Keep its exact destination in deployment configuration until the founder supplies a Telegram channel ID. This operator channel is separate from personal Watch enrollment; never manufacture nonce proof or change the existing bot webhook. Alert on scan/auth/pagination failure and add an independent freshness check for last successful scan older than 30 hours, so a dead scheduler cannot silently monitor itself.
- Validate healthy → forbidden-root detection → alert receipt → unchanged-state deduplication → changed-set alert → cleanup → recovery, plus auth failure, pagination failure and runner downtime. Use an emulator for deliberate violations; prove one authenticated production root scan and an explicitly labeled test delivery through the real channel. Capture delivery receipt and unattended scheduled-run receipt before claiming monitoring live.

### Destructive cutover and deletion contract

Full replacement, **no copying, dual reads, dual writes or compatibility aliases**. Inventory and switch every reader/writer before deleting anything: backend modules, browser subscriptions, bots, Cloud Functions, cron/task workers, webhooks, triggers, indexes, collection-group queries, TTL policies, fixtures, scripts, serialized document references and stored path strings. [Parent deletion does not delete subcollections](https://firebase.google.com/docs/firestore/manage-data/delete-data); recursive deletion must cover existing records beneath missing parents too. Deleting a root changes neither external Stripe objects nor GitHub/Google grants. Prevent replayed callbacks from silently reconstructing invalid entitlements after local payment ledgers reset; test the reset behavior and fail closed until fresh trusted provider facts are reconciled.

For the founder's review produce an immutable manifest with exact project/database, exact root collection names, each concrete descendant collection path, direct existing-document count, direct and recursive subcollection-instance counts, descendant-document count, total document count, missing-parent count, scan readTime, proposed target and consumer-cutover status. Include a summary grouped by subcollection path template plus a private concrete-path appendix. Mark every entry `deletionApproved: false` initially. Never enumerate sensitive record contents to populate this report.

Deletion is a separate final execution gate. Stop old jobs/writers and establish a fence; recount and seal the manifest. The founder approves its digest and exact scope. Any newly observed collection path, changed counts, stale cutover state or changed manifest requires renewed review rather than widening deletion silently. A separate deletion identity can have scoped cleanup access; application identities cannot bypass placement policy. Delete only explicitly approved legacy roots recursively, never `ext`, `users`, `spaces`, `_smoke`, `health_checks`, `invites` or `referrers` by a broad exclusion/prefix sweep. A reviewed candidate is still excluded until its deletion entry is explicitly approved. Restore writer traffic only on the new version; re-enumerate the entire tree and exercise real workflows. If rollback is needed after deletion, code must continue using approved paths; do not roll back to a legacy writer that recreates roots.

### Historical source-cutover checkpoint — 2026-10-26

The following subsection records the earlier implementation state verbatim. The verified policy cohort and 2026-10-28 client/monitor checkpoints above supersede its deployment and remaining-source statements; its audit and unapproved cleanup scope remain historical.

The founder authorized proceeding with source cutover. Reviewed integration branches relocate checkout, DataTug private state, AI allowance/usage, OVDB, Commītius, slug claims and Debtus transfers. Readers, query parents and result prototypes move together. These are implementation receipts, not production acceptance or deletion authorization.

The prepared Core root registry schema version 2 (source commit `c2408dcfc61eea4aec32a407ac1c309d37371ff6`, registry SHA-256 `de2fb1b69f5e630ebc6c52cedea9d5462c7f3b52314e1d0db45fdf0a9aa905b7`) classifies `referrers` as approved, retains the other 47 inventory names in `legacyRoots`, and sets `legacyWritableRoots` to an empty list. All 47 legacy roots are fenced from routine mutations in that prepared policy; observation entries remain until approved cleanup. The prior intermediate source digest `b285091803f1ab712f81470c90925cec6678ac7335874baa6ac80ee96c4a5d0c` (Core `72cfa7e94db6236127fbdebb76956c903e848c9d`) still treated `referrers` as the sole writable legacy exception and is superseded by the retention decision. The deployed scanner and observer keep the verified earlier `7e453f4e97be72eea320ddb0195fbb260e03ec44e82b1880d8f208520cf14f61` baseline (six approved roots plus 48 temporary read/write legacy allowances) until exact library, mirror and sealed configuration pins roll out together. Prepared source is not deployment evidence.

The final client rules source passed 286 emulator tests, including private bot/auth/game/money paths and per-user wallet invoice admissions and replay fences. Independent review found no issues; deployment remains unverified.

A retained-Space metadata projection at `2026-10-26T10:32:37.781175Z` found 60 Spaces: seven retain old string `publicSlug.namespace`, 53 have no slug metadata, and none carry new typed `scope`. Ordinary authorized Space reads and updates remain compatible. The seven stale slug fields require a separately counted, preconditioned field-reset proposal in the deletion manifest; whole Spaces must not be removed. No reset is authorized.

The bot cutover must verify real authenticated enrollment with the founder’s current Google sign-in and the approved Telegram chat. Existing identity proof is not copied. Fresh invoice admissions bind original user, native payer, bot, mode, amount and currency before issuance; adapter integration checks remain in progress. Do not enable a reset bot identity store until these checks and authenticated enrollment are complete. OVDB external-capacity reconciliation and competition Space ownership remain separate cutover gates.

Checkout completion requires fresh server-created session admission in the new namespace; old re-signed events cannot reconstruct entitlements, and updates cannot expand a personal admission into a Space. OVDB capacity counters fail closed until explicitly initialized from reconciled external-resource authority. No initialization or external-resource cleanup has been performed.

The unchanged full native Watch journey passed locally using backend `853211b18e607fcaec3c90e1ad8ad2e5d280a7d1` and CLI `0b76f32620d919b6973da39ddc118d96091990fb`, with three real emulator runners and recovery receipts. The earlier registration timeout did not reproduce; this does not establish its cause or exercise a registration deadline that did not occur. Backend `0c309f9d60c0c9a75b70546fc51a047ae126425d` subsequently pins that exact CLI for CI. Exact CI and deployed acceptance remain required.

Further inspection found eight active Bookius root factories absent from the frozen live inventory. Space-owned booking storage and the global vendor routing registry are being cut over with callers and contracts. Competition target/request/reservation storage still needs trusted owning-space propagation through Competios, Bookius and payment references; total capacity cannot establish zero existing occupancy after a reset. This remains an explicit full-cutover gap, with unapproved root mutations already denied. Unused constants do not establish populated collections or deletion scope.

## Tasks

### Task 1: Measure and review collection ownership

**Id:** t-001

**Status:** complete
**Implemented-by:** sneat-co/sneat-specs@2bc5b6d0217085fe453673ccbd409fd738001a62
**Note:** Fixed-snapshot recursive audit and independent count verification are complete; deletion approval remains separate.

**Verifies:** none — source-less audit plan; acceptance: all live roots classified with complete recursive counts, no partial scan reported as zero.

Owner: audit runner + founder. Authenticate read access, fix one snapshot readTime, enumerate all roots and descendants, include missing parent documents and pagination, and record metadata-only counts and concrete-path appendix privately. Compare the live roots to the supplied list and additional source risks. Founder decides each root candidate and target ownership. Outputs: reviewed registry and deletion-manifest draft. No delete calls.

### Task 2: Land the agent rule and canonical registry

**Id:** t-002

**Status:** complete
**Implemented-by:** sneat-co/sneat-go-core@32f9196f617670cc5cc99d6c50094c54344a5929
**Note:** Canonical Core registry, digest-verified Firebase mirror and shared AI-agent trigger are landed; deployed scanner uses the same digest.

**Verifies:** none — acceptance: AI instructions resolve one rule; policy, tests and monitor resolve one reviewed registry digest.

Owners: Backstage rule, Workbench instruction renderer, Sneat Firebase registry. Land `firestore-approved-root-collections` and add its trigger to shared agent instructions; label it Stated until real controls ship. Promote approved path templates to the runtime registry and document root addition authority. Include the founder's separate deletion-list approval gate. A registry change needs explicit review; scanning the live database cannot automatically approve what exists.

### Task 3: Enforce mandatory placement at every backend write boundary

**Id:** t-003

**Status:** in_progress

**Verifies:** none — acceptance: a new forbidden root cannot be created via any routine backend mutation surface; ordinary permitted work still passes.

Owners: reusable Sneat policy library, sneat-go-host construction/wiring, module key factories. Reuse existing DALgo access composition. Inventory and close raw-SDK bypasses. Cover host startup and all modules, jobs and bots. The verified Core v0.72.0 cohort has no writable legacy allowance; keep observation names visible until approved cleanup and never allow a new root. Historical rollout allowances are not current compatibility storage. Fail startup on missing/invalid policy. The final new-path deployment removes write eligibility for every relocated root before deletion; frozen observation names remain until approved cleanup, and any future root decision requires explicit review; no current legacy write exception remains. Any necessary DALgo gap is a separately reviewed additive proposal subject to its repository's human API-approval rule.

### Task 4: Add CI placement and security regression gates

**Id:** t-004

**Status:** in_progress

**Verifies:** none — acceptance: unknown/legacy roots and transaction/batch/native-SDK bypass tests fail before deployment.

Owners: policy library tests, Sneat Firebase emulator/rules suite and host CI. Implement the mutation matrix and representative clean-database journeys above. Update rules to explicitly protect the new global extension subtrees and preserve all existing access boundaries; remove stale root grants after consumers switch. Test secrets and financial/control data against malformed membership fields. Required CI becomes blocking when the new implementation is green; do not ship a permanently red or advisory-only gate.

### Task 5: Activate the daily DataTug dogfood Check and freshness alarm

**Id:** t-005

**Status:** complete
**Implemented-by:** sneat-co/sneat-go@abaa03c73b0580b3152e403928a11d0aa1260875
**Note:** Daily Check and separate observer activation are accepted. Current Google/Telegram ownership and bot-chat enrollment were verified, the recipient configuration was applied, and a real queued event was provider-acknowledged under the delivery fence on 2026-10-28. This is neither a human-read claim nor a new scan; shared-platform outage coverage is not claimed.

**Verifies:** none — acceptance: exact-root set evaluation, one real approved bot-chat delivery, an unattended scheduled-run receipt, and failure/freshness coverage.

Owners: DataTug schema-source/check implementation and platform operations. Use the existing sealed catalog and Core EvaluateCheck contracts for an operator-only metadata source and daily runner. The private sneat-co/sneat-datatug-proj repository owns the exact Check artifact; pin its bytes, digest and commit in operator configuration. Invoke scanning from the DataTug minute catchup task. Invoke freshness and delivery separately from sneat-app and its five-minute cron job; this observer never enumerates roots or updates scan success. Bind the existing bot secret read-only without rewriting it or changing IAM. Test delivery to the approved chat. Keep source credentials in the established secret binding. Monitoring stores evidence beneath its extension/space extension, creating no new root. Prove the whole detection/recovery journey and out-of-band heartbeat failure detection.

### Task 6: Switch every consumer to approved paths and seal deletion manifest

**Id:** t-006

**Status:** in_progress

**Note:** Secured DALgo paths within eight identified Cloud Run service/region slots have source/image-bound server-policy receipts and App Engine is disabled with bounded drain proof. Shared Telegram client deployment, full privacy-rule activation and current authenticated bot delivery have acceptance receipts. Fleet writer/bypass discovery and CI coverage audit, fresh final fence/drift checks and an approval-ready exact counted manifest remain incomplete; no deletion or retained-Space field reset is authorized.

**Verifies:** none — acceptance: zero routine old-path callers/writers, green new-path workflows and immutable fully counted manifest presented to founder.

Owners: Paymentus/checkout, DataTug, core modules, OVDB, Commītius, games, bot drivers and clients, with host wiring only in sneat-go. Replace all paths including root queries, parent-null assumptions and serialized key reconstruction; prepare indexes/TTL/rules/triggers before new traffic. Quiesce old workers/tasks and fence writes. No copy/backfill. Exercise fresh setup, auth, project creation, search, Watch and TEST checkout callbacks; explicitly verify stale event replay cannot recreate entitlements. Do not perform live payments. Recount under the fence and request approval of the exact manifest digest. Wait for that approval.

### Task 7: Execute only the founder-approved deletion scope and verify absence

**Id:** t-007

**Status:** planning

**Verifies:** none — acceptance: approved roots recursively removed, every kept root retained, no old root returns under real workload and the next daily scan.

Owner: one named cleanup operator. Validate project/database/manifest digest and cutover fence, abort on drift, then delete only approved concrete legacy trees including descendants under missing parents. Record per-entry results, failures and post-delete enumeration; resume idempotently within that same scope. No wildcard sweep. Validate new-path workflows, no security regression, no old roots and DataTug recovery. Capture deployed revision, policy digest, CI receipt, scheduled scan and delivery receipts. The effort is complete only when these controls are live and cleanup is verified; writing this plan is not that completion.

## Open Questions

- Implementers: resolve any remaining domain/API ownership gaps without changing the approved root decisions; referrers retention, the dedicated game extension, slug scope split and global Debtus transfer placement are approved.
- Operations: preserve the verified private bot recipient and normal future deployment routing; a future destination change requires its own reviewed authorization and delivery evidence.
- Implementers: complete the remaining fleet placement/bypass inventory and CI coverage; retain exact-source deployment and scheduled-run evidence for future monitor changes.
- Founder final gate: review and approve the exact counted recursive deletion manifest after cutover and writer fencing. No deletion has been approved or performed.

---
*This document follows the https://specscore.md/plan-specification*
