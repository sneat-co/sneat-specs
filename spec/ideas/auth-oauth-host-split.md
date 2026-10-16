---
format: https://specscore.md/idea-specification
status: Approved
---

# Idea: Split auth.sneat.co (Firebase sign-in) from oauth.sneat.co (Sneat OAuth server)

**Status:** Approved
**Date:** 2026-10-16
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might we keep sign-in for every Sneat app working even when the Cloudflare Workers daily quota runs out, without weakening the Sneat OAuth server's origin protection?

## Context

On 2026-10-16 the Cloudflare account exceeded the Workers Free plan's 100,000 requests/day limit and every Worker-first site returned **Error 1027** until 00:00 UTC. DemoDB downloads by CI consumed most of the quota, but the blast radius was account-wide.

`auth.sneat.co` is served by one Worker, `sneat-api-proxy` (zone route `auth.sneat.co/*`, code in `sneat-co/sneat-sites/infra/sneat-api-proxy/worker.js`), and does two unrelated jobs:

1. **Firebase sign-in helper** — `/__/auth/handler`, `/__/auth/iframe`, `/__/firebase/init.json`. `auth.sneat.co` is the default Firebase `authDomain` for every Sneat app, so this is on the sign-in path of all products. Because Sneat apps are not on Firebase Hosting, the Worker proxies these paths to `sneat-eur3-1.firebaseapp.com` and synthesises `init.json`.
2. **Sneat's own OAuth server** — `/oauth/authorize`, `/oauth/token`, `/oauth/userinfo`, `/oauth/revoke`, `/oauth/device/*`, `/device`, `/oauth/firebase-config` (Go, `sneat-co/sneat-core-modules/auth/oauth/api4oauth`, on Cloud Run). The Worker adds `X-Sneat-OAuth-Origin-Secret`; the Go handler rejects requests reaching the `run.app` URL without it.

So when the Worker quota ran out, sign-in failed for every Sneat product — even sites already moved to static assets — because the sign-in helper sits behind a Worker.

## Recommended Direction

Give each job its own host and the hosting that fits it:

- **`auth.sneat.co` → Firebase sign-in only, served natively by Firebase Hosting** (custom domain on a Hosting site in project `sneat-eur3-1`). Firebase Hosting serves `/__/auth/*` and `/__/firebase/init.json` itself — no proxy, no Worker, no request quota. The Firebase `authDomain` stays `auth.sneat.co`, so no app config changes and existing OAuth-provider redirect URIs (`https://auth.sneat.co/__/auth/handler`) stay valid.
- **`oauth.sneat.co` → Sneat's OAuth server**, behind the existing `sneat-api-proxy` Worker with its origin secret, exactly as today. The OAuth server is low-traffic, so its Worker invocations are negligible against the quota.

Users and apps see the split like this: an app's "Sign in with Google/Apple/…" flow uses `auth.sneat.co`; a CLI, MCP client or third-party integration that does "Sign in with Sneat" (authorization code or device flow) uses `oauth.sneat.co`.

This removes the Worker from the one path where an outage hurts every product, while keeping the OAuth server's security model unchanged.

## Alternatives Considered

- **Keep both jobs on `auth.sneat.co` behind the Worker (status quo).** Lost: a quota outage takes down sign-in everywhere; every sign-in costs several Worker invocations.
- **Both jobs on `auth.sneat.co` via Firebase Hosting, with Hosting rewrites of `/oauth/**` and `/device` to Cloud Run.** Lost: Firebase Hosting cannot add the origin-secret header, so the Go server would have to trust the `X-Forwarded-Host` that Hosting forwards; anyone calling the `run.app` URL directly could then claim to be `auth.sneat.co`. Weaker origin protection for the OAuth server, which issues tokens.
- **Self-host Firebase's sign-in helper files as static assets.** Lost: Firebase documents that this does not work for Apple sign-in or SAML (Apple is offered), and the copied files must be re-synced by hand.

## MVP Scope

Sign-in for all Sneat apps no longer depends on any Cloudflare Worker: `auth.sneat.co/__/auth/*` and `/__/firebase/init.json` are answered by Firebase Hosting, and the OAuth server answers on `oauth.sneat.co` with unchanged behaviour. Released CLIs move to `oauth.sneat.co` (one-time re-login; see Decision 0007).

## Not Doing (and Why)

- **Moving the OAuth server off Cloud Run** — not the problem; Cloud Run is fine for it.
- **Changing the Firebase `authDomain` away from `auth.sneat.co`** — would touch every app's config and every provider's registered redirect URI for no gain.
- **Dropping the origin secret** — it is what keeps the OAuth server reachable only through the proxy; the split exists precisely to keep it.
- **Fixing the redirect-sign-in storage-partitioning issue for apps that use a cross-origin `authDomain`** — real, but separate; tracked as its own follow-up.

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | Firebase Hosting serves `/__/auth/*` and `/__/firebase/init.json` on a custom domain such that redirect and popup sign-in work with `authDomain = auth.sneat.co` | Canary: add the custom domain to a Hosting site, sign in with Google and Apple (popup + redirect) in Chrome and Safari before switching DNS for real traffic |
| Must-be-true | The OAuth server can run with its public host (`oauth.sneat.co`) different from the Firebase `authDomain`, including its `/device` and `/oauth/authorize` pages that load Firebase sign-in | Go change + tests; browser test of device flow and authorize flow on the new host |
| Should-be-true | Only sneat-cli and datatug-cli call the OAuth server, so moving them is enough | Inventory done 2026-10-16 (no other clients found); re-check before cutover |
| Should-be-true | Switching DNS for `auth.sneat.co` can be done with no sign-in downtime | Provision the Firebase custom domain and certificate before moving traffic |
| Might-be-true | Firebase Hosting works behind a proxied Cloudflare record | Firebase and Cloudflare docs; otherwise use a DNS-only record |

## SpecScore Integration

- **New Features this would create:** TBD at design time (OAuth host configuration in sneat-core-modules auth/oauth).
- **Existing Features affected:** Sneat OAuth server (sneat-core-modules `auth/oauth`), Sneat CLI device login, Firebase auth configuration in sneat-libs (unchanged authDomain).
- **Dependencies:** [Decision 0007](../decisions/0007-auth-and-oauth-host-split.md).

## Open Questions

None at this time — host name and CLI compatibility are decided in Decision 0007.
