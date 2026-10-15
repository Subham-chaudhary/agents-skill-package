# Research: requirements for vibe-coded applications

**Prepared:** 2026-10-16  
**Scope:** People who use natural-language AI tools to build web applications, especially users who are not experienced software engineers.  
**Confidence note:** There is no single representative survey establishing a definitive ranking of all nontechnical vibe-coder needs. This document triangulates developer survey results, platform documentation, official migration guidance, and qualitative community discussions. Treat the resulting list as product requirements to validate, not a population-level ranking.

## Executive finding

A migration skill should not begin with framework-specific code changes. The most defensible requirement is to make the existing application understandable, preserve its behavior and data, and make every major change testable and recoverable. The visible UI is only part of the product; authentication, permissions, data, files, API integrations, environment configuration, and deployment behavior are just as important.

## Evidence

### 1. Generated code is often plausible but not correct

Stack Overflow's 2025 Developer Survey reports that 66% of respondents cited frustration with AI solutions that were nearly right but not quite, and 45% cited debugging AI-generated code as more time-consuming. The survey received over 49,000 responses overall, but these results are not restricted to nontechnical app builders. It also found that 46% actively distrusted AI output accuracy versus 33% who trusted it. This is evidence for verification and transparent reporting, not proof that every vibe coder shares the same pain points.

Sources:
- https://survey.stackoverflow.co/2025/
- https://survey.stackoverflow.co/2025/ai
- https://stackoverflow.co/company/press/archive/stack-overflow-2025-developer-survey/

### 2. Recovery points are a core usability pattern

Replit's version-control documentation describes Agent checkpoints as snapshots of project state, created at logical points in development, with the ability to return to a previous state. It separately recommends Git for longer-term history and collaboration. The migration kit should therefore require a Git baseline, preserve dirty work, and use a branch/checkpoint before major changes where available.

Source: https://docs.replit.com/replit-workspace/workspace-features/version-control

### 3. New framework features should be adopted incrementally

Next.js's official Vite migration guide recommends first obtaining a working Next.js application while retaining a purely client-side SPA/router if that is the least risky first step. It then guides the migration of config, entry point, assets, environment variables and scripts. This supports a staged migration rather than forcing SSR or rewriting every route on day one.

Sources:
- https://nextjs.org/docs/pages/guides/migrating/from-vite
- https://nextjs.org/docs/app/guides/migrating/from-create-react-app
- https://nextjs.org/docs/app/guides/migrating/app-router-migration

### 4. Secrets and environment handling need explicit migration checks

Next.js documents that environment variables are server-only by default; variables prefixed with `NEXT_PUBLIC_` are included in browser JavaScript at build time. Its self-hosting guide also explains how one Docker image can be promoted across environments when runtime variables are used appropriately. Therefore migration must inventory variable *names and usage*, identify public versus server-only values, preserve environment separation, and never put secret values in reports or source.

Sources:
- https://nextjs.org/docs/app/guides/self-hosting
- https://nextjs.org/docs/app/guides/environment-variables

### 5. Data schema changes should be versioned and tested

Supabase's database migration documentation recommends tracking schema changes as migration files, testing locally, and deploying migrations from version-controlled state. It warns that editing a remote schema outside the migration flow can create synchronization problems. This is a representative pattern for apps that use a managed backend; the skill should discover the app's actual database provider and keep its existing migration mechanism.

Sources:
- https://supabase.com/docs/guides/deployment/database-migrations
- https://supabase.com/docs/guides/local-development/database-migrations

### 6. Agent-specific instructions should be discoverable and task-oriented

Vercel's Agent Skills overview describes skills as reusable packages of instructions, scripts and resources. A separate Next.js engineering post describes an evaluation where a compact, always-present `AGENTS.md` documentation index outperformed an on-demand skill for certain version-specific framework questions; the authors distinguish persistent general context from skills suited to explicit workflows such as migration. This kit is a workflow-specific skill, and should not replace project-wide `AGENTS.md` conventions.

Sources:
- https://vercel.com/blog/agent-skills-explained-an-faq
- https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals

### 7. Hosting/deployment concerns show up outside code generation

Qualitative community guidance in vibe-coding discussions highlights hosting, databases, object storage, secrets, domains, CI/CD and separation between development and production. These discussions are self-selected and not statistically representative; they are best treated as discovery signals. They nevertheless align with documented platform workflows where app publication includes environment configuration, a backend/data service, and domains.

Sources:
- https://www.reddit.com/r/vibecoding/comments/1pj6ngp/how_to_deploy_your_app_to_the_web_for_beginners/
- https://www.reddit.com/r/vibecoding/comments/1wecpeb/deployment_of_webapps/
- https://docs.replit.com/replit-workspace/workspace-features/version-control

## Derived product requirements

Ranked by risk to migration quality, not by claimed population frequency:

1. **App inventory before edits.** Identify routes, workflows, data and integrations from the repository and configuration.
2. **Feature parity map.** Every existing user-facing route and critical workflow has a destination and a verification state.
3. **Recoverability.** Baseline Git SHA, no lost dirty work, dedicated branch/checkpoint for large changes, clear rollback path.
4. **Data continuity.** Preserve database schema, migration history, user records, uploads, storage paths and access rules.
5. **Authentication continuity.** Preserve providers, sessions, roles, callback URLs, redirects, password-reset and verification flows.
6. **Secret hygiene.** Inventory names and usage only; server/client boundary review; never print secret values; flag client-exposed keys.
7. **Incremental migration.** Start with the lowest-risk path; don't force SSR/Server Components just to satisfy a label.
8. **Actionable feedback.** Translate build/test failures into a plain-language summary with a next step, while retaining exact technical evidence.
9. **Previews before production.** Run smoke tests in an isolated preview; require explicit authorization for production/data/DNS/billing changes.
10. **Predictable project structure.** Separate routing, reusable UI, domain features, integrations and server-only logic without overengineering tiny projects.
11. **Preserve current useful choices.** Don't simultaneously replace framework, styling, auth, database, ORM and hosting unless explicitly scoped.
12. **Honest unknowns.** Clearly label untested routes, unavailable secrets/services, missing tests and manual decisions.

## How to validate these requirements with real users

For a later product research round, interview 8–12 people who built an app with Lovable, Bolt, Replit, Cursor, Claude Code or a similar tool and who have limited professional software experience. Ask them to screen-share one failed or difficult migration/deployment. Rank needs by observed frequency and harm; don't lead with a framework question. Record whether the problem was code, integrations, data, deployment, cost, security, or recovery.
