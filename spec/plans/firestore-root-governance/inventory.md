# Firestore root collection inventory — 2026-10-25

Project: `sneat-eur3-1`; database: `(default)` (console confirms location `eur3`, Standard edition).

The founder supplied this inventory; the open Firebase console independently displays the same root names. There are 54 roots: 6 approved baseline roots, 43 relocation proposals, and 5 root candidates for founder review. Classification is a proposal, never deletion approval.

**Counts measured successfully.** The complete fixed-snapshot metadata scan and per-root/per-subcollection tables are in the private workspace report `.wb/runtime/firestore-root-governance/root-review.md`. Concrete document IDs and descendant paths stay in its private JSON/CSV appendix; they are not committed to the specification repository. The dashes below intentionally mean “see private measured report”, not zero.

Direct records count only existing documents at that collection path. Direct subcollection instances count distinct child collection paths beneath every document reference at that level, including missing parent documents. Recursive instances count all deeper collection paths, not just distinct collection IDs. Descendant records count all existing documents below those child collections, excluding the root collection's direct records. Total records = direct + descendants. The final scan must provide a second table for every concrete subcollection path, its direct records and recursive totals; do not collapse identical names under different parents.

| Root collection | Classification | Proposed collection path | Direct records | Subcollections: direct / recursive | Descendant records |
|---|---|---|---:|---:|---:|
| `_smoke` | approved | `_smoke` | — | — / — | — |
| `aiCloudAllowance` | relocate | `ext/aicloud/allowances` | — | — / — | — |
| `aiUsage` | relocate | `ext/aigateway/usage` | — | — / — | — |
| `botPlatforms` | review-root-candidate | `ext/bots/platforms` | — | — / — | — |
| `bttt-matches` | relocate | `ext/bidding-tictactoe/matches` | — | — / — | — |
| `checkoutAccountAcceptedRevisions` | relocate | `ext/checkout/accountAcceptedRevisions` | — | — / — | — |
| `checkoutAccountFinancialEvents` | relocate | `ext/checkout/accountFinancialEvents` | — | — / — | — |
| `checkoutAccountInvoiceIndex` | relocate | `ext/checkout/accountInvoiceIndex` | — | — / — | — |
| `checkoutAccountOwners` | relocate | `ext/checkout/accountOwners` | — | — / — | — |
| `checkoutAccountPaymentIndex` | relocate | `ext/checkout/accountPaymentIndex` | — | — / — | — |
| `checkoutAccountSubscriptionIndex` | relocate | `ext/checkout/accountSubscriptionIndex` | — | — / — | — |
| `checkoutAccountSubscriptions` | relocate | `ext/checkout/accountSubscriptions` | — | — / — | — |
| `checkoutBillingAccountIndex` | relocate | `ext/checkout/billingAccountIndex` | — | — / — | — |
| `checkoutCustomerIndex` | relocate | `ext/checkout/customerIndex` | — | — / — | — |
| `checkoutCustomers` | relocate | `ext/checkout/customers` | — | — / — | — |
| `checkoutEntitlements` | relocate | `ext/checkout/entitlements` | — | — / — | — |
| `checkoutOfferBuyers` | relocate | `ext/checkout/offerBuyers` | — | — / — | — |
| `checkoutOfferLedgers` | relocate | `ext/checkout/offerLedgers` | — | — / — | — |
| `checkoutOfferPlaces` | relocate | `ext/checkout/offerPlaces` | — | — / — | — |
| `checkoutQuoteHistory` | relocate | `ext/checkout/quoteHistory` | — | — / — | — |
| `checkoutSessionIndex` | relocate | `ext/checkout/sessionIndex` | — | — / — | — |
| `checkoutUnmappedFinancialEvents` | relocate | `ext/checkout/unmappedFinancialEvents` | — | — / — | — |
| `checkoutUnmappedPayments` | relocate | `ext/checkout/unmappedPayments` | — | — / — | — |
| `commitiusCommitments` | relocate | `ext/commitius/commitments` | — | — / — | — |
| `currentSpaceServiceRefreshDuePartitionsV1` | relocate | `ext/paymentus/currentSpaceServiceRefreshDuePartitionsV1` | — | — / — | — |
| `datatugGithubProjectCreateOperations` | relocate | `ext/datatug/githubProjectCreateOperations` | — | — / — | — |
| `datatugGithubProjectLocators` | relocate | `ext/datatug/githubProjectLocators` | — | — / — | — |
| `datatugGithubQueryOperations` | relocate | `ext/datatug/githubQueryOperations` | — | — / — | — |
| `datatugPaidMoneyMonths` | relocate | `ext/datatug/paidMoneyMonths` | — | — / — | — |
| `datatugPersonalLogGrants` | relocate | `ext/datatug/personalLogGrants` | — | — / — | — |
| `datatugPersonalLogOperations` | relocate | `ext/datatug/personalLogOperations` | — | — / — | — |
| `datatugPersonalLogSubjects` | relocate | `ext/datatug/personalLogSubjects` | — | — / — | — |
| `datatugPersonalLogUsers` | relocate | `ext/datatug/personalLogUsers` | — | — / — | — |
| `datatugPlanApplications` | relocate | `ext/datatug/planApplications` | — | — / — | — |
| `datatugProjectQuickSearch` | relocate | `ext/datatug/projectQuickSearch` | — | — / — | — |
| `datatugWatchControl` | relocate | `ext/datatug/watchControl` | — | — / — | — |
| `datatug_github_app_credentials` | relocate | `ext/datatug/githubAppCredentials` | — | — / — | — |
| `datatug_github_oauth_app_credentials` | relocate | `ext/datatug/githubOAuthAppCredentials` | — | — / — | — |
| `datatug_projects` | relocate | `ext/datatug/projects` | — | — / — | — |
| `ext` | approved | `ext` | — | — / — | — |
| `health_checks` | approved | `health_checks` | — | — / — | — |
| `invites` | approved | `invites` | — | — / — | — |
| `moneyEvents` | review-root-candidate | `ext/paymentus/moneyEvents` | — | — / — | — |
| `oauthDeviceAuthorizations` | review-root-candidate | `ext/auth/deviceAuthorizations` | — | — / — | — |
| `oauthDeviceUserCodes` | review-root-candidate | `ext/auth/deviceUserCodes` | — | — / — | — |
| `ovdbAccessTokens` | relocate | `ext/ovdb/accessTokens` | — | — / — | — |
| `ovdbDeviceAuthorizations` | relocate | `ext/ovdb/deviceAuthorizations` | — | — / — | — |
| `ovdbDeviceUserCodes` | relocate | `ext/ovdb/deviceUserCodes` | — | — / — | — |
| `ovdbWritableSandboxCapacity` | relocate | `ext/ovdb/writableSandboxCapacity` | — | — / — | — |
| `referrers` | review-root-candidate | `ext/anybot/referrers or ext/debtus/referrers` | — | — / — | — |
| `slugs` | relocate | scope-dependent extension `slugNamespaces/{kind}/claims` | — | — / — | — |
| `spaces` | approved | `spaces` | — | — / — | — |
| `transfers` | relocate | `ext/debtus/transfers` | — | — / — | — |
| `users` | approved | `users` | — | — / — | — |

