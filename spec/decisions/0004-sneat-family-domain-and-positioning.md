---
format: https://specscore.md/decision-specification
status: In Review
---
# Decision: sneat.family — domain role, family positioning, and marketing site

**Status:** In Review
**Date:** 2026-07-13
**Owner:** alex
**Tags:** website, brand, positioning, seo, domains
**Source Idea:** —
**Supersedes:** —
**Superseded By:** —

## Context

The Sneat platform needed a dedicated public home for its family ecosystem,
distinct from the application (sneat.app) and the company/ecosystem site
(sneat.co). This decision records the domain responsibilities, the family
product positioning, the information architecture, and the accuracy rules that
govern the new marketing site at **https://sneat.family**, implemented in
`sneat-sites/websites/sneat.family` (Astro → Cloudflare Workers static assets,
matching the sneat.co / sneat.money hosting pattern).

## Decision

### Domain responsibilities
- **sneat.family** — family-oriented marketing, storytelling, discovery, SEO,
  education, trust and product navigation. Marketing only; never hosts the app.
- **sneat.app** — the actual application, authentication and account creation
  (private beta). All product CTAs on sneat.family lead here (via `/pwa/`).
- **sneat.co** — company, wider ecosystem, technology, open source, investors,
  non-family verticals.

sneat.family never proxies or iframes the application. www.sneat.family 301s to
the apex (canonical host).

### Positioning
- Immediate promise: **“One connected place for your family life.”**
- Category ambition (used selectively, not inflated): building the operating
  system for modern family life.
- Sneat is a **connected family system**, not a calendar utility, an organiser,
  or a bag of disconnected tools. The narrative leads with outcomes and the
  structural problem (scattered tools + mental load), never by attacking
  competitors.
- Tone: British English; practical, warm, modest, ambitious without hype.
  Stripe-grade clarity, not a childish consumer template.

### Primary audiences (in order)
1. Busy parents. 2. Families with children. 3. Families coordinating children’s
school/activities/health/transport/documents/events. 4. Adults helping elderly
parents. The site is explicitly inclusive of extended, blended and
multi-household families and must never imply one fixed family structure.

### Capabilities — grouped by outcome, status-gated (accuracy rule)
Grouped as Family coordination / knowledge / administration / intelligence.
Every capability and product carries an honest status and must not be described
in the present tense unless true:
- **In the beta (now):** shared calendar, children’s activities, contacts &
  relationships graph, events, extended/emergency contacts, lists.
- **Early access:** reminders & responsibilities (remindius.app, live but early).
- **Coming soon:** documents, renewals, household budgeting, and the **Sneat AI
  assistant**. AI is prominent in the narrative but always marked coming soon;
  positioned as a layer across the family’s own connected context, not a chatbot.

### Connected-product inclusion rules
Only products that genuinely exist are listed, each labelled honestly:
- Included: Remindius (early), RenewOn, Documentus, Formius, Sizeus
  (sizechart.app), AnyMeter, ReQuoter, Bookius (all coming soon).
- **Omitted** (idea/placeholder-only or non-family): Yardius, Surpriseless,
  Budgetus, Kids-club, Schoolus / School Portal. Internal module names
  (Calendarius, Contactius) are not exposed as products.

