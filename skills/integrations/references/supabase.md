# Supabase (default: database, auth, storage, realtime)

## Install

```bash
bun add @supabase/supabase-js @supabase/ssr
bun add -d supabase                 # CLI, run via bunx supabase
```

## Env (`.env.example`)

```bash
NEXT_PUBLIC_SUPABASE_URL=                   # Project Settings -> API
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=       # publishable key (older projects: anon key)
SUPABASE_SECRET_KEY=                        # secret key (older: service_role), server-only, bypasses RLS
```

Older projects use JWT keys that start with `eyJ`, under the names `NEXT_PUBLIC_SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY`. Supabase is retiring these keys by the end of 2026. During a migration, keep them working first. Then plan the switch to publishable and secret keys as its own step, with the user (the user creates the new keys in the dashboard).

## Files

```ts
// src/lib/supabase/client.ts — browser
import { createBrowserClient } from "@supabase/ssr";

export const createClient = () =>
  createBrowserClient(process.env.NEXT_PUBLIC_SUPABASE_URL!, process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!);
```

```ts
// src/lib/supabase/server.ts — Server Components, Actions, Route Handlers
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";

export async function createClient() {
  const cookieStore = await cookies();
  return createServerClient(process.env.NEXT_PUBLIC_SUPABASE_URL!, process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!, {
    cookies: {
      getAll: () => cookieStore.getAll(),
      setAll: (toSet) => {
        try {
          toSet.forEach(({ name, value, options }) => cookieStore.set(name, value, options));
        } catch {
          // Called from a Server Component: proxy.ts refreshes the session instead.
        }
      },
    },
  });
}
```

- `src/proxy.ts` (Next.js 16+; `middleware.ts` on 15 and earlier, where a `proxy.ts` is never called): refresh the session on each request, following the proxy code in Supabase's Next.js SSR guide, and redirect signed-out users away from protected routes. Exclude static files and webhook routes in `config.matcher`.
- `src/server/supabase-admin.ts`: `import "server-only";` plus a `createClient(url, SUPABASE_SECRET_KEY)` client. Use it only for admin jobs. It bypasses RLS. Secret keys are refused in browsers.
- **Auth checks on the server:** use `supabase.auth.getClaims()`, which verifies the token signature. Use `getUser()` only when you need a fresh user record from the Auth server. Never trust `getSession()` in server code, including the proxy.

## Database

- Set up: `bunx supabase init`. The user runs `bunx supabase login`. Then `bunx supabase link --project-ref <ref>` against the **dev** project.
- Schema change: `bunx supabase migration new <name>`, write the SQL, and commit `supabase/migrations/*`.
- Local database (needs Docker): `bunx supabase start`, then `bunx supabase db reset` to apply the migrations.
- `bunx supabase db push` applies to the linked remote project. On production it needs approval every time.
- **RLS is on for every table** in exposed schemas, with policies based on `auth.uid()`. A table without RLS can be read by anyone who has the publishable key.
- Types: add a script `"db:types": "supabase gen types typescript --linked > src/types/database.ts"` and pass `Database` to the clients.

## Storage and realtime

- Buckets are private by default. Serve files with signed URLs. Write storage policies like RLS.
- Realtime: subscribe in a Client Component and unsubscribe on unmount. Load the initial data on the server.

## Verify

Sign up and sign in on dev, open a protected page while signed out (you should be redirected), read and write a row as two different users (each sees only their own), then `bun run build`. Also check the database security advisor in the Supabase dashboard for tables without RLS.
