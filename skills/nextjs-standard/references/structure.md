# Project structure

Fit the structure to the app's size. Do not create empty folders.

```text
src/
  app/                        # routes only: page, layout, loading, error, not-found, route
    (marketing)/              # route groups when sections need different layouts
    api/health/route.ts       # GET -> { status: "ok" } for the platform
    layout.tsx
    page.tsx
    globals.css
  components/
    ui/                       # design-system primitives (for example shadcn/ui copies)
  features/<feature>/         # once 2+ files belong to one feature
    components/
    actions.ts                # Server Actions ("use server")
    queries.ts                # server-side reads
    schema.ts                 # zod schemas and types
  lib/                        # shared helpers safe on client and server (utils, env)
  server/                     # server-only: db, provider admin clients, services
  types/                      # shared types and generated DB types
  proxy.ts                    # only if needed (auth redirects)
public/                       # small static assets only
tests/e2e/                    # Playwright; unit tests sit next to the code as *.test.ts
```

## Rules

- `app/` holds routing. Put logic in `features/` and `server/`.
- Every file in `src/server/` starts with `import "server-only";`.
- `src/lib/` must be safe to import from client code. Anything with a secret goes in `src/server/`.
- Use one env module: `src/lib/env.ts`, which validates public variables, and `src/server/env.ts`, which validates server variables with zod. Read `process.env` only there.
- Health route:

```ts
// src/app/api/health/route.ts
import { connection } from "next/server";

export async function GET() {
  await connection(); // answer per request, not from the build
  return Response.json({ status: "ok" });
}
```

Without `connection()`, the handler is prerendered at build time and does not prove that the server is alive.
