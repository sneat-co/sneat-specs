# Extension Backend Architecture — ports & adapters

**Status:** Recommended for all new extension backends · **Reference implementation:** `eventius/backend` + `sneat-go/pkg/modules/eventius/`
**Origin:** pattern emerged during the eventius extraction (2026-07); named and standardized during the ecosystem review 2026-07-09 (`backstage/spec/research/ecosystem-review-2026-07/`).

## The rule

An extension's backend domain module depends on **`dal-go/dalgo` only**. It must not import `sneat-go-core`, `sneat-core-modules`, or another extension's backend. Anything it needs from the platform or from sibling extensions is expressed as a **port** — a small Go interface defined *in the extension module* — and satisfied by an **adapter** that lives in the host composition root (`sneat-go`), where all the facades are already available.

```
eventius/backend (module: github.com/sneat-co/eventius/backend)
  go.mod: require github.com/dal-go/dalgo   ← the ONLY dependency
  eventius/ports.go:
      type HappeningCreator interface {
          CreateHappening(ctx, userID, spaceID string, spec HappeningSpec) (string, error)
      }

sneat-go/pkg/modules/eventius/adapters.go   ← the host's half of the boundary
  type happeningCreatorAdapter struct{}      // translates HappeningSpec →
  // dto4calendarius.CreateHappeningRequest and calls facade4calendarius.CreateHappening
  // (unit-tested in adapters_test.go)
```

## Why

1. **No version treadmill.** Extension backends that import `sneat-core-modules` directly lag the kernel by up to ~20 minor versions (observed across listus/assetus, 2026-07). A dalgo-only module has nothing to lag behind.
2. **Independent releases.** Tag `backend/vX.Y.Z` in the product repo; `sneat-go` consumes the tag. Kernel changes never force an extension re-release.
3. **Trivial testability.** Domain tests fake the port; no Firestore emulator, no platform bootstrapping.
4. **No public/private CI friction.** A dalgo-only module builds in any CI without GOPRIVATE/PAT (see the public-only constraint documented in `sneat-go-backend/pkg/extensions/standard_extensions.go`).
5. **Swappable implementations.** If the backing capability moves (e.g. calendar records to an OVDB-backed store), only the adapter in `sneat-go` changes.

## How to apply it

- **New extension backend:** start from `eventius/backend`'s shape, not from a direct-facade-import backend (e.g. `requoter/backend`, which composes `facade4contactus`/`facade4assetus`/`facade4calendarius` directly — the legacy style).
- **Cross-extension needs:** define a port per use-case (creator, notifier, resolver), keep it minimal (one or two methods), and pass primitives + the extension's own spec types across it. Never leak another extension's DBO/DTO types into the module.
- **Wiring:** the adapter is registered where the extension is composed — `sneat-go/pkg/sneatmain/sneat_main.go` / the extension's `module.go` under `sneat-go/pkg/modules/<id>/`.
- **Existing backends:** no forced migration; converge opportunistically when a backend needs a breaking kernel bump anyway.

## Doesn't this duplicate DBOs across extensions?

No — and if it starts to, the boundary is drawn wrong. What crosses a port is a
**caller-owned spec type** (e.g. `eventiuslib.HappeningSpec`: start, duration,
location), not the provider's DBO. The adapter translates spec → the provider's
request DTO. Duplicating *small spec types* across extensions is acceptable and
often desirable (each extension states only what it needs); needing a provider's
*full* DBO means either the two extensions are really one domain, or the type
belongs in a contract module (below).

## Where shared things go — the decision ladder

| What is shared | Where it lives | Example |
|---|---|---|
| Behavior (writes, use-cases), 1–2 consumers | Port in the consumer + adapter in `sneat-go` | eventius → calendarius happening creation |
| Types/read-models needed by several extensions | **`ext-<id>/backend` contract module** (dalgo-only, publicly buildable, versioned independently) | `ext-contactus/backend` — imported by `sneat-core-modules` and `requoter`; if happening types go wide, create `ext-calendarius/backend` |
| Behavior needed by many consumers | Facade *interface* in the `ext-<id>/backend` contract module; implemented once, injected by the host | (when a capability's ports proliferate past ~3 adapters) |
| Domain-free primitives only | `sneat-go-core` (`coretypes`, `facade`, validation glue) | — |

**Do not move domain types into `sneat-go-core`.** The kernel's fan-in is ~100+
packages; a domain type there turns every core release into a domain release and
recreates the version treadmill this standard exists to escape. The backend
contract module is the kernel-adjacent home — it is the same dependency-inversion
move as the frontend `@sneat/extension-<id>-contract` libraries.

## Boundary with other standards

- Repo/package naming: [repo-naming.md](repo-naming.md) (`ext-<id>` definition repos; `backend/vX.Y.Z` tags in product repos).
- Frontend counterpart: the contract/shared/internal triad (`sneat-libs/docs/extension-standards/`) — the same dependency-inversion idea; contract libs are the frontend's ports.
