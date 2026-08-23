---
format: https://specscore.md/features-index-specification
---

# Features

Feature specifications for this project.

## Index

| Feature | Status | Description |
|---------|--------|-------------|
| [System Space Type](system-space-type/README.md) | Deprecated | **Superseded by [Decision 0002](../../decisions/0002-reserved-extension-space-ids.md)** — replaced by the spaceless system namespace (`/ext/`) with per-record access control, specified in that Decision. |
| [Guardian Consent](guardian-consent/README.md) | Approved | Capture and store a **verifiable parental-consent record** — the lawful-basis artifact for processing a minor's data — at the **guardian-link** step of the invitus accept flow ([Decision 0003](../../decisions/0003-invite-acceptance-graph-edges.md)). |
| [Jurisdiction Resolver](jurisdiction-resolver/README.md) | Approved | A **stateless, near-pure platform resolver** that maps a subject (a minor in context) to the applicable **child-privacy rule set + consent age**, from ordered caller-supplied signals and never from IP-geo. |
| [Minor Data Protection](minor-data-protection/README.md) | Approved | A central **minor-data policy guard** — `minorDataPolicy(subject, regime) → constraints` — that **every** collection, sharing, advertising and profiling decision point across products consults to enforce **data-minimization defaults** and a binding **no-behavioral-ads / no-profiling** constraint for a known **minor**. |

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/features-index-specification*
