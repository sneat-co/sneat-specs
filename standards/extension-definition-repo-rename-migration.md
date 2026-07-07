# Migration runbook: extension-definition repos -> `ext-<id>`

A step-by-step plan to converge public extension-definition repos on the
[`ext-<id>` standard](./repo-naming.md). Not yet executed - this is the runbook
to decide and schedule against.

> **TL;DR.** This is a repo rename, Go module-path migration, and frontend layout
> cleanup. GitHub redirects handle git/web, but Go does not redirect module paths.
> Every importer's source and `go.mod` must switch. The hazard is the
> duplicate-module trap: importing both old and new module paths makes Go treat
> identical types as different types.

## Target

| Current repo | Target repo | Current Go module path | Target Go module path | Difficulty |
| --- | --- | --- | --- | --- |
| `gameboard-ext` | `ext-gameboard` | `github.com/sneat-co/gameboard-ext/backend` | `github.com/sneat-co/ext-gameboard/backend` | Low pilot |
| `sneat-team-ext` | `ext-sneat-team` | `github.com/sneat-co/sneat-team-ext/backend` | `github.com/sneat-co/ext-sneat-team/backend` | Low pilot |
| `contactus-ext` | `ext-contactus` | `github.com/sneat-co/contactus-ext/backend` | `github.com/sneat-co/ext-contactus/backend` | High |
| `bookius-contract` | `ext-bookius` | `github.com/sneat-co/bookius-contract/backend` | `github.com/sneat-co/ext-bookius/backend` | Medium |
| `kids-club-contract` | `ext-kids-club` | `github.com/sneat-co/kids-club-contract/backend` | `github.com/sneat-co/ext-kids-club/backend` | Medium |
| `schoolus-contract` | `ext-schoolus` | `github.com/sneat-co/schoolus-contract/backend` | `github.com/sneat-co/ext-schoolus/backend` | Medium |
| `sizeus-contract` | `ext-sizeus` | none today | none unless Go consumers are added | Low |
| `yardius-contract` | `ext-yardius` | none today | none unless Go consumers are added | Low |

## Target layout

Every target repo should use the same top-level structure:

```text
ext-<id>/
├── typespec/
├── backend/
└── frontend/
```

- `typespec/` holds the public wire shape and endpoint definitions.
- `backend/` holds Go DTOs, consts, facades, and model shapes needed by backend
  consumers.
- `frontend/` holds TypeScript/Angular DTOs, tokens, consts, and facade types
  needed by frontend consumers.

Existing repos that use root-level `src/` should move that package under
`frontend/`. Existing repos that already use `frontend/` should keep it.

## The duplicate-module trap

Go identifies a module by the path in its `go.mod`. If repo **A** has migrated to
`ext-contactus` while repo **B** still exposes types from `contactus-ext`, the
build can pull both modules:

```text
github.com/sneat-co/contactus-ext/backend
github.com/sneat-co/ext-contactus/backend
```

Those two modules may contain identical code, but Go treats their exported types
as different types. Any value crossing that boundary can fail to compile.

`contactus` is the risky case because contactus models are re-exported through
shared hubs and consumed by multiple repos. Contactus importers must migrate in
dependency order; do not merge a higher-level repo while one of its dependencies
still exposes the old contactus module path.

`gameboard-ext` and `sneat-team-ext` are smaller pilots because their contract
types do not transit a shared hub in the same way.

## Per-repo procedure

### Phase A - extension-definition repo

1. Rename the repo on GitHub:
   `gh repo rename ext-<id> -R sneat-co/<current-name>`
2. Update the local directory name:
   `mv <current-name> ext-<id>`
3. Update the git remote:
   `git remote set-url origin https://github.com/sneat-co/ext-<id>.git`
4. Update `backend/go.mod`, if present:
   `module github.com/sneat-co/ext-<id>/backend`
5. Rewrite internal Go self-imports from the old module path to the new module
   path.
6. Move root-level frontend package files into `frontend/` if the repo currently
   uses root `src/`, `package.json`, `tsconfig*.json`, or `pnpm*.yaml`.
7. Keep frontend/npm package names on the existing namespace:
   `@sneat/extension-<id>-contract`.
8. Update README, CI, `package.json` repository metadata, TypeSpec package names,
   and SpecScore metadata.
9. Add the `sneat-extension-definition` GitHub topic. Keep
   `sneat-extension-contract` during migration if consumers still depend on it.
10. Run the repo's checks:
   `go test ./...` in `backend/` if present, and the frontend package's build/test
   commands if present.
11. Commit and tag a release for Go modules, using the existing module tag scheme
    where applicable, so importers can require a concrete version.

### Phase B - importer repos

For each importer, in dependency order:

1. Rewrite Go imports from the old backend module path to
   `github.com/sneat-co/ext-<id>/backend`.
2. Update `go.mod` to require the new module path and the tag from Phase A.
3. Remove any old `require` or `replace` entry for the previous module path.
4. Run `go mod tidy`, `go build ./...`, and `go test ./...`.
5. Update frontend package references if local paths changed. Package names stay
   on `@sneat/extension-<id>-contract`.
6. Commit on a branch, open a PR, and require CI green before merging.

### Phase C - close-out

- Grep the workspace/org for old repo and module names.
- Update [repo-naming.md](./repo-naming.md#registry) from pending rename to the
  final `ext-<id>` repo name.
- Remove the legacy `sneat-extension-contract` topic after all repos and docs have
  moved to `sneat-extension-definition`.
- Confirm package registries, Go module tags, CI badges, and READMEs point at the
  new repo.

## Recommended sequencing

1. `gameboard-ext` -> `ext-gameboard`: smallest isolated pilot.
2. `sneat-team-ext` -> `ext-sneat-team`: same shape as the first pilot.
3. TS-first repos: `yardius-contract` -> `ext-yardius`, then
   `sizeus-contract` -> `ext-sizeus`.
4. Newer medium repos: `kids-club-contract`, `schoolus-contract`,
   `bookius-contract`.
5. `contactus-ext` -> `ext-contactus`: schedule as its own coordinated migration.
   Suggested dependency order: `ext-contactus` tag -> `contactus` ->
   `sneat-core-modules` -> `debtus`, `logistus`, `sneat-go-backend` ->
   `sneat-go`. Include `sneat-bots` only if it remains an active importer.

## Rollback

- Per importer: revert the importer PR atomically, including source imports and
  `go.mod` / `go.sum` changes.
- GitHub repo rename is reversible only before importer PRs have merged. Once
  importers consume `ext-<id>`, prefer rolling forward.

## Effort estimate

- Low pilots: about half a day each including CI.
- TS-first repos: a few hours each if there are no external package consumers.
- Medium repos with Go modules: about half a day each.
- `contactus`: multi-day, multi-PR effort because of dependency ordering and
  shared type exposure.
