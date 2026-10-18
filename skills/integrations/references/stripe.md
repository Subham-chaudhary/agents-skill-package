# Stripe (payments)

The default is **Stripe Checkout** (a hosted payment page): the least code and the least PCI scope. Use Elements only if the user needs an embedded form.

## Install

```bash
bun add stripe
bun add @stripe/stripe-js     # only for Elements or embedded checkout
```

## Env (`.env.example`)

```bash
STRIPE_SECRET_KEY=                    # sk_test_... in dev, server-only
STRIPE_WEBHOOK_SECRET=                # whsec_..., server-only
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=   # pk_test_..., only if using stripe-js
```

Development always uses **test mode** keys. Switching to live keys is a production change and needs approval.

## Files

```ts
// src/server/stripe.ts
import "server-only";
import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
```

Checkout through a Server Action: call `stripe.checkout.sessions.create({ mode, line_items: [{ price, quantity: 1 }], success_url, cancel_url, client_reference_id: user.id })`, then `redirect(session.url!)`. Keep price IDs in config or the database, never prices sent from the client.

```ts
// src/app/api/webhooks/stripe/route.ts
import type Stripe from "stripe";
import { stripe } from "@/server/stripe";

export async function POST(req: Request) {
  const signature = req.headers.get("stripe-signature");
  if (!signature) return new Response("Missing signature", { status: 400 });

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(await req.text(), signature, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch {
    return new Response("Invalid signature", { status: 400 });
  }

  // Idempotency: skip if event.id was already processed.
  switch (event.type) {
    case "checkout.session.completed":
      // fulfil the order from event.data.object
      break;
  }
  return Response.json({ received: true });
}
```

## Rules

- Fulfil orders from the webhook, never from the success page.
- Read the raw body with `req.text()` before verifying. Do not parse the JSON first.
- Exclude `/api/webhooks/*` from auth redirects in `proxy.ts`.
- Local webhooks: the user installs the Stripe CLI and runs `stripe listen --forward-to localhost:3000/api/webhooks/stripe`. It prints a `whsec_` value, which the user puts in `.env.local` themselves.

## Verify

Pay in test mode with card `4242 4242 4242 4242`, check that the webhook arrives and the order is marked paid, replay the event and check that nothing is duplicated, then `bun run build`.
