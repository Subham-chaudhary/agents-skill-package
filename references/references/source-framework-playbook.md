# Source-framework migration playbook

The skill must choose a migration lane after inspecting the repository. These are default strategies, not hard limitations.

| Source app | Recommended first move | Typical risk | Notes |
|---|---|---:|---|
| React + Vite SPA | Add Next.js carefully; initially preserve SPA semantics/router if needed; move routes incrementally | Low–medium | Audit Vite env variables, aliases, asset URLs, service worker, proxies and build scripts. Official guide: https://nextjs.org/docs/pages/guides/migrating/from-vite |
| Create React App | Follow current Next.js CRA guide; retain a client-only entry point during the first pass when safer | Low–medium | Audit `REACT_APP_*`, `homepage`, service worker, proxies, custom webpack/Babel changes and public assets. https://nextjs.org/docs/app/guides/migrating/from-create-react-app |
| Existing Next.js Pages Router | Keep Pages Router if it meets the standard; otherwise migrate page groups to App Router incrementally | Low–medium | Don't perform App Router conversion merely for cosmetic consistency. https://nextjs.org/docs/app/guides/migrating/app-router-migration |
| React app with a separate Node backend | Migrate frontend first; keep API boundary and backend while mapping auth/session/CORS/cookies | Medium | Consolidate endpoints into Next.js Route Handlers only after contract and parity tests exist. |
| Vue / Nuxt / Svelte / Angular | Rebuild UI route-by-route in React/Next.js; preserve service/API contracts first | Medium–high | No trustworthy mechanical source translation should be assumed. Use screenshots, route maps, tests and behavior descriptions. |
| Static HTML/CSS/JS site | Port pages and shared layout, preserve URL paths/assets/metadata; decide static vs server rendering from actual needs | Low–medium | Verify forms, external scripts, analytics, redirects, CSP, image paths and custom headers. |
| Django / Flask / Rails / Laravel / Spring or other non-Node backend | Migrate UI separately; retain backend behind a stable API until an explicit backend rewrite is justified | High | Do not port business rules or data semantics by translating files. Preserve migrations, auth, jobs and webhooks. |
| No Git repo / incomplete code export | Establish source ownership/export and identify missing runtime services before migration | High / blocked | Do not claim complete portability when source, database schema, secrets, uploads or provider configuration are missing. |
| Monorepo / microfrontend / multiple services | Inventory package and service boundaries; define migration scope per app | High | Preserve workspace tooling and service contracts; avoid moving whole directories without dependency analysis. |

## Framework-specific audit reminders

### React/Vite/CRA

- Map current route paths, route params, query behavior, history behavior and catch-all routes.
- Map environment variables. Vite's client exposure conventions differ from Next.js; don't mechanically prefix every variable with `NEXT_PUBLIC_`.
- Inventory `import.meta.env`, `import.meta.glob`, aliases, asset imports, `base` paths, proxy settings, service workers, and bundler plugins.
- Browser-only libraries may need a Client Component or dynamically imported client-only boundary.
- Start by keeping the current router/app as a client-side shell if needed, then migrate to App Router deliberately.

### Vue/Svelte/Angular or other UI frameworks

- Produce a route and UI-state map before rebuilding.
- Recreate component behavior in React, not a line-by-line syntax translation.
- Preserve API request/response shapes and business behavior where possible.
- Use visual comparison where possible; a successful compile does not prove UI parity.

### Existing backend service

- Document API schemas, status codes, auth headers/cookies, CORS, pagination, file transfers, timeouts, webhook signature checks and error semantics.
- Treat cookies and sessions especially carefully when changing domains/origins.
- Keep existing services running through preview validation unless user explicitly authorizes a cutover.
- Any backend rewrite requires a parity checklist and explicit decision about language, runtime and data model.

## Escalate instead of guessing when

- A database provider/schema cannot be determined.
- An auth flow depends on live OAuth, payment or email provider configuration that cannot be tested safely.
- The source repo lacks critical generated code or required services.
- Data migration could be destructive or non-reversible.
- A third-party API or SDK does not support the target Node runtime.
- A user-visible workflow cannot be inferred from code or safely reproduced from baseline evidence.
