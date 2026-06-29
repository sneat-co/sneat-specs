---
format: https://specscore.md/features-index-specification
---

# Features

Feature specifications for this project.

## Index

| Feature | Status | Description |
|---------|--------|-------------|
| [system-space-type](system-space-type/README.md) | Deprecated | **Superseded by [Decision 0002](../decisions/0002-reserved-extension-space-ids.md)** — replaced by the spaceless [system namespace](reserved-extension-space-ids/README.md) (`/ext/`) with per-record access control. (Was: a SpaceTypeSystem space for shared cross-user records.) |
| [reserved-extension-space-ids](reserved-extension-space-ids/README.md) | Draft | A spaceless system namespace for an extension's global, cross-user records: they live at `/ext/{ext-id}/...` (not in any space), referenced by a suffix-less `{ext-id}/{collection}/{doc-id}` ref, with the linkage validator/resolver gaining a spaceless branch and access authorized per-record. Resolves the addressing open question from the now-superseded system-space-type. |
| [guardian-consent](guardian-consent/README.md) | Approved | Capture and store a verifiable parental-consent record (the lawful-basis artifact) at guardian-link, in the system namespace (`/ext/{ext-id}/...`), with lightweight verification; gate a minor's data until valid consent exists. |
| [jurisdiction-resolver](jurisdiction-resolver/README.md) | Approved | Map a subject to the applicable child-privacy rule set + consent age from ordered signals (declared country, else deduced from venue/region/locale; never IP-geo), backed by a static country-level config, with a strictest-applicable fallback when signals conflict or are absent. |

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/features-index-specification*
