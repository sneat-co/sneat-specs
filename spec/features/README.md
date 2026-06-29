---
format: https://specscore.md/features-index-specification
---

# Features

Feature specifications for this project.

## Index

| Feature | Status | Description |
|---------|--------|-------------|
| [system-space-type](system-space-type/README.md) | Deprecated | **Superseded by [Decision 0002](../decisions/0002-reserved-extension-space-ids.md)** — replaced by the spaceless system namespace (`/ext/`) with per-record access control, specified in that Decision. (Was: a SpaceTypeSystem space for shared cross-user records.) |
| [guardian-consent](guardian-consent/README.md) | Approved | Capture and store a verifiable parental-consent record (the lawful-basis artifact) at guardian-link, in the system namespace (`/ext/{ext-id}/...`), with lightweight verification; gate a minor's data until valid consent exists. |
| [jurisdiction-resolver](jurisdiction-resolver/README.md) | Approved | Map a subject to the applicable child-privacy rule set + consent age from ordered signals (declared country, else deduced from venue/region/locale; never IP-geo), backed by a static country-level config, with a strictest-applicable fallback when signals conflict or are absent. |
| [minor-data-protection](minor-data-protection/README.md) | Approved | A central minor-data policy guard that every collection/share/ad/profiling point consults to enforce data-minimization defaults and a binding no-behavioral-ads constraint for a known minor, for as long as they are a minor. |

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/features-index-specification*
