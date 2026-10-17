---
name: chalkdoor-nextjs-standard
description: Chalkdoor development rules for writing app code. Covers the Next.js App Router + TypeScript single-server architecture, Bun, Turbopack, versions, package choice, SSR/CSR rendering, project structure, package.json scripts, minimal backend and error handling, and code comments. Use whenever you write or change application code.
---

# Next.js standard

## Architecture: one server

```text
browser ── Next.js server (one process) ── providers (Supabase, Neon, Stripe, ...)
            ├─ Server Components   pages, data reads
            ├─ Client Components   interactive leaves only
            ├─ Server Actions      form mutations
            ├─ Route Handlers      webhooks, public API, health
            └─ src/server/         server-only services and clients
```

Everything the app needs runs in this one Next.js app on the Chalkdoor platform. Ask the user before you add a separate backend, worker or service. If one is unavoidable, record the reason in `state.yaml` `decisions`.

If the user asks for another framework, follow root `SKILL.md` rule 1.

## Versions

- **Next.js:** the stable release (npm `latest` tag). **TypeScript:** the latest release. Check inside the project with `bun pm view next version` and `bun pm view typescript version`. Record both in `state.yaml` `stack`. Next.js 16 needs Node.js 20.9 or newer.
- **TypeScript 7 setup.** TypeScript 7 (the native compiler) has no JavaScript API yet, and typescript-eslint needs one. Install both:
  - `bun add -d "typescript@~6.0" "@typescript/native@npm:typescript@latest"`
  - `tsc` (used by `next build` and `typecheck`) then runs TypeScript 7. `require("typescript")` (used by typescript-eslint) gets the 6.x API.
  - Choose the `typescript` range from `bun pm view typescript-eslint peerDependencies`. When that range accepts 7.x, switch to a single `typescript@latest` and remove `@typescript/native`.
  - Do not use the official `@typescript/typescript6` alias with Bun. With Bun 1.3 it resolves to itself, and `require("typescript")` returns an empty module, which crashes ESLint.
  - Tested on 2026-10-19 with Next.js 16.4.2, TypeScript 7.0.2 / 6.0.3 and Bun 1.3.14: lint, typecheck and build pass.
- Upgrade on its own branch (`chore/upgrade-next-<v>`) with `bunx next upgrade` (or `bunx @next/codemod@canary upgrade latest`), never in the middle of a feature.
- For API details, trust the installed version's docs over memory. `create-next-app` adds an `AGENTS.md` (`--agents-md`, on by default) that points to the docs bundled in `node_modules/next/dist/docs/`, and `next dev` keeps it current. Keep it, and read the docs page you need.

## Bun

| Need | Command |
|---|---|
| Install from lockfile | `bun install` |
| Add a dependency / dev dependency | `bun add <pkg>` / `bun add -d <pkg>` |
| Remove | `bun remove <pkg>` |
| Run a script | `bun run <script>` |
| One-off CLI | `bunx <pkg>` |

- Never use npm, yarn, pnpm or npx. Commit `bun.lock`.
- Bun does not run dependency install scripts unless the package is in `trustedDependencies`. Add a package there only with a stated reason.
- Bun is the package manager and script runner. Next.js runs on its default Node.js runtime unless the platform says otherwise.
- **Bundler:** Turbopack, the default for `next dev` and `next build`. No Vite. Use `--webpack` only when a required plugin needs it, and record why.

## Package choice

Choose in this order: a built-in Next.js feature → a web platform API → the list in `references/packages.md` → a vetted community package (vetting rules in the same file). If the user asks for something unusual, first propose the community-standard alternative. If they still want theirs, explain the risk, pin the version and record it in `decisions`. Never install experimental, canary or unknown packages.

## References (open only for the step you are on)

| Step | File |
|---|---|
| Choosing SSR, static, client, Server Action or Route Handler | `references/rendering.md` |
| Where files go | `references/structure.md` |
| Picking or vetting a package | `references/packages.md` |
| `package.json` scripts, `next.config.ts`, build | `references/scripts.md` |
| Server logic, validation, error handling | `references/backend.md` |

## Code style

- TypeScript `strict`. No `any`; use `unknown` and narrow it. Validate every external input with `zod` at the boundary.
- Server Components by default. Add `'use client'` only on leaves that need state, effects, event handlers or browser APIs.
- Files are kebab-case, components PascalCase, and imports use the `@/` alias.
- **Comments are short and written for the next reader, often an LLM.** Comment only intent, invariants, constraints and non-obvious "why". Do not narrate what the code says, leave no commented-out code, and write a `TODO` only with a reason. One line is usually enough.

## Done check per change

1. `bun run lint && bun run typecheck`, plus `bun run test` if tests exist.
2. `bun run build` before merging into `main`.
3. Check the affected route in the browser if a browser tool is available. If not, say it was not checked.
4. Update `state.yaml` `checks` and `features`.
