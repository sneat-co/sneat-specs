# Extension Backend Architecture — ports & adapters

**Status:** Required target for extension extraction · **Reference implementation:** `togethered/backend` + `sneat-bots/extensions/togethered` + `sneat-go/pkg/modules/togethered/module.go`
**Origin:** pattern emerged during the eventius extraction (2026-07); named and standardized during the ecosystem review 2026-07-09 (`backstage/spec/research/ecosystem-review-2026-07/`).

## The rule

All extension implementation belongs outside `sneat-go`. The extension's core
backend packages define domain/application behavior and the ports they need.
Concrete platform adapters and cross-extension adapters built against public
contracts live in the extension implementation repository, in an explicitly
named integration package. Bot delivery lives in
`sneat-bots/extensions/<id>` while bot dependencies are being maintained
centrally.

> **Release-blocking dependency rule:** an extension implementation repository
> MUST NOT import another extension implementation module. A path such as
> `github.com/sneat-co/<provider>/backend/...` from a different extension is
> forbidden, including from an `integration` package. Depend on the provider's
> `github.com/sneat-co/ext-<provider>/backend` contract module, or define a
> consumer-owned port and inject it. Only the executable composition root may
> import both concrete implementations.

`sneat-go` is the executable composition root only: construct/configure
integrations, register routes/delayers/bot profiles, and translate the final
host callback types. A package under `sneat-go/pkg/modules/<id>` must not contain
domain decisions, persistence algorithms, command handlers, presentation, or a
reusable adapter implementation.

The dependency-inversion rule applies to the **whole extension implementation
module**. Core packages should depend on `dalgo` and small generic libraries
and express external needs as extension-owned ports. A separate integration
package may import platform and extension **contract modules** to implement
those ports, but it must not import a sibling implementation backend. Core
packages must never import that integration package.

```
togethered/backend (module: github.com/sneat-co/togethered/backend)
  integration4togd/cross_extension.go:       ← consumer-owned port
      type OutingPayments interface {
          CommitToPayShare(ctx, occasionID, userID string) error
      }

togethered/backend/integration4togd/        ← concrete adapters
  // calls ext-bookius/ext-calendarius contracts and the injected payment port

sneat-bots/extensions/togethered/           ← bot commands and presentation

sneat-go/pkg/modules/togethered/module.go    ← composition only
  // supplies host callbacks and registers the integration
```

## Why

1. **A visible dependency boundary.** An extension implementation cannot reach
   into a sibling implementation. Contract dependency updates are isolated in
   one integration package instead of spreading through the extension.
2. **Independent releases.** Tag `backend/vX.Y.Z` in the product repo;
   `sneat-go` and `sneat-bots` consume the tag. Host refactors do not require
   copying extension logic back into the executable repository.
3. **Trivial testability.** Domain tests fake the port; no Firestore emulator, no platform bootstrapping.
4. **Focused tests.** Core packages test against fake ports; integration
   packages test contract-backed adapters without linking sibling
   implementations.
5. **Swappable implementations.** If the backing capability moves (for example
   calendar records to an OVDB-backed store), only the extension's integration
   adapter changes.

## How to apply it

- **New extension backend:** put domain/application packages and their ports
  under `<id>/backend`; put concrete platform adapters and adapters that consume
  public extension contracts in an explicit integration package in the same
  implementation repo.
- **Cross-extension needs:** depend on the provider's `ext-<id>/backend`
  contract, or define a consumer-owned port per use-case (creator, notifier,
  resolver). Keep it minimal and pass primitives plus contract/caller-owned spec
  types across it. Never import the provider implementation or leak its private
  DBO/DTO types into the consumer module.
- **Bot delivery:** keep command registration, handlers, callbacks,
  presentation, and conversational bindings under
  `sneat-bots/extensions/<id>`. They call extension backend APIs; they do not
  define business rules and never import `sneat-go`.
- **Wiring:** `sneat-go/pkg/modules/<id>/module.go` may import the backend,
  integration, and bot packages and bind host callbacks. Keep this package
  deliberately small.
- **Existing backends:** extract one extension at a time using the checklist
  below; do not leave compatibility copies in `sneat-go` after consumers move.

## Repeatable extraction checklist

1. Inventory every package and file under `sneat-go` owned by the extension,
   including bot profiles, delayed handlers, cron functions, notification
   renderers, and tests.
2. Classify each item as core domain/application, concrete integration adapter,
   bot delivery, or executable composition.
3. Move core and non-bot integration code to `<id>/backend`; introduce narrow
   ports/callbacks anywhere the moved code would otherwise import `sneat-go`.
   Do not "solve" Sneat-Go ownership by moving a forbidden sibling-backend
   dependency into the extension implementation repository.
