## no-name

> Third-party integrations (Stripe, monitoring, infrastructure)


# Third-Party Integrations

## Stripe Integration

### Official Libraries (Use These)

```typescript
// Installation
npm install @stripe/react-stripe-js @stripe/stripe-js

// Setup
import { Elements } from '@stripe/react-stripe-js'
import { loadStripe } from '@stripe/stripe-js'

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!)

export function CheckoutPage({ clientSecret }) {
  return (
    <Elements stripe={stripePromise} options={{ clientSecret }}>
      <CheckoutForm />
    </Elements>
  )
}
```

### Webhooks via Inngest (REQUIRED)

```typescript
// app/api/webhooks/stripe/route.ts
export async function POST(request: Request) {
  const event = stripe.webhooks.constructEvent(body, signature, webhookSecret);

  await inngest.send({
    name: "stripe/webhook.received",
    data: { type: event.type, event },
  });

  return Response.json({ received: true });
}

// inngest/stripe.ts
export const handleCheckoutCompleted = inngest.createFunction(
  { id: "stripe-checkout-completed", retries: 3 },
  { event: "stripe/webhook.received" },
  async ({ event, step }) => {
    if (event.data.type !== "checkout.session.completed") return;

    await step.run("update-subscription", async () => {
      await db.users.update({
        /* ... */
      });
    });

    await step.run("send-email", async () => {
      await resend.emails.send({
        /* ... */
      });
    });
  }
);
```

## Infrastructure Services

### Required Services

- **Infisical** - Secrets management (open source)
- **Vercel KV** - Rate limiting (Redis)
- **Resend** - Transactional emails
- **Minio** - File storage (self-hosted S3)
- **Inngest** - Background jobs, Stripe webhooks, cron jobs

### Monitoring

- **Sentry** - Error tracking (automatic, no manual logging needed)
- **PostHog** - Analytics, session replay, feature flags
- **Axiom** - Structured logging (500GB/month free)

## Environment Variables

```bash
# Database
DATABASE_URL=
SUPABASE_URL=
SUPABASE_ANON_KEY=

# Auth
BETTER_AUTH_SECRET=          # 32+ chars
SESSION_ENCRYPTION_KEY=      # 64 hex chars (32 bytes)
GOOGLE_CLIENT_ID=

# Payments
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_WEBHOOK_SECRET=

# Infrastructure
RESEND_API_KEY=
KV_URL=
MINIO_ENDPOINT=
INNGEST_EVENT_KEY=

# Monitoring
SENTRY_DSN=
NEXT_PUBLIC_POSTHOG_KEY=
AXIOM_TOKEN=
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrielmouallem)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gabrielmouallem)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
