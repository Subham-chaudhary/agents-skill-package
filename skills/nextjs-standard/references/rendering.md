# Rendering decisions

## Which model the app uses

- **New projects:** `create-next-app` turns on **Cache Components** (`cacheComponents: true`, `partialPrefetching: true`). Keep both. Each route gets a static shell, and the dynamic parts stream in.
- **Migrations:** do not enable Cache Components as part of the migration. It is a separate, later slice, done only with user agreement (it surfaces build errors for uncached data outside `<Suspense>`). Without it, the previous model applies: reading `cookies()`, `headers()` or `searchParams` makes the whole route dynamic.
- Record the model in `state.yaml` `decisions`.

## Pick the lightest option

| Need | Use | Cache Components notes |
|---|---|---|
| Same for everyone (landing, docs, pricing) | Server Component | Prerendered automatically. |
| Shared data that changes now and then (catalog, posts) | `'use cache'` + `cacheLife('hours')` + `cacheTag('posts')` | Becomes part of the static shell. Always pair `use cache` with a `cacheLife`. |
| Per-user or per-request data (dashboard, account) | Server Component inside `<Suspense fallback={...}>` | Required. A runtime API (`cookies`, `headers`, `searchParams`, dynamic `params`) or an uncached fetch outside `<Suspense>` gives a "blocking-route" error. |
| A unique value per request (`Date.now()`, `Math.random()`, `crypto.randomUUID()`) | `await connection()` first, inside `<Suspense>` | Or `use cache` to share one value. |
| Interactivity (live form state, menus, charts, maps) | Client Component leaf | `'use client'` on the smallest component. The page stays a Server Component. |
| Form submit or mutation from your own UI | Server Action | Validate with zod and check auth in the action. Then refresh the data (table below). |
| Webhooks, callbacks, public JSON API, health | Route Handler `app/**/route.ts` | `GET` handlers are prerendered like pages. Call `await connection()` when the answer must be live. |
| Realtime updates | Client Component subscription | For example Supabase Realtime. The initial data comes from the server. |
| Auth redirects or headers before a route renders | `src/proxy.ts` | Called `middleware.ts` before Next.js 16. Node.js runtime only. Keep it thin. |
| Work after the response (logging, emails) | `after()` from `next/server` | |

Push `await` (and the `<Suspense>` around it) as deep in the tree as you can, so that more of the page stays in the static shell. Pass the `params` promise down instead of awaiting it in the layout.

## Cache refresh after a mutation (Next.js 16)

| Need | API |
|---|---|
| The user must see their own change right away (forms, settings) | `updateTag(tag)` (Server Actions only) |
| A short delay is fine (posts, catalog) | `revalidateTag(tag, "max")`. The second argument is required in Next.js 16. |
| Refresh a whole path | `revalidatePath(path)` |
| Refresh client UI that is not tied to a tag | `refresh()` |

## Rules

- Fetch on the server. Run independent fetches in parallel with `Promise.all`. Do not fetch your own Route Handlers from Server Components; call the server function directly.
- Pass only serializable props from server to client. Never pass secrets, and never import from `src/server/` in a client file.
- Add `loading.tsx`, `error.tsx` and `not-found.tsx` to a segment when the user would otherwise see a blank screen or a crash.
- Set `metadata` / `generateMetadata` on every public page.
- Server Actions are public endpoints. Check auth in each one. A proxy matcher change can silently remove proxy coverage.
- `use cache` entries live in memory per instance by default. Do not rely on them persisting across deploys or instances.
- Images go through `next/image` and fonts through `next/font`. Add remote image hosts to `images.remotePatterns`.
- When unsure, read the docs bundled with the installed version: `node_modules/next/dist/docs/` (the caching and cache-components pages).
