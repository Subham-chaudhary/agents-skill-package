# Migration plan

## Goal and scope

- Target framework: Next.js App Router + TypeScript
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

- [ ] Record Git status and baseline SHA.
- [ ] Preserve pre-existing changes.
- [ ] Record current commands and failures.

### Phase 1 — Target skeleton and shared shell

- [ ] Add the smallest usable Next.js structure.
- [ ] Keep existing visual system and dependencies unless incompatible.
- [ ] Verify the application starts before moving additional features.

### Phase 2 — Route/feature migration

- [ ] Migrate one coherent route/feature at a time.
- [ ] Keep source routes/APIs available while validating.
- [ ] Add/adjust tests and update the parity matrix for each slice.

### Phase 3 — Integrations and server/client boundaries

- [ ] Confirm auth/session behavior.
- [ ] Confirm database and file storage behavior.
- [ ] Verify secret and public environment variable mapping.
- [ ] Confirm external integrations/webhooks in a safe test environment.

### Phase 4 — Final verification

- [ ] Run all available quality gates.
- [ ] Verify all mapped routes or record gaps.
- [ ] Smoke-test the development preview.
- [ ] Write `VALIDATION_REPORT.md`.

### Phase 5 — Release (separate approval)

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
