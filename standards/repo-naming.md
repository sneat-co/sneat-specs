# Repository Naming

How Sneat repositories are named, so the org stays navigable and extensions are
easy to find.

## Taxonomy

| Family | Pattern | Examples |
| --- | --- | --- |
| **Platform / infra** | `sneat-*` | `sneat-go`, `sneat-libs`, `sneat-specs`, `sneat-core-modules`, `sneat-ext-template` |
| **Extension — implementation** | `<id>` | `listus`, `contactus`, `gameboard`, `assetus` |
| **Extension — contract** | `<id>-contract` | `listus-contract`, `contactus-contract` |

A **Sneat extension** is a self-contained vertical. Each extension is **two
repos**:

- **`<id>`** — the implementation: the Go backend module, the Angular/Ionic
  frontend libraries, and the standalone app.
- **`<id>-contract`** — the frozen cross-repo **contract surface**: the TypeSpec
  definitions plus the shared Go models / DTOs / consts / briefs that other repos
  import.

### Why `-contract`, not `-ext`

The whole vertical *is* an extension, so an `-ext` suffix on the second repo
doesn't say what makes it different. That repo is specifically the **contract** —
the stable interface other repos depend on. `<id>-contract` names the thing.

> **Legacy:** the existing contract repos still use the old `-ext` suffix
> (`contactus-ext`, `gameboard-ext`, `sneat-team-ext`). Renaming them is a
> coordinated Go-module migration (their `github.com/sneat-co/<id>-ext/backend`
> module path is imported by ~118 files across ~9 repos) — see the
> [migration runbook](./contract-repo-rename-migration.md). New contract repos use
> `-contract` from the start.

### Why no `ext-` prefix on implementation repos

Extensions are **named products**, not anonymous plugins — `listus`, `contactus`,
`calendarius`. The `-us` suffix is the house brand signature; the implementation
repo keeps the clean product name so it aligns with the app, domain, and package
(`listus` → `listus.app` → `@sneat/listus`). An `ext-` prefix was **considered and
rejected** (2026-06-28): it would bury the distinctive `-us` mark under a generic
label, split the product name from the repo name, and add nothing users ever see.
Platform-level grouping is handled by **topics** (below), not by repo names.

`-us` is the **default, not a rule**. Some extensions are deliberately branded
otherwise — `gameboard`, `rsvp-express`, `sneat-team` — and are left as-is; brand
intent wins over mechanical consistency.

## Discoverability — GitHub topics

Repo names don't group in the org list, so finding "all the extensions" relies on
**topics**, not naming:

| Topic | Applied to |
| --- | --- |
| `sneat-extension` | every extension **implementation** repo |
| `sneat-extension-contract` | every extension **contract** repo |

Find them all: **`github.com/sneat-co?q=topic:sneat-extension`**.

Topics are the canonical grouping mechanism — apply them to every extension repo
regardless of its name, so discoverability never depends on a rename.

## Registry

First-party Sneat extensions and their repos. Keep this in sync when adding an
extension.

| Extension | Implementation repo | Contract repo | Notes |
| --- | --- | --- | --- |
| assetus | `assetus` | — | |
| budgetus | `budgetus` | — | |
| calendarius | `calendarius` | — | backend in `calendarius`; frontend currently in `sneat-libs` (migration in progress) |
| contactus | `contactus` | `contactus-ext` → `contactus-contract` | contract repo pending rename |
| debtus / splitus | `debtus` | — | |
| docus | `docus` | — | |
| eventus | `eventus` | — | |
| formius | `formius` | — | reusable personal-profile platform (`formius.app`); renamed from `fillless` 2026-07-06; frontend-only, no Go backend |
| gameboard | `gameboard` | `gameboard-ext` → `gameboard-contract` | contract repo pending rename |
| kids-club | `kids-club` | `kids-club-contract` | first `-contract`-convention repo; impl repo private |
| listus | `listus` | — | |
| logistus | `logistus` | — | `logist-apps` holds the deployable app(s) |
| rsvp-express | `rsvp-express` | — | |
| sneat-team | `sneat-team` | `sneat-team-ext` → `sneat-team-contract` | contract repo pending rename |
| trackus | `trackus` | — | |
| yardius | `yardius` | `yardius-contract` | |

> Rows marked "pending rename" keep their `-ext` repo until the module-path
> migration runs. Verify this table against the live org before relying on it.

## Related

- [Sneat extension standards](https://github.com/sneat-co/sneat-libs/blob/main/docs/extension-standards/README.md)
  — backend wiring, frontend apps, UX.
- [`extension-contract-repo`](https://github.com/sneat-co/sneat-libs/blob/main/spec/features/extension-contract-repo/README.md)
  — what the contract repo contains and why it's frozen.