4. Move bot-owned code as one directory to `sneat-bots/extensions/<id>` and
   remove every `sneat-go` import from it.
5. Reduce `sneat-go/pkg/modules/<id>` to registration/configuration and narrow
   host type translations. Cron registration in `pkg/sneatmain` should point to
   the implementation package.
6. Test the backend, bot repository, and executable together in a temporary Go
   workspace. Then publish in dependency order: backend tag, bot tag, Sneat-Go
   version update.
7. Merge each repository through a PR only after its CI passes; delete the old
   Sneat-Go packages and update architecture/readme ownership links in the same
   extraction.

Before publishing, scan the resulting backend's imports. Every import of a
different Sneat extension's implementation module is a release blocker:

```sh
rg 'github\.com/sneat-co/[^/]+/backend' <id>/backend
```

Expected cross-extension matches use `github.com/sneat-co/ext-<id>/backend`.
Matches for a sibling product implementation (for example
`github.com/sneat-co/paymentus/backend`) must be replaced with a contract or an
injected consumer-owned port. Moving the import into a package named
`integration` does not make it acceptable.

Make this scan a required CI job for every extracted implementation backend.
Togethered's `.github/workflows/backend-ci.yml` is the reference: it excludes
the module's own path and `ext-*` contract paths, then fails on every remaining
Sneat product-backend import before lint/test/build can publish a tag.

### Composition-root size check

Treat growth in `sneat-go/pkg/modules/<id>` as a design alarm. Normal contents
are a `module.go`, small callback translations, and focused wiring tests. If a
function can be unit-tested without starting the executable host, it probably
belongs in the extension implementation or bot repository instead.

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
| Behavior (writes, use-cases), 1–2 consumers | Port in the consumer + adapter that calls an injected provider contract; the composition root supplies the concrete provider | eventius → calendarius happening creation |
| Types/read-models needed by several extensions | **`ext-<id>/backend` contract module** (dalgo-only, publicly buildable, versioned independently) | `ext-contactus/backend` — imported by `sneat-core-modules` and `requoter`; if happening types go wide, create `ext-calendarius/backend` |
| Behavior needed by many consumers | Facade *interface* in the `ext-<id>/backend` contract module; implemented once, injected by the host | (when a capability's ports proliferate past ~3 adapters) |
| Domain-free primitives only | `sneat-go-core` (`coretypes`, `facade`, validation glue) | — |

### What moves to `ext-<id>/backend` vs what stays private

- **Moves (shared vocabulary):** `dto4<id>` request/response types, brief/read
  models, consts, and facade *interfaces*.
- **Stays in `<id>/backend` (implementation):** `dbo4<id>` storage schemas,
  `dal4<id>`, `api4<id>`, facade *implementations*. Another extension needing
  your DBO is a boundary smell — offer a brief or a port instead.
- **Mandate:** per [repo-naming.md](repo-naming.md), *every* extension has an
  `ext-<id>` definition repo carrying both frontend and backend contracts. This
  section only clarifies which backend packages belong there.
- **Reference:** `ext-contactus/backend` (`contactusmodels` + `facade4contactus`)
  is the completed shape.
- **Known compliance gaps (2026-07-09):** ~~`ext-calendarius` missing entirely~~
  → **created 2026-07-09** ([sneat-co/ext-calendarius](https://github.com/sneat-co/ext-calendarius),
  `backend/v0.0.2`: shared models plus happening/recurring facade contracts;
  sneat-go's eventius adapter still to be repointed per its README migration plan);
  ext-sizeus/ext-yardius lack backend contracts; ext-kids-club/ext-schoolus lack
  frontend contracts; several contracts are duplicated between `ext-<id>` and
  `<id>` repos with diverging versions (bookius, gameboard, sizeus) — the
  `ext-<id>` copy is canonical, delete the in-product fork when touched.

**Do not move domain types into `sneat-go-core`.** The kernel's fan-in is ~100+
packages; a domain type there turns every core release into a domain release and
recreates the version treadmill this standard exists to escape. The backend
contract module is the kernel-adjacent home — it is the same dependency-inversion
move as the frontend `@sneat/extension-<id>-contract` libraries.

## Boundary with other standards

- Repo/package naming: [repo-naming.md](repo-naming.md) (`ext-<id>` definition repos; `backend/vX.Y.Z` tags in product repos).
- Frontend counterpart: the contract/shared/internal triad (`sneat-libs/docs/extension-standards/`) — the same dependency-inversion idea; contract libs are the frontend's ports.
