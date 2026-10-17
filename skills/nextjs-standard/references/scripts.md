# Scripts and build

## package.json scripts

Merge `templates/project/package.scripts.json` into `package.json`:

| Script | Command | Use |
|---|---|---|
| `dev` | `next dev` | Local dev server (Turbopack) |
| `build` | `next build` | Production build (Turbopack) |
| `start` | `next start` | Serve the build locally |
| `lint` | `eslint .` | Lint. Next.js 16 removed `next lint`, and `next build` no longer lints |
| `lint:fix` | `eslint . --fix` | |
| `typecheck` | `next typegen && tsc --noEmit` | `typegen` writes the route types (`PageProps`, `LayoutProps`) without a full build |
| `test` | `bun test` | Add once the first test exists (`bun test` fails when there are no tests) |
| `test:e2e` | `playwright test` | Only if Playwright is set up |
| `check` | `bun run lint && bun run typecheck && bun run build` | Run before merging into `main` |

Always run scripts with `bun run <name>`. Add integration scripts (for example `db:*`) from `skills/integrations`. Keep the list short; delete scripts nobody uses.

## next.config.ts

Keep what `create-next-app` generated and add `output: "standalone"`:

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "standalone", // Chalkdoor platform serves the standalone server
  cacheComponents: true, // new projects only, see rendering.md
  partialPrefetching: true,
  turbopack: {
    rules: {
      "*.css": { loaders: ["@tailwindcss/turbopack"], as: "*.css" },
    },
  },
};

export default nextConfig;
```

- Add `images.remotePatterns` for remote image hosts.
- Do not add `experimental` flags unless the user agrees and the decision is recorded.
- Keep the `packageManager`, `ignoreScripts` and `trustedDependencies` fields that `create-next-app --use-bun` writes. They pin Bun and allow only the native install scripts that Next.js needs (`sharp`, `unrs-resolver`).
- If Next.js warns that it found a lockfile in a parent folder (often a stray `package-lock.json` in the home folder), set `outputFileTracingRoot` to the project folder.

## Build and release

- Local check: `bun run check`. Then `bun run start` and open `/` and `/api/health`. With `output: "standalone"`, `next start` prints a warning; that is fine for a local check.
- The Chalkdoor platform runs the production build and injects production env vars. Never put production values in files.
- `NEXT_PUBLIC_*` values are inlined at **build** time. Server values on prerendered (static) pages are also read at build time. If a page must read a server env var at **runtime**, call `await connection()` (from `next/server`) before reading `process.env`.
- `next dev` writes to `.next/dev` and `next build` to `.next`, so both can run at the same time.
- Deploying or promoting to production needs the user's explicit approval each time.
