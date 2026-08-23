---
format: https://specscore.md/decision-specification
status: In Review
---
# Decision: Unified space registration — generic space types with module markers

**Status:** In Review
**Date:** 2026-08-23
**Owner:** alex
**Tags:** spaces, extensions, registration, vendors, authorization, space-types
**Source Idea:** —
**Supersedes:** —
**Superseded By:** —

## Context

### Goal

**Four public surfaces — school-portal.app, gametable.space, noticeboard.cc and
sneat.club — offer vendor registration through one reusable approach that stays
domain-specific in its details.** "Reusable" means one operation, one
authorization model, one idempotency contract and one frontend flow.
"Domain-specific" means each product still asks for its own fields, claims its
own public URL namespace, and seeds its own module document — declared, not
re-implemented.

Registering is the first thing every Sneat product asks a new operator to do —
register a school, a gaming venue, a community centre, a sports club. Each
product implemented that flow independently, and the implementations disagree on
nearly every axis: which space type to use, whether the extension is activated on
the space, whether the module marker is authoritative, where the registration
form lives, and whether registration data beyond the title is persisted at all.

### "Vendor" is not a new record

Eventius already settled the shape and states it in the code
(`eventius/backend/eventius/business_ports.go`): *"No new 'merchant' or 'vendor'
record is created — the company space IS the vendor identity, the location
contact IS the venue."* `RegisterBusiness` composes `CreateCompanySpace` +
`CreateLocationContact` into one flow over ports the host wires. That principle
is adopted platform-wide here: **a vendor is a registered Space**, and vendor
registration is space registration plus the domain's own details.

Two things called "vendor" exist today and must stay linked rather than siloed:
the **business space** above, and bookius' `VendorBotDbo` — the payment-collection
Telegram bot, which already carries an optional `VendorSpaceID` pointing at that
space (see decision 0010 in `backstage`, vendor payment bots). Collecting money
is therefore a *capability added to a registered space*, not a second identity.

### The precedent for "reusable but domain-specific"

Eventius' vertical registry (`eventius/backend/eventius/verticals.go`) is the
in-house pattern: a declarative row per vertical carrying its identity mode and
known labels, with the comment *"New verticals slot in by adding a row … nothing
else in the engine needs to change."* This decision applies that same shape to
registration.

The platform is not missing the machinery. `sneat-core-modules/spaceus` already
provides every part:

- `facade4spaceus.CreateSpace` — creates the space, seats the creator as
  member/creator/owner/contributor, idempotent on `RequestID`.
- `facade4spaceus.CapabilityAuthority.Reserve` — appends an extension id to
  `dbo4spaceus.SpaceDbo.Modules` and issues a bounded, command-bound reservation
  (see the Stable `core-space-capability` feature in `sneat-core-modules`).
- `facade4spaceus.CurrentSpaceAccessAuthority.VerifyActiveOrdinarySpace` —
  refuses when the module marker is absent from `Modules`.
- `contract4spaceus.ManagedSpaceDirectory` — "which of my spaces have extension
  X enabled and grant me role Y", revalidated against authoritative space state.
- `facade4spaceus.ClaimPublicSlug` — a globally unique public slug in a
  caller-supplied namespace, claimed in the same transaction as the space.

Competios is the only product that uses these as a set
(`api4competiosapp.ManagedEventOwnerSpaceDirectory`). What is missing is an
*assembly*: nothing joins the five writes that together constitute registration,
so every product drops a different one.

The most damaging symptom is the space-type vocabulary. Four now exist:

| Source | Values it believes in |
|---|---|
| `sneat-go-core/coretypes` | `personal`, `family`, `group`, `company`, `space`, `club`, `system`, `spot`, `community-center` |
| `@sneat/core-public` (TypeScript) | `family`, `personal`, `group`, `company`, `team`, `parish`, `educator`, `realtor`, `sport_club`, `cohabit`, `community-center`, `unknown` |
| `const4assetus` (pending, local literals) | `community`, `school` — "NOT yet present in coretypes … declared here so `DeriveOwnerType` maps them the moment spaceus ships them" |
| `gametable/web` (over the wire) | `gametable` |

Only five values are common to the first two. Assetus and NoticeBoard
independently reached for the same concept and spelled it `community` and
`community-center`; neither knew of the other. GameTable's registration page
posts `gametable`, which `coretypes.IsValidSpaceType` rejects, so
`POST /v0/spaces/create_space` returns 400 — and the client reports that failure
to the operator as `success: true, isSimulated: true`.

## Decision

### 1. A space's *type* describes its membership shape; its *modules* describe what it does

Products do not mint space types. `coretypes.SpaceType` is a closed, generic
vocabulary describing how membership works, and `dbo4spaceus.SpaceDbo.Modules`
— which already carries the extension id and is already the authorization axis —
is the discriminator for what a space is *for*.

The registration rule is: **staff are members, the public are customers →
`company`.**

