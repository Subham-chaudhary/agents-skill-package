# chaldkoor-migration-kit

**Draft v0.1.0 — research-backed, audit-first agent skill for migrating AI-built web apps to Chalkdoor's Next.js standard.**

This package is intentionally a migration *workflow* rather than a code-conversion script. Existing vibe-coded applications vary widely, and migrating only the visible UI can break authentication, API behavior, persistence, uploads, payments, or production configuration.

## Package contents

- `SKILL.md` — reusable agent instructions.
- `research/vibe-coding-requirements.md` — evidence, caveats and resulting product requirements.
- `references/source-framework-playbook.md` — migration lanes based on the source architecture.
- `references/target-standard.md` — the draft target structure and conventions.
- `references/quality-gates.md` — release acceptance criteria.
- `templates/MIGRATION_ASSESSMENT.md` — current-state inventory.
- `templates/MIGRATION_PLAN.md` — route/feature migration plan.
- `templates/VALIDATION_REPORT.md` — evidence of checks and remaining risks.
- `templates/chaldkoor.migration.yaml` — machine-readable, secret-free migration state.

## How to use it

Install or copy this directory into the skill location supported by your coding agent, or provide `SKILL.md` as the skill's entry point. The format follows the open Agent Skills directory pattern: a `SKILL.md` with YAML frontmatter plus optional references and templates.

Suggested agent request:

> Use chaldkoor-migration-kit in Audit mode. Inspect this repository without changing app behavior. Produce a route/feature inventory, risk register, target structure, and a phased Next.js migration plan. Do not expose secret values or modify production services.

After review, the next request can explicitly authorize migration of an agreed phase.

## Draft scope decisions

- Target: Next.js App Router and TypeScript.
- Goal: preserve behavior first; adopt Next.js rendering patterns incrementally.
- Backend: preserve existing backends behind APIs by default until an explicit, testable decision says to migrate them.
- Styling/auth/database: preserve existing working choices initially; don't bundle unrelated replacements into the framework migration.
- Release: develop and validate in preview before production promotion.

## Important limitation

A skill standardizes agent instructions; it does not itself enforce runtime security. Chalkdoor's build and hosting control plane must independently enforce isolation, resource limits, secret injection, network rules, and deployment permissions.
