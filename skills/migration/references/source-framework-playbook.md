# Source-framework migration playbook

Choose a lane after you inspect the repository. Every lane ends on Next.js App Router, TypeScript and Bun, as one server (root `SKILL.md` rules 1–4). The lane only decides how to get there safely. These are default strategies, not hard limits.

| Source app | Recommended first move | Typical risk | Notes |
|---|---|---:|---|
| Next.js App Router, version below 16 | Upgrade first: `bunx @next/codemod@canary upgrade latest`, then apply the standard | Low | The codemod handles `middleware` → `proxy`, `next lint` → ESLint, and stabilized APIs. Also run the async request API codemod if needed. https://nextjs.org/docs/app/guides/upgrading/version-16 |
| Next.js Pages Router | Upgrade to 16, then move page groups to the App Router one at a time | Low–medium | Both routers can run side by side during the move. https://nextjs.org/docs/app/guides/migrating/app-router-migration |
| React + Vite SPA | Add Next.js carefully; first keep SPA behavior and the router if needed; then move routes one by one | Low–medium | Audit Vite env variables, aliases, asset URLs, service worker, proxies and build scripts. https://nextjs.org/docs/app/guides/migrating/from-vite |
| Create React App | Follow the Next.js CRA guide; keep a client-only entry point during the first pass when that is safer | Low–medium | Audit `REACT_APP_*`, `homepage`, service worker, proxies, custom webpack/Babel changes and public assets. https://nextjs.org/docs/app/guides/migrating/from-create-react-app |
| React app with a separate Node backend | Migrate the frontend first; keep the API boundary and the backend while you map auth, sessions, CORS and cookies | Medium | Then fold the endpoints into Route Handlers or Server Actions slice by slice, after contract tests exist. Until then, record the backend as a temporary exception in `decisions`. |
| Vue / Nuxt / Svelte / Angular | Rebuild UI route-by-route in React/Next.js; preserve service/API contracts first | Medium–high | No trustworthy mechanical source translation should be assumed. Use screenshots, route maps, tests and behavior descriptions. |
| Static HTML/CSS/JS site | Port pages and shared layout, preserve URL paths/assets/metadata; decide static vs server rendering from actual needs | Low–medium | Verify forms, external scripts, analytics, redirects, CSP, image paths and custom headers. |
| Django / Flask / Rails / Laravel / Spring or other non-Node backend | Migrate the UI separately; keep the backend behind a stable API | High | Ask the user whether to port the backend into Next.js (the one-server standard) as a later, tested project, or keep it as a recorded exception. Do not port business rules or data semantics by translating files. Preserve migrations, auth, jobs and webhooks. |
| No Git repo / incomplete code export | Establish source ownership/export and identify missing runtime services before migration | High / blocked | Do not claim complete portability when source, database schema, secrets, uploads or provider configuration are missing. |
| Monorepo / microfrontend / multiple services | Inventory package and service boundaries; define migration scope per app | High | Preserve workspace tooling and service contracts; avoid moving whole directories without dependency analysis. |

## Typical vibe-coding sources

These are common shapes. Always confirm them from the repo, not from the tool name.

| Source | Usually | Watch for |
|---|---|---|
| Lovable | React + Vite + TypeScript + Tailwind + shadcn/ui, often Supabase | Supabase keys in client code, missing RLS, a generated Supabase client file, and edge functions under `supabase/functions` |
| Bolt | Vite + React, sometimes Next.js | Mixed package managers, missing lockfile, env values committed in `.env` |
| v0 | Next.js App Router + shadcn/ui | Often only partly wired. Check for placeholder data and missing server code |
| Replit | Varies; often a React frontend plus an Express server and Postgres | A separate server process (the one-server rule applies), and Replit-only env or secrets |
| Cursor / Claude Code / Copilot projects | Anything | Read the existing `AGENTS.md` / `CLAUDE.md` / rules files, and keep their useful instructions |

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