### Origin story
The Dan & Tala Rushmore seven-panel story is reused (same panel art) but
**reframed** as “a familiar family story”, not literal company history. The
explicit “the Rushmores are fictional, but the chaos is real” statement is
preserved. Canonical home moves to **/story/**; the sneat.co copy should later
receive a cross-domain canonical to avoid duplication (see Open Questions).

### Information architecture (initial release)
`/`, `/why-sneat/`, `/features/`, `/families/` (+ `/parents/`,
`/children-and-activities/`, `/elderly-parents/`), `/ai/`, `/products/`,
`/story/`, `/about/`, `/pricing/`, `/private-beta/`, `/security/`, `/privacy/`,
`/faq/`, `/contact/`, `/accessibility/`, `/terms/`, custom 404. URL conventions
are trailing-slash directories, ready to scale to future product/use-case/
region/locale pages. English-first with hreflang (`en`, `x-default`) already
emitted so localisation (e.g. Russian) can be added without a URL migration.

### Trust, SEO, analytics
- Privacy-first GA4 with Consent Mode v2, EU/UK geo-gated, cookieless until
  consent, Google Signals/ad-personalisation disabled; a dedicated GA4 property
  is still required (site ships with analytics disabled until then).
- Full technical SEO: per-page titles/descriptions, canonical, OG/Twitter,
  Organization + WebSite + SoftwareApplication + FAQPage JSON-LD, sitemap,
  robots.txt, llms.txt, custom 404. Security + caching headers via `_headers`.
- Trust/legal pages (privacy, terms) ship as clearly-marked drafts (noindex)
  pending legal review; binding policies remain with the app on sneat.app.

## Rationale
- A dedicated family home lets us tell a coherent outcomes-led story to busy
  parents without diluting sneat.co (company/investors) or overloading the app
  entry on sneat.app. Splitting audiences by domain keeps each surface focused.
- Reusing the existing Astro → Cloudflare hosting pattern (not a new repo) means
  zero new infrastructure, shared conventions, and one-command deploys.
- Gating every public claim on typed `status` data enforces honesty during
  private beta and makes "what we may say" a reviewable code change, not prose.

## Declined Alternatives

### Host family marketing on sneat.app or sneat.co

Rejected: sneat.app is the application (auth/accounts) and must not carry SEO
marketing; sneat.co is company/investor-facing. Mixing audiences on one host
weakens all three and blurs the calls to action.

### A separate new repository for sneat.family

Rejected: the `sneat-sites` monorepo already provides the Astro build, Cloudflare
deploy and CI pattern. A new repo adds overhead for no technical gain (brief §21
prefers existing conventions unless there is a strong technical reason).

### Move the application to sneat.family

Rejected — and explicitly forbidden by the brief. The app stays on sneat.app;
sneat.family is marketing only and links out to it, never proxying or iframing.

### Serve www as a second custom domain

Rejected in favour of a 301 from www to the apex, to avoid duplicate-content
hosts and keep one canonical marketing host (matches the sneat.money pattern).

## Consequences at Decision Time
- A coherent, honest, accessible (WCAG 2.2 AA target) family marketing site is
  live and integrated with the existing sites; sneat.co points family visitors
  to sneat.family.
- Public copy is constrained by status data files, so future launches must
  update `data/capabilities.ts` / `data/products.ts` to change what is asserted.
- Trust/legal pages ship as noindex drafts until legal review; analytics stays
  off until a dedicated GA4 property exists.

## Observed Consequences
- 2026-07-13: initial release deployed (20 pages, worker `sneat-family`, apex +
  www→apex 301, security/caching headers, sitemap/robots/llms.txt). All routes
  return 200; no mobile overflow. No post-launch issues recorded yet.

## Affected Features
- None yet (marketing site; no SpecScore feature depends on it). Related product
  specs are referenced only descriptively (Remindius, RenewOn, Documentus,
  Formius, Sizeus, AnyMeter, ReQuoter, Bookius) and are not modified by this
  decision.

## Open Questions
- Dedicated GA4 property + measurement ID for sneat.family.
- Confirm `hello@sneat.family` mailbox before relying on it publicly.
- Cross-domain canonical (or 301) from sneat.co/about/origin-story/ →
  sneat.family/story/ to prevent duplication (deferred: it alters an existing
  ranking page and warrants review).
- Founder/legal sign-off on trust/legal drafts, data-export/portability
  commitments, and the AI data-handling description before they read as
  guarantees.
- Confirm pricing direction (free-in-beta stated; future tiers left open).
