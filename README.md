# sneat-specs

Public specifications and standards for the Sneat platform (SpecScore-managed).

## 🧭 Start here: [Core Entities of the Sneat Platform](standards/core-entities.md)

The foundational reference every extension builds on — User, Space (including
virtual entity-scoped spaces), Contact (a **place is a location contact**),
Venue as a composition, Happening, the occasion family and its shared
participation vocabulary, Linkage, and the cross-cutting patterns
(ports & adapters, contract modules, attend-not-join, reuse-before-build).
Read it before designing a new extension or a new entity.

## Contents

| Directory | Purpose |
|---|---|
| [`spec/`](spec/README.md) | SpecScore-formatted specifications — `features/`, `ideas/`, `plans/`, `decisions/`. |
| [`standards/`](standards/README.md) | House standards & conventions that span repos (not SpecScore-gated). |

## Standards

- [**Core entities & their relations**](standards/core-entities.md) — the
  foundational entity map and composition patterns (start here).
- [Extension backend architecture](standards/extension-backend-architecture.md) —
  ports & adapters; dalgo-only extension backends.
- [Repo naming](standards/repo-naming.md) — products, `ext-<id>` contract
  repos, discovery topics.
- [Frontend UX](standards/frontend-ux/README.md) — cards, buttons, lists, page
  layout, forms, modals, and loading/empty/error states for Sneat extensions.
