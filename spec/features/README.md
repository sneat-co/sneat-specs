---
format: https://specscore.md/features-index-specification
---

# Features

Feature specifications for this project.

## Index

| Feature | Status | Description |
|---------|--------|-------------|
| [system-space-type](system-space-type/README.md) | Approved | A SpaceTypeSystem space whose module-record writes are open to any authenticated user (public reads), branched in the spaceus access check, with per-record mutation authorization delegated to the owning extension. |
| [reserved-extension-space-ids](reserved-extension-space-ids/README.md) | Draft | A spaceless system namespace for an extension's global, cross-user records: they live at `/ext/{ext-id}/...` (not in any space), referenced by a suffix-less `{ext-id}/{collection}/{doc-id}` ref, with the linkage validator/resolver gaining a spaceless branch and the SpaceTypeSystem ACL lifted to the namespace. Resolves the addressing open question from system-space-type. |

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/features-index-specification*
