# Standards

Cross-repo house standards and conventions for the Sneat platform. Unlike
[`spec/`](../spec/README.md), these are living conventions rather than
SpecScore-gated feature specs — they describe how we build, not what we're
building.

## Contents

| Standard | Purpose |
|---|---|
| [Brand Usage](brand-usage.md) | "Sneat" never stands alone in copy — always suffixed (Sneat Co./App/Team/Work, the Sneat platform, a Sneat product). Founder rule 2026-07-09. |
| [Frontend UX](frontend-ux/README.md) | UI/UX conventions for Sneat extensions — cards, buttons, lists, page layout, forms, modals, and loading/empty/error states, each citing real components from `calendarius` / `contactus` / `listus`. |
| [Repository Naming](repo-naming.md) | How repos are named (`sneat-*` platform, `<id>` product/implementation repos, `ext-<id>` public extension-definition repos) + the GitHub-topics registry for finding all extensions. |
| [Extension Backend Architecture](extension-backend-architecture.md) | Ports & adapters for extension backends: domain module depends on `dalgo` only; ports defined in the extension, adapters in the host (`sneat-go`). Reference: `eventius/backend`. |
