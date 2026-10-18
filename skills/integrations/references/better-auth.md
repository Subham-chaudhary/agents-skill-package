# Better Auth (self-hosted auth in your own database)

Use this when the user wants auth data in their own Postgres (usually Neon + Drizzle) and no hosted auth vendor.

## Install

```bash
bun add better-auth
```

## Env (`.env.example`)

```bash
BETTER_AUTH_SECRET=   # random 32+ bytes; the user generates it with: bunx auth@latest secret
BETTER_AUTH_URL=      # http://localhost:3000 in dev
```

## Files

```ts
// src/server/auth.ts
import "server-only";
import { betterAuth } from "better-auth";
import { drizzleAdapter } from "better-auth/adapters/drizzle";
import { nextCookies } from "better-auth/next-js";
import { db } from "@/server/db";

export const auth = betterAuth({
  database: drizzleAdapter(db, { provider: "pg" }),
  emailAndPassword: { enabled: true },
  plugins: [nextCookies()], // lets Server Actions set auth cookies
});
```

```ts
// src/app/api/auth/[...all]/route.ts
import { toNextJsHandler } from "better-auth/next-js";
import { auth } from "@/server/auth";

export const { GET, POST } = toNextJsHandler(auth);
```

```ts
// src/lib/auth-client.ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient();
```

- Server session: `await auth.api.getSession({ headers: await headers() })`. Check it in every Server Action and Route Handler.
- Schema: run `bunx auth@latest generate` to create the Drizzle tables, then follow the migration flow in `neon.md`.
- `proxy.ts`: for a fast redirect, check only the cookie with `getSessionCookie(request)` from `better-auth/cookies`. That check is optimistic and not secure, so every page, action and route still validates with `auth.api.getSession` on the server.

## Verify

Sign up, sign out, sign in, open a protected route while signed out (you should be redirected), check that the session rows exist in the dev database, then `bun run build`.
