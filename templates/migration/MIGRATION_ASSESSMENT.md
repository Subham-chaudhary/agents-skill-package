# Migration assessment

> Chalkdoor migration report. Lives in `chalkdoor_agent/reports/` (local only). Do not include secret values. Replace `Unknown` with evidence or a clearly stated gap.

## Summary

- Project/repository:
- Date:
- Repo strategy: same repo (`migrate/chalkdoor`) / new repo (keep history / fresh)
- Source branch / baseline SHA:
- Source framework(s), versions, package manager:
- Origin tool (Lovable, Bolt, v0, Replit, other):
- Target: Next.js App Router + TypeScript + Bun, one server, Chalkdoor platform
- Migration lane:
- Overall risk: Low / Medium / High / Blocked
- Current build status: Pass / Fail / Not run

## 1. Source inventory

| Area | Current state | Evidence/file(s) | Confidence / unknowns |
|---|---|---|---|
| Runtime & package manager | | | |
| Framework / router | | | |
| Build / start commands | | | |
| Styling / UI library | | | |
| Tests / lint / types | | | |
| Hosting/deployment | | | |

## 2. Route and workflow inventory

| Route / entry | Purpose | Auth required/role | Data/API dependencies | Key states | Destination | Risk |
|---|---|---|---|---|---|---|
| `/` | | | | | | |

## 3. Data, authentication and integrations

- Auth provider/session/cookies/roles:
- Database and schema/migration path:
- Data access rules / RLS / permissions:
- Upload/object storage:
- External APIs and integrations:
- Webhooks:
- Email/SMS:
- Payments/billing:
- Analytics/realtime/jobs:

## 4. Environment variable inventory (names only)

| Name | Used in file/module | Server-only or public | Required in dev/prod | Secret? | Migration action |
|---|---|---|---|---|---|

Never paste values here.

## 5. UI, assets and external behavior

- Design system / tokens:
- Fonts/icons/images/favicon:
- URL paths / redirects / query semantics:
- SEO/metadata/robots/sitemap:
- PWA/service worker/browser-only libraries:
- Localization/accessibility/responsive requirements:

## 6. Baseline checks

| Check | Command/tool | Result | Notes |
|---|---|---|---|
| Install | | | |
| Lint | | | |
| Type check | | | |
| Unit/integration tests | | | |
| Build | | | |
| Critical route smoke tests | | | |

## 7. Risks and open questions

| Risk/question | Severity | Evidence | Safe recommendation | Needs user decision? |
|---|---|---|---|---|

## 8. Recommended migration sequence

1.
2.
3.

## 9. Explicit non-goals for this migration

- No production data mutation without approval.
- No provider, authentication or database replacement unless separately agreed.
- No unrelated dependency modernization.
