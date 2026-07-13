---
format: https://specscore.md/decision-specification
status: In Review
---
# Decision: Sneat.work hosting (app.sneat.work) and the Sneat.team merge

**Status:** In Review
**Date:** 2026-07-13
**Owner:** alex
**Tags:** website, brand, hosting, domains, auth, seo
**Source Idea:** —
**Supersedes:** —
**Superseded By:** —

## Context

Sneat.work is the professional/work side of Sneat (co-equal with Sneat.family and
Sneat.club) — a modular, team-first platform. Two structural questions were open:

1. **Where does the authenticated app live?** The `sneat-work` worker previously
   root-mounted the Angular app at `sneat.work/` (landing on reserved paths, app
   everywhere else), per [ADR-0001 root-mounted app routing](https://github.com/sneat-co/backstage/blob/main/docs/decisions/0001-root-mounted-app-routing.md).
2. **What happens to Sneat.team?** `sneat.team` had a standalone landing + app +
   waitlist/Stripe. The decision is to **merge it into the Team module of
   Sneat.work** (business/work teams; sport stays on Sneat.club).

This decision records the domain responsibilities, the app subdomain, and the
redirect strategy, resolving the corresponding Open Questions in the `sneat-work`
idea.

## Decision

### Domain responsibilities
- **sneat.work** — public product & module landings (marketing): `/`, `/team/`,
  `/flow/`, `/privacy/`. Marketing only; canonical host for SEO/OG.
- **app.sneat.work** — the authenticated Sneat Work Angular application
  (root-mounted SPA). Deep links use the real scheme
  `app.sneat.work/space/:spaceType/:spaceID` (+ module segments, e.g. `/lists`,
  `/retros`), matching the existing Sneat app router — no new URL scheme invented.
- **sneat.team** — retained only as a **301 redirect** into the merged module:
  `sneat.team/*`, `www.sneat.team/*` → `https://sneat.work/team/` (query
  preserved); `app.sneat.team/*` → `https://app.sneat.work/<path>` (path + query
  preserved). `app.sneat.team` is not built. `scrumspace.app` (legacy) is
  repointed straight to `sneat.work/team/` to avoid a redirect chain.

One `sneat-work` worker serves both `sneat.work` and `app.sneat.work` (a single
canonical deployment), routed by hostname in `landings/worker.js`.

### Team merge
- The former Sneat.team becomes **`sneat.work/team/`**: a landing that preserves
  the strongest Sneat.team content (the rituals wedge — Sprints & Retros,
  standups, Moodometer, IssueNumber.One) and reframes it — **rituals are the
  current wedge, not the full scope**. Team grows into the people-and-organisation
  layer (members, roles, org units, invitations, permissions, ownership,
  onboarding, recurring meetings, decisions, goals, ways of working).
- Not renamed to "Rituals". Not a sports product (that is Sneat.club).

### Rationale for `app.<subdomain>` (departure from ADR-0001, recorded)
[ADR-0001](https://github.com/sneat-co/backstage/blob/main/docs/decisions/0001-root-mounted-app-routing.md)
mandates a **same-origin root-mount** so the landing can read the app's Firebase
auth session (persisted in **per-origin IndexedDB**) and show signed-in state. A
subdomain breaks that single-origin read. This decision **departs from ADR-0001
for Sneat.work**, accepting the consequence and taking the benefits:

- **Consequence:** the `sneat.work` marketing pages **cannot** read the app's auth
  session; they link out ("Open the app") rather than personalising. This is
  acceptable for a pure marketing surface.
- **Benefits:** a clean marketing/app separation; redirect-based auth is fully
  self-contained on the app origin (`authDomain` auto-defaults to
  `app.sneat.work`); analytics and canonical SEO are unambiguous; the split scales
  to many Sneat.work modules without polluting the app origin with marketing paths.

### Safe, staged rollout
`landings/wrangler.jsonc` ships `vars.APP_SPLIT_ENABLED: "false"`. While false,
`sneat.work` still serves the app shell in place (backward compatible), so
deploying the code before `app.sneat.work` exists breaks nothing. It is flipped to
`"true"` only after the manual provider steps below, at which point app deep links
on `sneat.work` `301` to `app.sneat.work`.

## Rationale
- A marketing/app split by subdomain is the clearest way to make "sneat.work =
  the platform, app.sneat.work = the app" legible to visitors, search engines and
  analytics, and it keeps auth self-contained.
- Merging Sneat.team removes a duplicate product identity and a second app host,
  giving one product hierarchy, one app, simpler auth, and room for many modules.
- A single hostname-routed worker avoids duplicating the deployment.

## Declined Alternatives

### Keep the app root-mounted on `sneat.work` (strict ADR-0001)
Rejected for Sneat.work: it entangles marketing and app on one origin, and the
brief explicitly asked for `sneat.work` (marketing) + `app.sneat.work` (app). The
per-origin-auth benefit of ADR-0001 (landing shows signed-in state) is low value
for a modular marketing site whose CTA is simply "Open the app".

### Two separate deployments (landing worker + app worker)
Rejected: unnecessary duplication. One worker with two custom domains is a single
canonical deployment and keeps the existing app-assembly step.

### Keep Sneat.team as a separate product / app
Rejected: it maintains two parallel content trees and two app identities. The
founder decision is to merge; `sneat.team` becomes a redirect. (This supersedes
the "sneat.team survives a merge / is a sibling" note in
[`backstage/spec/ideas/seeds/sneat-team-vs-sportius.md`](https://github.com/sneat-co/backstage/blob/main/spec/ideas/seeds/sneat-team-vs-sportius.md).)

### Redirect every sneat.team path to a fabricated equivalent
Rejected: there is one `/team/` page; fabricating `/team/<path>` deep links would
404. Paths collapse to `/team/` with query preserved (invite/campaign context).

## Consequences at Decision Time
- `sneat.work` becomes marketing-only with `/team/` and `/flow/` module pages;
  the app moves to `app.sneat.work` (once the manual steps land).
- `sneat.team-website` becomes a redirect worker; its waitlist/Stripe backend is
  retired (KV data retained in Cloudflare; code in git history).
- The following are **manual provider/DNS actions** (not repo-changeable), and
  gate flipping `APP_SPLIT_ENABLED` to `"true"`:
  1. Attach `app.sneat.work` as a Cloudflare Custom Domain on the `sneat-work`
     worker (needs `Zone:DNS:Edit`).
  2. Zone route `app.sneat.work/__/*` → the `firebase-auth-proxy` worker.
  3. Firebase Auth (`sneat-eur3-1`) → Authorised domains → add `app.sneat.work`.
  4. OAuth providers → add `https://app.sneat.work/__/auth/handler` + JS origin.
  5. Point `sneat.team` nameservers at Cloudflare and enable the redirect routes.

## Observed Consequences
- 2026-07-13: repo changes prepared (sneat-work landing `/team/` + `/flow/`,
  hostname-routed worker + `app.sneat.work` route behind `APP_SPLIT_ENABLED`;
  sneat.team-website redirect worker; scrumspace redirect retargeted). Manual DNS
  + Firebase steps pending; no live cutover yet.

## Affected Features
- None directly (marketing + hosting). Related specs referenced descriptively:
  the Team-rituals suite, Flow, and the `sneat-work` idea.

## Open Questions
- Whether `app.sneat.work` should open a space selector, a work-specific
  onboarding, or a default work dashboard (today the app home renders the space
  selector, `<sneat-spaces-card/>`).
- Whether Sneat.work should ever pin `authDomain` explicitly vs. the current
  origin-default (kept default so previews/local dev work unchanged).
- When to retire the `sneat-team-vs-sportius` seed's "survives a merge" note fully
  vs. keep it as historical context.
- Confirm the `app.` subdomain as a sanctioned ecosystem pattern (it is currently
  an Open Question in the site-hosting-pattern feature; this decision sets the
  first precedent).

---
*This document follows the https://specscore.md/decision-specification*
