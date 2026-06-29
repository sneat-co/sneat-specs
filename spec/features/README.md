---
format: https://specscore.md/features-index-specification
---

# Features

Feature specifications for this project.

## Index

| Feature | Status | Description |
|---------|--------|-------------|
| [system-space-type](system-space-type/README.md) | Approved | A SpaceTypeSystem space whose module-record writes are open to any authenticated user (public reads), branched in the spaceus access check, with per-record mutation authorization delegated to the owning extension. |
| [reserved-extension-space-ids](reserved-extension-space-ids/README.md) | Draft | A `$`-prefixed, well-known space-id convention for an extension's reserved SpaceTypeSystem space (`$invitus`, `$gameboard`): `$` reserved to the platform, `$`-prefix implies System type, id derived from the extension id with no lookup. Resolves the addressing open question from system-space-type. |

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/features-index-specification*
