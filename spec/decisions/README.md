---
format: https://specscore.md/decisions-index-specification
---

# Decisions

## Decisions

| # | Decision | Status | Date | Tags | Affected |
|---|----------|--------|------|------|----------|
| [0001](0001-unified-invite-and-rsvp-model.md) | Unified invite & RSVP model | In Review | 2026-06-29 | — | — |
| [0002](0002-reserved-extension-space-ids.md) | Spaceless system namespace for global extension records | In Review | 2026-06-29 | — | `reserved-extension-space-ids` — **removed** (Idea + Feature): redundant with this Decision, which is now the authoritative record for the spaceless namespace., [`system-space-type`](../features/system-space-type/README.md) — **superseded**: its space-type-level ACL is replaced by per-record authorization., [`eventus/mini-products/togethered`](https://github.com/sneat-co/backstage) (backstage) — records at `/ext/togethered/...`., Decisions [0001](0001-unified-invite-and-rsvp-model.md) and [0003](0003-invite-acceptance-graph-edges.md) — drop `$<ext>` space references. |
| [0003](0003-invite-acceptance-graph-edges.md) | Invite acceptance — graph edges, membership, and authority | In Review | 2026-06-29 | — | — |
| [0004](0004-sneat-family-domain-and-positioning.md) | sneat.family — domain role, family positioning, and marketing site | In Review | 2026-07-13 | website, brand, positioning, seo, domains | None yet (marketing site; no SpecScore feature depends on it). Related product |
| [0005](0005-sneat-work-hosting-and-team-merge.md) | Sneat.work hosting (app.sneat.work) and the Sneat.team merge | In Review | 2026-07-13 | website, brand, hosting, domains, auth, seo | None directly (marketing + hosting). Related specs referenced descriptively: |
| [0006](0006-unified-space-registration.md) | Unified space registration — generic space types with module markers | In Review | 2026-08-23 | spaces, extensions, registration, vendors, authorization, space-types | `core-space-capability` (sneat-core-modules, Stable) — `validateOrdinarySpace` becomes registry-driven, spaceus public-slug (sneat-core-modules) — gains the HTTP route it currently lacks, schoolus / gametable / communitycentrum / sneat-club — vendor & space registration, vendor-payment-bots (backstage decision 0010) — `VendorSpaceID` links a payment bot to the registered space |

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/decisions-index-specification*
