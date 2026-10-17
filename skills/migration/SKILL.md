---
name: chalkdoor-migration
description: Move an existing vibe-coded app, in any framework or state, to the Chalkdoor standard without losing behavior, data or history. Covers the repo strategy (same or new repo), inventory, baseline, plan, and ordered transitions of the version-control state, Bun, Next.js, integrations and UI/UX, then validation and cutover. Use when state.yaml says segment migration.
---

# Migration

Goal: move the app from its current state to the Chalkdoor **dev state** (Next.js + Bun + structure), **version-control state** (git rules, private GitHub) and **UI/UX state**, one reversible step at a time. Never do a blind rewrite.

## 0. Ask once

In one message, each with a recommendation. Record the answers in `state.yaml` and `chalkdoor_agent/migration.yaml`:

1. **Repo:** migrate in this repo on a new branch (recommended), or into a new repo?
2. **GitHub control mode**, if it is not set yet (`skills/github`).
3. **UI:** keep the current look (recommended for the first pass), or move to a design system afterwards?
4. **Off-limits:** anything that must not change (URLs, a provider, a page)?
5. **Production:** is the app live? Where, and with real users or data?

## 1. Protect the current state

- Check `git status`. If there is uncommitted work, do not stash, discard or reset it. Ask the user to commit it, or (with permission) commit it as `chore: snapshot before Chalkdoor migration`.
- No git repo yet: `git init -b main`, set up `.gitignore` first (`skills/git-workflow`), then commit the snapshot.
- Record `baseline_sha` (`git rev-parse HEAD`).
- **Same repo:** create `migrate/chalkdoor` from the baseline. All migration commits go on top. Never rewrite earlier commits unless the user asks (`skills/git-workflow` § History).
- **New repo:** create it private (`skills/github`), then ask:
  - **Keep history:** `git remote rename origin legacy`, add the new `origin`, then push. The old repo stays untouched.
  - **Fresh start:** copy the tree without `.git`, `node_modules`, `.env*` and `chalkdoor_agent`, then make a first commit `migrate: import <source repo> at <baseline sha>`.

## 2. Inventory (read only)

Fill `chalkdoor_agent/migration.yaml` and `chalkdoor_agent/reports/MIGRATION_ASSESSMENT.md` (template: `templates/migration/`). Use the code, configs and lockfiles as evidence, never the UI or the README alone. Record:

- the framework, versions, package manager, and build, start and test commands,
- routes, layouts, redirects, query behavior, and the main user journeys with their states,
- auth (provider, session, roles, callbacks), databases and migrations, access rules (RLS), and storage,
- integrations, webhooks, email, payments, jobs and realtime,
- env var **names** and where they are used (never values). Flag secrets that reach client code,
- UI library, assets, SEO, and existing tests and their status.

## 3. Baseline

Run only the commands you understand, with the **current** package manager. Record what passes and what fails before you change anything. No tests? Write a manual smoke list of the critical journeys. Do not claim visual parity without a browser tool.

## 4. Plan

Fill `reports/MIGRATION_PLAN.md`. Choose the lane from `references/source-framework-playbook.md`. Map every route and feature to a target, with one action each: `keep`, `wrap`, `port`, `rewrite`, `retire-with-approval` or `unknown`. Show the user the plan summary (5–10 lines) and get a go-ahead.

## 5. Transitions (in this order, each its own commits)

| # | Transition | What | Commit type |
|---|---|---|---|
| 1 | **Version control** | `.gitignore`, `.gitattributes`, `.env.example`; untrack ignored files; scan the full history for secrets and report the count only (if any are found, the user rotates them first). Branch and commit rules apply from now on. | `chore:` |
| 2 | **Bun** | `bun install` converts `package-lock.json`, `yarn.lock` (v1, and Berry when exact) and `pnpm-lock.yaml` (v7+) to `bun.lock` and keeps the locked versions. Remove the old lockfile and run scripts through `bun run`. The app must behave exactly as at baseline. | `build:` |
| 3 | **Next.js** | Follow the lane, route by route. App Router, TypeScript (`skills/nextjs-standard` § Versions), structure, scripts, `output: "standalone"`, health route. Existing Next.js under 16: upgrade first with `bunx @next/codemod@canary upgrade latest` (it handles `middleware` → `proxy` and `next lint` → ESLint). | `migrate:` per slice |
| 4 | **Integrations** | Keep the providers. Move secret clients into `src/server/`. Map env names (for example `VITE_X` → `NEXT_PUBLIC_X` only if it is truly public). Follow `skills/integrations`. | `migrate:` |
| 5 | **UI/UX** | Parity first, with the same look. A design-system switch or UX fixes come only if the user agreed, as separate slices (`skills/ui-ux`). | `style:` / `feat:` |
| 6 | **Cache Components** (optional) | Later, only with agreement (`skills/nextjs-standard/references/rendering.md`). | `refactor:` |

Rules for every slice:

- Small and reviewable. Keep the legacy code until the slice passes its checks.
- Do not change framework, database, auth, UI library and business logic in the same step.
- Run the done check (`skills/nextjs-standard`), update the plan and `migration.yaml`, then commit.
- Production databases, deploys, DNS, billing and live provider settings need explicit approval each time.

## 6. Validate

Run the gates in `references/quality-gates.md`. Fill `reports/VALIDATION_REPORT.md`. Never mark a gate `pass` without evidence.

## 7. Cutover

1. Merge `migrate/chalkdoor` into `main` (a PR in `full` mode) after the user agrees.
2. Change the README line to its "migrated" form (`templates/project/readme-line.md`). Set `segment: standard`.
3. Deploying to production is a separate action and needs explicit approval. Record the rollback point (the baseline SHA, the last release).

## 8. Report in plain language

Cover: what the app used before, what changed and why, what was verified (with the commands), what is left or still on the old setup, the decisions the user needs to make, and how to preview and roll back.
