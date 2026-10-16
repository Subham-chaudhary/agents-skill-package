# Migration quality gates

A gate can be `pass`, `fail`, `blocked`, `not-run`, or `not-applicable`. Never mark a gate `pass` without evidence.

## Gate 0 — Source preservation

- [ ] Repository, branch and baseline commit are recorded where available.
- [ ] The repo strategy (same repo on `migrate/chalkdoor`, or a new private repo) is recorded.
- [ ] Pre-existing uncommitted changes are preserved.
- [ ] Earlier commits were not rewritten, or the user asked for it and a backup branch exists.
- [ ] Migration changes are isolated, or the reason they are not is documented.
- [ ] No production resource has been changed without explicit approval.

## Gate 1 — Inventory completeness

- [ ] All routes/pages and important route states are inventoried.
- [ ] Critical user journeys are listed.
- [ ] Auth methods, roles, session/cookie behavior and callbacks are recorded.
- [ ] Data stores, schema migrations, data access rules and file storage are recorded.
- [ ] Third-party APIs, webhooks, emails, payments, analytics, jobs and realtime behavior are recorded.
- [ ] Environment-variable names and their client/server uses are inventoried without exposing values.
- [ ] Assets, metadata, URL redirects and hosting assumptions are recorded.

## Gate 2 — Application behavior

- [ ] Every in-scope route has a destination or documented approved retirement.
- [ ] Critical journeys work end-to-end in the preview.
- [ ] Loading, empty, success, validation, unauthorized and error states are tested where applicable.
- [ ] Auth/session behavior and authorization boundaries are tested.
- [ ] Data reads/writes, uploads, integrations and webhooks in scope are tested safely.
- [ ] Links, deep links, query parameters, redirects and not-found routes behave as intended.
- [ ] Mobile/responsive layout and major accessibility behaviors are checked.

## Gate 3 — Code and build

- [ ] `bun install` succeeds, `bun.lock` is the only lockfile, and the locked versions match the baseline (except for agreed upgrades).
- [ ] Next.js App Router + TypeScript, `output: "standalone"`, and a live `/api/health` route.
- [ ] `.gitignore`, `.gitattributes` and `.env.example` are in place, and `chalkdoor_agent/` is ignored.
- [ ] Lint passes or baseline/new failures are explicitly distinguished.
- [ ] Type-check passes or failures are reported with context.
- [ ] Tests pass or missing tests are acknowledged.
- [ ] Production build succeeds.
- [ ] Application starts using the target runtime and responds to its health endpoint/check.
- [ ] No unrelated forced dependency upgrades or lockfile churn occurred.

## Gate 4 — Security and data

- [ ] No secret values entered source, commits, client bundles, migration reports or logs.
- [ ] The full-history secret scan was run. Any secrets found were rotated by the user.
- [ ] The GitHub repo is private, unless the user confirmed public twice.
- [ ] Server-only service clients are not importable from Client Components.
- [ ] Authorization is enforced server-side, not solely by hiding UI.
- [ ] External inputs are validated server-side.
- [ ] Database schema changes are versioned and tested on a non-production database.
- [ ] Production data is not reset, overwritten or migrated without explicit approval.
- [ ] Required resource/network/tenant controls are enforced by the hosting platform, not merely described in code.

## Gate 5 — Release readiness

- [ ] Development/preview URL is healthy and smoke-tested.
- [ ] Runtime environment variables and service bindings are documented by name.
- [ ] Any schema or integration changes have an approved rollout sequence.
- [ ] Exact source SHA and artifact/build ID are recorded where available.
- [ ] Rollback procedure and previous release are identified.
- [ ] Production promotion is explicitly authorized and is a separate action from migration.

## Fail/blocked response

When a gate fails, don't hide it or mark the whole task done. Report:

1. the gate and result,
2. exact failing command or observed behavior,
3. likely cause (clearly labeled as diagnosis, not fact, if unconfirmed),
4. the smallest safe next step,
5. whether user input or credentials/environment access are needed.