| Registering | Space type | Why |
|---|---|---|
| School Portal — a school | `company` | staff are members; students and parents are customers |
| GameTable — a venue | `company` | staff are members; players are customers |
| NoticeBoard.cc — a centre | `company` | staff are members; the public are customers |
| kids-club — an operator business | `company` | already chosen by elimination; now by rule |
| eventius — a business | `company` | already implemented as `BusinessRegistrar.CreateCompanySpace` |
| **sneat.club — a sports club** | `club` | players, guardians, coaches and volunteers are *members*; membership **is** the relationship, not a customer relationship |
| Circleus — a circle | `group` | member-managed; no staff/customer split |

A dedicated space type is reserved for cases where **membership semantics
genuinely differ** — as with `spot`, which has no members at all, and `system`,
which is platform-owned. A centre whose staff are members and whose public are
customers is a *role* distinction (see Competios decision 0009,
"venue content management requires member role"), not a type one.

Consequently:

- **`community-center` is retired.** It exists in `coretypes`, both TypeScript
  unions, `const4communitycentrum` and two tests, and has no production data
  behind it because the NoticeBoard backend was never wired into `sneat-go`.
- **Assetus's pending request for `community` and `school` is cancelled** before
  those literals become a third and fourth spelling of the same idea.
- **`space` is no longer issued** for new registrations — GameTable's backend
  facade was its only caller — but remains valid in `IsValidSpaceType` so any
  legacy record still reads.
- The TypeScript union is reconciled with `coretypes` and kept in step by a test
  that fails on divergence.

### 2. One space may carry more than one product

This follows directly from making the marker, not the type, the discriminator. A
community centre that also runs GameTable tables holds both extension ids in
`Modules` and registers once per product against the same space. Therefore
`ManagedSpaceDirectory` and any "pick an existing space" surface filter by
**extension id**, never by space type.

The sports club is the case that shows the rule discriminating rather than
collapsing everything to `company`. A sneat.club club is type `club` because its
players and coaches are members — and when that club starts collecting
subscriptions it does **not** become a `company`: it gains a payments capability
in `Modules` and a linked `VendorBotDbo`. Type answers "how does membership
work"; modules answer "what can this space do". Under a per-product type scheme
that club would need a type that is both.

### 3. Collecting money is a capability added to a registered space

Vendor registration is space registration plus a payments capability, in that
order. The registration profile creates the space and its module marker; the
vendor payment linkage (bookius `VendorBotDbo.VendorSpaceID`, its provider token,
its hosted bot) attaches to the space afterwards and is never a precondition for
registering. This keeps a free, unpaid, fully-registered vendor a valid state —
which `checkout`'s provisioner already assumes, since it sets a plan on a
`SpaceID` that must exist before payment.

### 4. Registration is one platform operation, driven by a per-extension profile

An extension **declares** what registration means for it, alongside the
extension id and known hosts it already declares:

```go
func Extension() extension.Config {
	return extension.NewExtension(ExtensionID,
		extension.RegisterKnownHosts(KnownHosts...),
		extension.RegisterSpaceProfile(spaceus.SpaceRegistrationProfile{
			SpaceType:     coretypes.SpaceTypeCompany, // staff are members
			SlugNamespace: "communitycentrum:space",   // "" = no public slug
			CreatorRoles:  []string{const4contactus.SpaceMemberRoleMember},
			SeedModuleDoc: newCentreModuleDbo, // optional
		}),
	)
}
```

`facade4spaceus.RegisterSpace` executes that profile in **one transaction**:

1. Create or replay the space, with the type taken from the profile — never from
   the request, so a client cannot name a type the extension does not own.
2. Persist the registration fields (`CountryID` today; `City`/`Timezone` as
   `CreateSpaceRequest` grows) rather than validating and discarding them.
3. Append the extension id to `SpaceDbo.Modules`, reusing the semantics
   `CapabilityAuthority.Reserve` already implements.
4. Claim the public slug when the profile names a namespace and the caller
   supplied a slug.
5. Seed `/spaces/{spaceID}/ext/{extID}` so the first real write updates a
   document that exists.