Targets are proposed storage ownership, not authorization grants. Space-owned data defaults to `spaces/{space}/ext/{extension}/…`; global extension coordinators use `ext/{extension}/…`. An `ext` root allow-list alone does not approve arbitrary extension IDs or child paths. Preserve stable document IDs where useful; reset data rather than copy it, per founder direction. The `ext/checkout` namespace follows the founder's explicit example even though Paymentus owns its backend.

## Source evidence and consumers

Remote `origin/main` was fetched on 2026-10-25. Read these exact revisions before implementing; this is source evidence, not proof of what revision is deployed:

| Owner repository | Revision | Evidence |
|---|---|---|
| sneat-co/paymentus | `3085de99e56132d6b3789118eb7da2dbff206aad` | `backend/subscriptions/offer_ledger.go:22-28,221-233`; `account_reconciliation_store.go`; `space_service_current_refresh.go:121`; `backend/moneyevents/dalgostore/dalgostore.go:18` |
| datatug/backend | `3e61e5ffd92b718b094f476a6e45daa742012cb8` | `models4datatug/project.go:16,87`; `github_app_auth.go:11-13`; `github_project.go:78,154`; `plan_application.go:12-13`; `facade4datatug/watch_cloud.go:86`; `quicksearch4datatug/firestore.go:14`; `googlecloudlogs4datatug/registry.go:29-31` |
| sneat-co/sneat-core-modules | `365c49eb191fdcb2ae1c7956979677e81a388b22` | `auth/oauth/const4oauth/const.go:9-11`; `aicloud/dbo4aicloud/allowance_dbo.go:16`; `aigateway/facade4aigateway/record_usage.go:17`; `invitus/facade4invitus/record_contexts.go:9` |
| sneat-co/ovdb | `e49ea8f4f167cdd313cb994df75ef011c8b9b1dc` | `backend/dal4ovdb/device_auth_store.go:17-19,351-359`; `sandbox_store.go:16-18`; `connection_store.go:18-20` |
| sneat-co/commitius | `2b83dd89f76c494a6af90c6355aa9e300dbcab5e` | `backend/dal4commitius/keys.go:12-17` explicitly cites ADR 0009 for personal, non-space storage; global extension placement can preserve that model |
| sneat-co/debtus | `909d39afb9604913c14301da7261db7f6e7e40de` | `backend/debtus/models4debtus/transfer.go:62`; `referrer.go:9` |
| bots-go-framework/bots-fw-store-dalgo | `743adf61fec59b40e8f4a0d8d095bfde3f1c64ee` | `store.go:24` platform root; also inspect sibling Telegram driver and wallet bot-charge consumers |
| sneat-games/bidding-tictactoe | `18242e5e2b0823c545364977c735fe40f153c3e2` | `server-go/btttmatch/store.go:11` |
| sneat-co/sneat-go-core | `3855e233054ce6b6ca8fdb095b07d805510e02bf` | `slugs/types.go:18,22-30,96-108,148` shared registry mixes global and space-local opaque namespaces |
| dal-go/dalgo | `d8ee60a8be438d34bf0716642649e0060070fcec` | `access/database.go:54-56,81` mandatory database policies; existing generic policy API avoids a new exported DALgo interface |
| datatug/datatug-cli | `d84b81689e157b5f2f3f95c6e8b4513e57a1f339` | `pkg/schemers/firestoreschema/simple_provider.go` lists root/parent collections; `pkg/watch/definitions.go` accepts interval 86400 seconds; neither proves a deployed schema-enumeration Check |
| datatug/datatug-apps | `6d2b35dcdbe4a043c8367627e84eaa3aa557f76a` | `libs/datatug/main/src/lib/services/repo/project-item-service.ts:137`; `datatug-store.service.firestore.ts:15,100` hard-code root reads and subscriptions; change with backend and rules |
| sneat-co/sneat-firebase | `5929a3c925caa5319667b6c119c7b902574f35e0` | `firebase/firestore.rules` has client read grants, no write grants, legacy DataTug project roots and root catch-all membership-shaped grants |

