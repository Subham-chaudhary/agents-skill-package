# Migration plan

## Goal and scope

- Target: Next.js App Router + TypeScript + Bun, one server (Chalkdoor standard)
- Repo strategy and migration branch:
- Source framework:
- Included routes/features:
- Excluded routes/services:
- Backend retained behind API? Why?
- Constraints / user-approved decisions:

## Architecture decision log

| Decision | Chosen approach | Alternatives considered | Reason/evidence | Reversible? |
|---|---|---|---|---|

## Route / feature parity matrix

| Feature or route | Source behavior / API contract | Target location | Action (`keep`, `wrap`, `port`, `rewrite`, `retire-with-approval`, `unknown`) | Tests | Status |
|---|---|---|---|---|---|

## Data and integration plan

| System | Current behavior | Planned treatment | Environment(s) | Test method | Risk / rollback |
|---|---|---|---|---|---|

## Phases

### Phase 0 — Preservation and baseline

- [ ] Record Git status, baseline SHA and repo strategy.
- [ ] Preserve pre-existing changes (snapshot commit only with permission).
- [ ] Record current commands and failures with the current package manager.

### Phase 1 — Version-control state

- [ ] `.gitignore`, `.gitattributes`, `.env.example`; untrack ignored files.
- [ ] Full-history secret scan (count only); the user rotates any secrets found.

### Phase 2 — Bun

- [ ] `bun install` migrates the lockfile; locked versions match the baseline.
- [ ] Old lockfile removed; scripts run through `bun run`; behavior unchanged.

### Phase 3 — Next.js, route by route

- [ ] Smallest usable Next.js App Router shell (or an upgrade to Next.js 16 first).
- [ ] Keep the existing visual system and dependencies unless incompatible.
- [ ] Migrate one coherent route or feature at a time; keep the legacy code until it passes.
- [ ] Add or adjust tests and update the parity matrix for each slice.

### Phase 4 — Integrations and server/client boundaries

- [ ] Confirm auth/session behavior.
- [ ] Confirm database and file storage behavior.
- [ ] Verify secret and public environment variable mapping.
- [ ] Confirm external integrations/webhooks in a safe test environment.

### Phase 5 — UI/UX (only if agreed)

- [ ] Design-system adoption and UX fixes as separate slices.

### Phase 6 — Final verification

- [ ] Run all available quality gates.
- [ ] Verify all mapped routes or record gaps.
- [ ] Smoke-test the development preview.
- [ ] Write `VALIDATION_REPORT.md`.

### Phase 7 — Cutover and release (separate approvals)

- [ ] Merge `migrate/chalkdoor` into `main` after the user agrees; update the README line.
- [ ] Record the source SHA/build artifact.
- [ ] Confirm rollback path.
- [ ] Obtain explicit production promotion approval.
- [ ] Promote only the verified release; monitor after cutover.

## Acceptance criteria

- Critical workflows:
- Required routes:
- Authentication/roles:
- Data integrity:
- Visual/responsive expectations:
- Security requirements:
- Required build/test checks:

## Risks / blockers

| Item | Severity | Mitigation | Owner / decision needed |
|---|---|---|---|
