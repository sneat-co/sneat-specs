---
format: https://specscore.md/decision-specification
status: In Review
---
# Decision: Split auth.sneat.co (Firebase sign-in) from oauth.sneat.co (Sneat OAuth server)

**Status:** In Review
**Date:** 2026-10-16
**Owner:** alex
**Tags:** auth, oauth, hosting, domains, cloudflare, firebase, reliability
**Source Idea:** auth-oauth-host-split
**Supersedes:** —
**Superseded By:** —

## Context

On 2026-10-16 the shared Cloudflare account exceeded the Workers Free plan's 100,000 requests/day limit; every Worker-first site returned Error 1027 until 00:00 UTC. `auth.sneat.co` — the Firebase `authDomain` of every Sneat app — is served by the `sneat-api-proxy` Worker, so sign-in failed across all products, including sites already served as static assets.

The same host also carries Sneat's own OAuth server (Go, `sneat-core-modules/auth/oauth/api4oauth`, Cloud Run), which the Worker fronts with an origin secret (`X-Sneat-OAuth-Origin-Secret`). The two jobs have different requirements: sign-in must never depend on a request quota; the OAuth server needs the origin secret. See the [Idea](../ideas/auth-oauth-host-split.md) for the full context and alternatives.

## Decision

Split the two jobs onto two hosts (founder decision, 2026-10-16: "I choose option 2"):

| Host | Serves | Hosted by | Worker? |
|------|--------|-----------|---------|
| **`auth.sneat.co`** | Firebase sign-in helper only: `/__/auth/handler`, `/__/auth/iframe`, `/__/firebase/init.json` | **Firebase Hosting** custom domain (project `sneat-eur3-1`), native reserved URLs | **No** |
| **`oauth.sneat.co`** | Sneat OAuth server: `/oauth/authorize`, `/oauth/token`, `/oauth/userinfo`, `/oauth/revoke`, `/oauth/device/*`, `/device`, `/oauth/firebase-config`, and its discovery metadata | Cloud Run behind the existing `sneat-api-proxy` Worker (route `oauth.sneat.co/*`), origin secret unchanged | Yes (low traffic) |

### Where each host is used

- **`auth.sneat.co`** — used implicitly by the Firebase JS SDK in every Sneat web app (`authDomain: auth.sneat.co`) when a user signs in with Google, Apple, or another Firebase provider (popup or redirect). OAuth providers' registered redirect URI stays `https://auth.sneat.co/__/auth/handler`. Users see it only briefly during sign-in. No app config changes.
- **`oauth.sneat.co`** — used explicitly by clients that do **"Sign in with Sneat"**: the Sneat CLI and other CLIs' device login (`/oauth/device/code`, then the user opens `https://oauth.sneat.co/device`), MCP clients and third-party integrations using the authorization-code flow (`/oauth/authorize`, `/oauth/token`), and token introspection or revocation. It is the OAuth issuer in metadata and tokens.
- The OAuth server's own pages (`/device`, `/oauth/authorize`) still sign the user in through Firebase, with `authDomain = auth.sneat.co`.

### Compatibility

Old `https://auth.sneat.co/oauth/*` and `/device` URLs (released CLIs, stored client configs) keep working during a compatibility window through a method-preserving redirect (308) or a narrow route to the OAuth server; the mechanism and window length are set in the implementation plan. After the window they are retired.

## Rationale

Sign-in is on the critical path of every Sneat product and Firebase's sign-in helper is native only on Firebase Hosting, so serving it there removes the dependency on a Worker and its daily quota at no cost. The OAuth server is a token issuer whose protection relies on the origin secret only a Worker can add; it is low-traffic, so keeping it behind the Worker costs almost nothing. Splitting by host gives each job the hosting that fits it without weakening either.

## Declined Alternatives

### Status quo — both jobs on auth.sneat.co behind the Worker

A Workers quota outage takes down sign-in for every Sneat product, and every sign-in costs Worker invocations.

### Both jobs on auth.sneat.co via Firebase Hosting with rewrites to Cloud Run

Firebase Hosting cannot add the origin secret, so the Go server would have to trust a forwarded host header that anyone calling the `run.app` URL could forge — weaker protection for a token issuer.

### Self-host Firebase's sign-in helper as static assets on Cloudflare

Firebase documents that this does not support Apple sign-in or SAML (Apple is offered), and the copied files must be re-synced by hand.

## Consequences at Decision Time

**Pros**
- Sign-in for every Sneat app no longer depends on a Cloudflare Worker or its daily quota; a Worker outage cannot take down sign-in.
- No Worker invocations per sign-in (several per sign-in today).
- Firebase serves `/__/firebase/init.json` natively; the synthesised copy in the Worker and the Cloud Run `/oauth/firebase-config` dependency on the sign-in path go away.
- The OAuth server keeps its origin-secret protection unchanged.
- Clear ownership: each host has one job.

**Cons / costs**
- OAuth clients and stored configurations that name `auth.sneat.co/oauth/...` must move to `oauth.sneat.co`; the issuer URL changes, so token validation must accept the new issuer (and the old one during the window).
- One more hostname to operate (DNS, certificate, route).
- Uses Firebase Hosting for one host, an exception to the "web surfaces deploy to Cloudflare" preference — justified because Firebase's sign-in helper is native only on Firebase Hosting.
- The OAuth server still runs behind a Worker, so OAuth (not sign-in) remains subject to the Workers quota; acceptable at its traffic level.

## Observed Consequences

None yet — not implemented at decision time.

## Affected Features

- Sneat OAuth server — `sneat-core-modules` `auth/oauth` (OAuth host configuration, issuer, metadata).
- Sneat CLI device login and other OAuth clients — issuer and endpoint URLs move to `oauth.sneat.co`.
- `sneat-api-proxy` Worker (`sneat-sites/infra/sneat-api-proxy`) — routes and `/__/auth` proxying.
- Firebase auth configuration in Sneat apps (`sneat-libs`) — unchanged (`authDomain: auth.sneat.co`).

## Implementation notes

- **sneat-core-modules (`auth/oauth`)**: separate the OAuth public host from the Firebase `authDomain` (new config field; `allowedRequestHost`, issuer and metadata URLs use the OAuth host; `/oauth/firebase-config` keeps returning `authDomain: auth.sneat.co`).
- **sneat-go**: wire the new configuration only (per `rule:sneat-go-wire-and-configure-only`).
- **sneat-sites (`infra/sneat-api-proxy`)**: add route `oauth.sneat.co/*`; drop the `/__/auth` and `init.json` proxying for `auth.sneat.co`; remove the `auth.sneat.co/*` route at cutover.
- **Firebase Hosting**: a site for `auth.sneat.co` with the custom domain provisioned (certificate active) **before** DNS moves, so the cutover has no sign-in downtime.
- **Clients**: update the Sneat CLI and other clients to `oauth.sneat.co`; ship before the compatibility window ends.

## Open Questions

- Final OAuth host name: `oauth.sneat.co` (working assumption) or a `sneat.cloud` host.
- Compatibility-window mechanism and length for old `auth.sneat.co/oauth/*` URLs.

---
*This document follows the https://specscore.md/decision-specification*