Two HTTP routes replace the per-product surfaces:
`POST /v0/spaces/register_space`, and `POST /v0/spaces/list_manageable?ext=…`
(generalising Competios' `ListEventOwnerSpaces`).

### 5. One authorization rule

The module marker is written at registration and checked on every module-scoped
write. `dal4spaceus.RunModuleSpaceWorkerWithUserCtx` — the path schoolus and
every classic extension uses — today checks membership only and never reads
`Modules`, while Competios treats the same field as the gate. It gains the
`Modules` check that `AccessAuthority` already applies, after a backfill deriving
`Modules` for existing spaces from their `/ext/*` subcollections.

### 6. `RequestID` is client-minted and stable

Idempotency is currently defeated from both ends: `gametable/web` sends
`req-venue-${Date.now()}` (new on every attempt, so a double submit creates two
venues), while `facade4gametable` and `facade4communitycentrum` derive the id
from an *optional* `spec.SpaceID` (empty ⇒ the constant `"gt-space-"` /
`"cc-space-"`, so a user's second venue replays the first and fails the title
check with a misleading conflict). A registration client mints a UUID when the
form is opened, not when it is submitted, so a retry is a retry.

## Rationale

- `Modules` already is the authorization axis. Making it the product
  discriminator too means one fact in one place, rather than a type and a marker
  that can disagree.
- A space type per product multiplies work in files no product owns: the
  ordinary-space switch in `capability_authority.go`, the TypeScript union, every
  emoji/icon pipe, every space-picker filter, and `const4assetus.DeriveOwnerType`.
  The assetus/NoticeBoard collision is that cost arriving before either shipped.
- Registration is five facts written together. Expressing it as a declaration the
  platform executes — rather than five calls each product makes in its own order
  — is what makes "did we register correctly?" reviewable.
- Retiring `community-center` is cheapest now: six files, no production data.

## Declined Alternatives

### A space type per product (`community-center`, `school`, `venue`)
Rejected. It reproduces the module marker in a second, weaker form: the type
cannot express a space that carries two products, and every new product forces
edits to unrelated modules. NoticeBoard and assetus had already diverged on the
spelling of one such type before either reached production.

### Keep per-product registration facades, unify only the space type
Rejected. The type mismatch is the visible symptom; the missing transaction is
the defect. Aligning types alone would still leave GameTable creating spaces with
no module marker and NoticeBoard discarding the operator's city.

### Derive the asset owner type from a new field on the space
Under generic types, `const4assetus.DeriveOwnerType` can no longer distinguish a
school from a venue from a centre — all three are `company`. Denormalising an
`AssetOwnerType` from the registration profile onto the space record was
considered and rejected: it stores the same fact twice and can drift from
`Modules`. `DeriveOwnerType` instead takes `Modules` as an additional input.

### Registration gated behind payment
Not adopted. `checkout`'s `entitlementProvisioner` sets a space's plan from an
entitlement carrying a `SpaceID`, so the space must exist before checkout. The
free tier is therefore "registered but unpaid", and registration itself needs no
payment gate.

## Consequences at Decision Time

- `coretypes.SpaceTypeCommunityCentre` and both TypeScript occurrences of
  `community-center` are removed; `const4communitycentrum` registers `company`.
- `const4assetus`' `spaceTypeCommunity` / `spaceTypeSchool` literals are removed
  and `DeriveOwnerType` gains `Modules` as an input. **This is a real change to
  assetus, not a no-op**, and lands with the seam rather than after it.
- `validateOrdinarySpace` is driven by the registration-profile registry instead
  of a hard-coded switch, so adding a product no longer requires editing
  `capability_authority.go`.
- GameTable's `web/` registration page must send a valid type, mint a stable
  `RequestID`, and stop reporting failed registrations as successes.
- `gametable/backend` and `communitycentrum/backend` are absent from
  `sneat-go/go.mod` and have no `api4` package; both must be wired before
  registration is reachable at all.
- kids-club's `resolveBusinessSpaceID` and rsvp-express's `resolveFamilySpaceID`
  are superseded by `list_manageable`.
- `sneat-club` is today landings-only (Astro; no app, no backend), so it has no
  registration surface to migrate. It is the greenfield adopter: its club
  registration should be built on the seam rather than added first and unified
  later, which is how the other three arrived here.
- Vendor payment linkage stays where it is (bookius `VendorBotDbo`), attaching to
  a registered space. No product gains a merchant or vendor record of its own.

## Observed Consequences

- 2026-08-23: Phase 0 prepared — GameTable web registration corrected (valid
  space type, stable `RequestID`, fake-success fallback removed) and the
  constant-`RequestID` defect fixed in both `facade4gametable` and
  `facade4communitycentrum`. The `RegisterSpace` seam itself is not yet built.

## Affected Features

- `core-space-capability` (sneat-core-modules, Stable) — `validateOrdinarySpace` becomes registry-driven
- spaceus public-slug (sneat-core-modules) — gains the HTTP route it currently lacks
- schoolus / gametable / communitycentrum / sneat-club — vendor & space registration
- vendor-payment-bots (backstage decision 0010) — `VendorSpaceID` links a payment bot to the registered space

## Open Questions

- Is a public slug mandatory at registration for public-facing products?
  Claiming it inside the registration transaction guarantees every registered
  venue has one, at the cost of failing registration on a slug collision.
- Should `personal` and `family` spaces ever carry product modules, or is
  registration restricted to ordinary shared spaces?
- Does the `Modules` backfill need to run before or after
  `RunModuleSpaceWorkerWithUserCtx` starts enforcing the marker? (Before, if any
  existing space has module data but no marker — which must be measured, not
  assumed.)

---
*This document follows the https://specscore.md/decision-specification*
