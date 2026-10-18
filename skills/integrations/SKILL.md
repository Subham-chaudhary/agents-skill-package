---
name: chalkdoor-integrations
description: How to integrate third-party services into a Chalkdoor Next.js app. Covers env var and secret handling, server-only clients, and dev/prod separation, with one playbook per provider. Providers are Supabase (the default for database, auth and storage), Neon with Drizzle, Stripe, Clerk and Better Auth. Use when adding, configuring or debugging any external service.
---

# Integrations

## Defaults

| Need | Default | Alternatives |
|---|---|---|
| Database, auth, storage, realtime | **Supabase** (all-in-one) | Neon + Drizzle for the database |
| Auth (if not Supabase Auth) | Supabase Auth | Clerk (hosted UI), Better Auth (self-hosted, in your DB) |
| Payments | Stripe | |

Use the default unless the user asks otherwise or the app already uses an alternative. Migration: keep the existing provider. If it is not in this table (for example Firebase), keep it behind `src/server/` and ask before you replace it. Record choices in `state.yaml` `stack`.

## Playbooks (open only the one you need)

| Provider | File |
|---|---|
| Supabase | `references/supabase.md` |
| Neon + Drizzle | `references/neon.md` |
| Stripe | `references/stripe.md` |
| Clerk | `references/clerk.md` |
| Better Auth | `references/better-auth.md` |

## Rules for every provider

1. **Secrets stay with the user.** Tell the user which variable names to put in `.env.local` and where to find each value in the provider dashboard. Never ask them to paste a value into chat. To check that a variable is present without printing it, use `grep -c '^NAME=.' .env.local`.
2. **`.env.example`** lists every name with an empty value and a comment, and it is committed. `.env.local` is never committed.
3. **Public vs server:** only values that are safe in the browser get the `NEXT_PUBLIC_` prefix (URLs, publishable keys). Secret keys stay unprefixed and are read only in `src/server/`.
4. **Server-only clients:** admin, secret-key and database clients live in `src/server/<provider>.ts` with `import "server-only";`.
5. **Validate env** at startup with zod (`src/server/env.ts`, `src/lib/env.ts`). Fail with a clear message that names the missing variable.
6. **Separate dev and prod:** use separate projects, branches or test modes for development. Production values are injected by the Chalkdoor platform and never stored in files.
7. **Schema changes** go through versioned migration files in git. Applying a migration to production needs explicit approval each time.
8. **Webhooks:** verify the signature, make the handler idempotent (store event IDs), return 2xx quickly, and exclude the route from auth in `proxy.ts`.
9. **Provider CLIs** run through `bunx`. Interactive logins are done by the user (in Claude Code: `! bunx supabase login`).
10. **Never** change production auth settings, delete data or projects, or switch payment mode to live without the user's explicit approval.
