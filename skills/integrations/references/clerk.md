# Clerk (hosted auth)

Use this when the user wants ready-made sign-in UI, social logins and user management without building them.

## Install

```bash
bun add @clerk/nextjs
```

## Env (`.env.example`)

```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=   # Clerk dashboard -> API keys
CLERK_SECRET_KEY=                    # server-only
```

Use a Clerk **development** instance for dev. Production is a separate instance.

## Files

```ts
// src/proxy.ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isProtected = createRouteMatcher(["/dashboard(.*)", "/settings(.*)"]);

export default clerkMiddleware(async (auth, req) => {
  if (isProtected(req)) await auth.protect();
});

export const config = {
  matcher: [
    // Skip Next.js internals and static files
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

The file is `proxy.ts` on Next.js 16+ and `middleware.ts` on 15 and earlier; the contents are the same. Webhook routes stay public because only `isProtected` routes call `auth.protect()`.

- Root layout: put `<ClerkProvider>` **inside** `<body>`, not around `<html>`.
- Server: `const { userId } = await auth()` from `@clerk/nextjs/server`. Check it in every Server Action and Route Handler.
- UI: `<Show when="signed-out"><SignInButton /><SignUpButton /></Show>` and `<Show when="signed-in"><UserButton /></Show>`. Older SDKs use `SignedIn` / `SignedOut`; follow the installed version.

## With a database

- Store `userId` (Clerk's ID) as the owner column.
- With Supabase: set up Supabase's Clerk third-party auth integration so that RLS can read the Clerk user ID.
- Sync users through Clerk webhooks (verify the signature, as in the webhook rules) only if the app needs user rows.

## Verify

Sign up on the dev instance, open a protected route while signed out (you should be redirected), sign out, then `bun run build`.
