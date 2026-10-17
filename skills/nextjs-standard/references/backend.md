# Minimal backend and error handling

Keep the server thin, with three layers at most:

```text
Server Action / Route Handler   validate input (zod), check auth, map errors to responses
        ↓
src/server/<domain>.ts          business logic, plain async functions
        ↓
provider client                 Supabase / Drizzle / Stripe (server-only)
```

Do not add repositories, DI containers or service classes unless the user asks for them.

## Error rules

- **Expected errors** (invalid input, not found, forbidden, conflict) → return them as values the UI can show.
- **Unexpected errors** → log them on the server with context, then show a generic message. Never send stack traces, SQL or provider messages to the client.
- Never log secrets, tokens, full request bodies with personal data, or cookies.
- Every route segment that can fail has an `error.tsx` (`'use client'`, with a retry button). The root layout also has `global-error.tsx`.

## Server Action pattern

```ts
// src/features/notes/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";
import { requireUser } from "@/server/auth";
import { createNote } from "@/server/notes";

export type ActionResult<T> =
  | { ok: true; data: T }
  | { ok: false; error: string; fieldErrors?: Record<string, string[]> };

const Input = z.object({ title: z.string().trim().min(1).max(200) });

export async function createNoteAction(
  _prev: ActionResult<{ id: string }> | null,
  formData: FormData,
): Promise<ActionResult<{ id: string }>> {
  const parsed = Input.safeParse(Object.fromEntries(formData));
  if (!parsed.success) {
    return { ok: false, error: "Check the form.", fieldErrors: z.flattenError(parsed.error).fieldErrors };
  }

  const user = await requireUser(); // redirects to sign-in when there is no session

  try {
    const note = await createNote(user.id, parsed.data);
    revalidatePath("/notes");
    return { ok: true, data: { id: note.id } };
  } catch (err) {
    console.error("createNoteAction failed", { userId: user.id, err });
    return { ok: false, error: "Could not save the note. Try again." };
  }
}
```

The client uses it with `useActionState(createNoteAction, null)`.

## Route Handler pattern

```ts
// src/app/api/notes/route.ts
import { z } from "zod";

const Input = z.object({ title: z.string().trim().min(1).max(200) });

export async function POST(req: Request) {
  const parsed = Input.safeParse(await req.json().catch(() => null));
  if (!parsed.success) return Response.json({ error: "Invalid input" }, { status: 400 });

  try {
    // ...auth check, then call src/server/*
    return Response.json({ ok: true }, { status: 201 });
  } catch (err) {
    console.error("POST /api/notes failed", err);
    return Response.json({ error: "Internal error" }, { status: 500 });
  }
}
```

## Scheduled jobs

There is no separate worker. Use one of these:

- the platform's cron calling a Route Handler that checks a `CRON_SECRET` header, or
- the provider's scheduler (for example Supabase `pg_cron`).

Record the choice in `decisions`.
