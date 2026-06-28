# Migration runbook: `<id>-ext` → `<id>-contract`

A step-by-step plan to rename the legacy contract repos to the
[`<id>-contract` standard](./repo-naming.md). **Not yet executed** — this is the
runbook to decide/schedule against.

> **TL;DR.** The rename is a Go **module-path** migration
> (`github.com/sneat-co/<id>-ext/backend` → `…/<id>-contract/backend`). GitHub's
> repo redirect handles git/web, but Go does **not** redirect module paths — every
> importer's source *and* `go.mod` must switch. The hazard isn't the edit count;
> it's the **duplicate-module trap** (below). Do the two small repos first as
> pilots; treat `contactus-ext` as a separate, carefully-ordered project.

## Scope

| Contract repo | Module path | External importer repos | Difficulty |
| --- | --- | --- | --- |
| `gameboard-ext` | `…/gameboard-ext/backend` | `gameboard`, `sneat-go` | **Low** (pilot) |
| `sneat-team-ext` | `…/sneat-team-ext/backend` | `sneat-team`, `sneat-go` | **Low** (pilot) |
| `contactus-ext` | `…/contactus-ext/backend` | `contactus`, `debtus`, `logistus`, `sneat-bots`, `sneat-core-modules`, `sneat-go-backend`, `sneat-go` | **High** |

Frontend/TypeSpec impact is negligible (no meaningful non-Go references found).
The contract repos also carry `frontend/` and `typespec/` dirs — check their
`package.json` `repository` field and any CI/submodule references, but expect no
import-path churn there.

## The duplicate-module trap (read first)

Go identifies a module by the path in its `go.mod`. If repo **A** has migrated to
`contactus-contract` while repo **B** (which A depends on) still exposes types
from `contactus-ext`, the build pulls **both** modules — and
`contactus-ext.Contact` ≠ `contactus-contract.Contact` even though the code is
identical. Any value crossing the A↔B boundary fails to compile.

`contactus` models are re-exported through `sneat-core-modules`
(userus/spaceus/invitus/linkage/auth) and consumed by `sneat-go`,
`sneat-go-backend`, and `sneat-bots`. So contactus importers **cannot** migrate
independently — they migrate **bottom-up in dependency order**, and a repo isn't
done until all of its dependencies that expose contactus types are done.

`gameboard-ext` and `sneat-team-ext` don't have this problem: their types don't
transit a shared hub, so their 2 importers can go in a single coordinated step.

## Per-contract procedure

### Phase A — the contract repo itself

1. **Rename on GitHub:** `gh repo rename <id>-contract -R sneat-co/<id>-ext`
   (creates a redirect for git remotes; topics are preserved).
2. **Update its own module path:** in `<id>-contract/backend/go.mod` set
   `module github.com/sneat-co/<id>-contract/backend`; rewrite internal
   self-imports:
   `grep -rl 'sneat-co/<id>-ext/backend' | xargs sed -i '' 's#sneat-co/<id>-ext/backend#sneat-co/<id>-contract/backend#g'`
3. `go build ./... && go test ./...` inside the contract repo.
4. Commit and **tag a release** (e.g. `git tag backend/vX.Y.Z` per the existing
   tagging scheme) so importers can require a concrete version.
5. Update local git remote: `git remote set-url origin …/<id>-contract.git`.

### Phase B — each importer repo (in dependency order)

For each importer, from leaves up to `sneat-go` last:

1. **Rewrite imports:**
   `grep -rl 'sneat-co/<id>-ext/backend' --include='*.go' | xargs sed -i '' 's#sneat-co/<id>-ext/backend#sneat-co/<id>-contract/backend#g'`
2. **Update `go.mod`:** replace the `require github.com/sneat-co/<id>-ext/backend …`
   line with the new path + the tag from Phase A.4. Optionally add a temporary
   `replace … => ../<id>-contract/backend` for local cross-repo dev; **remove it
   before merge**.
3. `go mod tidy && go build ./... && go test ./...`.
4. Commit on a branch, open a PR, require **CI green** before merge.
5. Merge, then move to the next importer up the chain.

### Phase C — close-out

- Grep the whole org for stragglers:
  `grep -rE 'sneat-co/<id>-ext' .` should return nothing but historical changelogs.
- Update the [registry](./repo-naming.md#registry): move the repo from the
  "→ pending rename" column to its final `<id>-contract` name.
- Confirm the `sneat-extension-contract` topic survived the rename (it should).

## Recommended sequencing

1. **`gameboard-ext` first** — smallest, isolated. Validates the whole procedure
   (rename → tag → 2 importers → CI). Treat as the pilot; capture any surprises.
2. **`sneat-team-ext`** — same shape, apply lessons.
3. **`contactus-ext`** — only after the pilots. Plan it as its own effort with a
   written dependency order: `contactus-contract` (tag) → `contactus`,
   `sneat-core-modules` (the hub) → `debtus`, `logistus`, `sneat-bots`,
   `sneat-go-backend` → `sneat-go`. Land each as a CI-gated PR; do **not** merge a
   higher-level repo until every dependency that re-exports contactus types is
   merged.

## Rollback

- Per importer: revert the PR (the renamed module still resolves; reverting just
  points back at the redirected old path *if* you also revert that importer's
  go.mod — keep revert atomic per repo).
- The GitHub rename is reversible (`gh repo rename <id>-ext`), but only do this if
  no importer has merged yet; once importers are on `-contract`, roll forward
  rather than back.

## Effort estimate

- Pilots (`gameboard-ext`, `sneat-team-ext`): ~½ day each incl. CI.
- `contactus-ext`: a multi-day, multi-PR effort across 7 repos with careful
  ordering. The naming win is cosmetic (topics already solve discoverability), so
  weigh whether the churn is worth it or whether to leave `contactus-ext` as the
  one grandfathered exception.
