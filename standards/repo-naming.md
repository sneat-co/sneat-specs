# Repository Naming

How Sneat repositories are named, so the org stays navigable and extensions are
easy to find.

## Taxonomy

| Family | Pattern | Examples |
| --- | --- | --- |
| **Platform / infra** | `sneat-*` | `sneat-go`, `sneat-libs`, `sneat-specs`, `sneat-core-modules`, `sneat-ext-template` |
| **Extension - product / implementation / app** | `<id>` | `listus`, `contactus`, `gameboard`, `assetus` |
| **Extension - public definition** | `ext-<id>` | `ext-contactus`, `ext-gameboard`, `ext-yardius` |

A Sneat extension normally has two repos:

- **`<id>`** - the product / implementation / app repo: the deployable app,
  implementation backend, implementation frontend libraries, and product-specific
  internals. This repo is usually private. `listus` is the standing exception:
  its implementation repo is public.
- **`ext-<id>`** - the public extension-definition repo: the stable cross-repo
  boundary other repos import. It defines the extension's public wire shape,
  shared DTOs, facade interfaces, consts, and frontend contract tokens/types.

Repo names and package names intentionally diverge here:

- repo: `ext-<id>`
- frontend package: `@sneat/extension-<id>-contract`

The repo name groups public extension-definition repos in org and directory
listings. The package name stays on the established `extension-*-contract`
namespace so existing consumers and import paths stay stable.

The `<id>` repo may also be a standalone product or app. It is not the public
definition of the extension. The `ext-<id>` repo is the durable public boundary.

### Why `ext-<id>`

The extension-definition repos are public and numerous. A prefix groups them in
GitHub org lists, local directories, and editor trees:

```
ext-bookius
ext-contactus
ext-gameboard
ext-kids-club
ext-schoolus
ext-sizeus
ext-sneat-team
ext-yardius
```

The older suffix forms (`contactus-ext`, `gameboard-ext`, `sneat-team-ext`) and
the newer `<id>-contract` form (`yardius-contract`, `sizeus-contract`, ...)
represent the same role, but they do not group together and they split the
convention. New extension-definition repos use `ext-<id>`.

Renaming existing repos is a coordinated module/package migration - see the
[migration runbook](./extension-definition-repo-rename-migration.md).

### Why no `ext-` prefix on implementation repos

Implementation repos are named products, not anonymous plugins: `listus`,
`contactus`, `calendarius`, `gameboard`, `sneat-team`. The implementation repo
keeps the clean product name so it aligns with the app, domain, and package
where appropriate (`listus` -> `listus.app` -> `@sneat/listus`).

`-us` is the default, not a rule. Some extensions are deliberately branded
otherwise — `gameboard`, `rsvp-express`, `sneat-team` — and are left as-is; brand
intent wins over mechanical consistency.

## Extension-definition repo layout

Every `ext-<id>` repo uses the same top-level shape. A directory can be present
as a scaffold even before it has much content; consumers should know where to
look.

| Directory | Purpose |
| --- | --- |
| `typespec/` | Public wire shape and endpoint definitions. |
| `backend/` | Go DTOs, consts, facades, and model shapes needed by backend consumers. |
| `frontend/` | TypeScript/Angular contract package: DTOs, tokens, consts, and facade types needed by frontend consumers. |

The package published from `frontend/` should normally be named
`@sneat/extension-<id>-contract`.

Allowed supporting files/directories include `README.md`, `specscore.yaml`,
`.github/`, `scripts/`, and `parity/`. Implementation code, deployable apps, and
private service internals stay in the `<id>` repo.

## Discoverability — GitHub topics

Repo names provide first-pass grouping. GitHub topics provide the canonical
machine-readable registry:

| Topic | Applied to |
| --- | --- |
| `sneat-extension` | every extension product / implementation / app repo |
| `sneat-extension-definition` | every public `ext-<id>` definition repo |

Use both topics while migrating:

- `sneat-extension-contract` - legacy topic retained until all repos and docs
  move to `sneat-extension-definition`.
- `sneat-extension-definition` - target topic for all `ext-<id>` repos.

Find them all: `github.com/sneat-co?q=topic:sneat-extension`.

## Registry

First-party Sneat extensions and their repos. Keep this in sync when adding an
extension.

| Extension | Product / implementation repo | Extension-definition repo | Notes |
| --- | --- | --- | --- |
| assetus | `assetus` | — | |
| bookius | `bookius` | `bookius-contract` -> `ext-bookius` | pending rename |
| budgetus | `budgetus` | — | |
| calendarius | `calendarius` | — | backend in `calendarius`; frontend currently in `sneat-libs` (migration in progress) |
| contactus | `contactus` | `contactus-ext` -> `ext-contactus` | pending rename |
| debtus / splitus | `debtus` | — | |
| docus | `docus` | — | |
| eventus | `eventus` | — | |
| formius | `formius` | — | reusable personal-profile platform (`formius.app`); renamed from `fillless` 2026-07-06; frontend-only, no Go backend |
| gameboard | `gameboard` | `gameboard-ext` -> `ext-gameboard` | pending rename |
| kids-club | `kids-club` | `kids-club-contract` -> `ext-kids-club` | pending rename |
| listus | `listus` | — | |
| logistus | `logistus` | — | `logist-apps` holds the deployable app(s) |
| rsvp-express | `rsvp-express` | — | |
| schoolus | `schoolus` | `schoolus-contract` -> `ext-schoolus` | pending rename |
| sneat-team | `sneat-team` | `sneat-team-ext` -> `ext-sneat-team` | pending rename |
| sizeus | `sizeus` | `sizeus-contract` -> `ext-sizeus` | pending rename |
| trackus | `trackus` | — | |
| yardius | `yardius` | `yardius-contract` -> `ext-yardius` | pending rename |

Rows marked "pending rename" keep their current repo name until the module-path
and package migration runs. Verify this table against the live org before relying
on it.

## Related

- [Sneat extension standards](https://github.com/sneat-co/sneat-libs/blob/main/docs/extension-standards/README.md)
  — backend wiring, frontend apps, UX.
- [`extension-contract-repo`](https://github.com/sneat-co/sneat-libs/blob/main/spec/features/extension-contract-repo/README.md)
  — legacy name for what the `ext-<id>` definition repo contains and why it is
  frozen.