## Additional source roots absent from the console snapshot

These are **source risks to inspect**, not claims of populated production collections and not additions to the deletion list: `oauthAuthorizationCodes`, `ovdbWritableSandboxes`, `ovdbWritableSandboxOwners`, `ovdbConnections`, `ovdbConnectionSessions`, `ovdbSpaceConnections`, `datatug_github_oauth_states`, DataTug personal-log pending/completion/terminal/cleanup roots, AI question/call/day/person roots, and Commītius drafts/certificates/wallet roots. Scan the full deployed dependency graph for runtime-generated names, not just prefixes or this list.

## Root candidates for founder review

- `botPlatforms`: shared bot framework coordination; recommend `ext/bots/platforms` unless declared platform infrastructure.
- `invites`: founder approved root placement on 2026-10-25; removed from root candidates.
- `moneyEvents`: shared money evidence, but currently owned by Paymentus; recommend `ext/paymentus/moneyEvents` with existing fences intact.
- `oauthDeviceAuthorizations`, `oauthDeviceUserCodes`: platform auth bootstrap before a space exists; either explicitly approve roots or put global state under `ext/auth`.
- `slugs`: removed from root candidates; proposed relocation separates global public-space lookup claims from space-local claims. See the plan refinement; final target templates remain under review.
- `referrers`: Debtus and Anybot use the same name; classify records and intended shared contract before choosing one owner.
- `transfers`: removed from root candidates; propose one authoritative `ext/debtus/transfers/{id}` record, with optional space-local references/projections. This follows the founder suggestion and current optional `OwnerSpaceID` semantics; it is not deletion approval. Do not conflate with Paymentus transfer subcollections.

The recommendation is to keep the six approved roots unless a reviewed infrastructure reason warrants an exception. No root candidate is automatically allowed by the proposed registry.

## Slug and transfer follow-up source evidence

Refreshed owning remote sources on 2026-10-25: Sneat Core `3855e233054ce6b6ca8fdb095b07d805510e02bf` (`slugs/types.go:22-30,96-108`); core modules `365c49eb191fdcb2ae1c7956979677e81a388b22` (`spaceus/facade4spaceus/resolve_public_slug.go:31-44`); Bookius `a09fe803088ee1be3e458380d2f2705b2a4c5724` (`backend/facade4bookius/booking.go:100-105,213`); Debtus `909d39afb9604913c14301da7261db7f6e7e40de` (`backend/debtus/models4debtus/transfer.go:225-230`). Slugs hold target identity/type, claim audit fields and release/rename metadata, not the target entity's domain data. The private audit snapshot has two Sneat Club claims and five Schoolus claims; no Bookius root claims were observed.
