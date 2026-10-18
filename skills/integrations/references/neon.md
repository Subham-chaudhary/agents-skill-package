# Neon + Drizzle (serverless Postgres)

Use this when the user picks Neon, or when the app needs Postgres without Supabase. Auth then comes from Clerk or Better Auth.

## Install

```bash
bun add @neondatabase/serverless drizzle-orm
bun add -d drizzle-kit @next/env
```

## Env (`.env.example`)

```bash
DATABASE_URL=            # Neon pooled connection string (host contains -pooler), server-only
DATABASE_URL_UNPOOLED=   # direct connection, used by drizzle-kit for migrations
```

Use a Neon **dev branch** for development. Production gets its own connection string, injected by the platform.

## Files

```ts
// src/server/db/index.ts
import "server-only";
import { neon } from "@neondatabase/serverless";
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "./schema";

export const db = drizzle({ client: neon(process.env.DATABASE_URL!), schema });
```

```ts
// drizzle.config.ts
import { loadEnvConfig } from "@next/env";
import { defineConfig } from "drizzle-kit";

loadEnvConfig(process.cwd()); // drizzle-kit runs outside Next.js, so load .env.local the same way Next does

export default defineConfig({
  schema: "./src/server/db/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: { url: process.env.DATABASE_URL_UNPOOLED! },
});
```

The `neon-http` driver does not support interactive transactions. If the app needs them, use `Pool` from `@neondatabase/serverless` with `drizzle-orm/neon-serverless` (`drizzle({ client: pool })`). That driver needs the `ws` package on Node.js.

## Scripts

```json
"db:generate": "drizzle-kit generate",
"db:migrate": "drizzle-kit migrate",
"db:studio": "drizzle-kit studio"
```

- `bun run` does **not** load `.env.local` into scripts (tested with Bun 1.3.14). `loadEnvConfig` in `drizzle.config.ts` covers drizzle-kit. For any other CLI that needs env vars, use `bun --env-file=.env.local run <script>`.
- Flow: edit `schema.ts`, run `db:generate`, review the SQL in `drizzle/`, commit it, then run `db:migrate` on the dev branch.
- Do not use `drizzle-kit push` except for throwaway prototypes. Running migrations on production needs approval.

## Verify

Run `db:migrate` on dev, then one read and one write through a Server Action, then `bun run build`.
