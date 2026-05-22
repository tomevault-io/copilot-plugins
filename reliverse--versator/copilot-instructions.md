## clerk-auth

> this codebase uses @clerk for authentication and user management, integrated with:


# clerk-auth (llm guidance)

## overview
this codebase uses @clerk for authentication and user management, integrated with:
- drizzle orm for syncing user data to the database
- svix for webhook delivery, signature verification, and keeping clerk data in sync with drizzle

## best practices for llms

### 1. type safety
- always use official clerk types (e.g., `UserJSON`, `WebhookEvent`) from `@clerk/nextjs/server` for webhook payloads and user objects.
- when mapping clerk user data to the db, use the local `NewUser` type for inserts/updates.

### 2. webhook handling
- all webhook endpoints must verify the svix signature before processing. use the `Webhook` class from the `svix` npm package and the `CLERK_WEBHOOK_SECRET` env variable.
- read the raw request body (not parsed json) for signature verification.
- only parse and process the event after successful verification.
- handle only relevant event types (e.g., `user.created`, `user.updated`). ignore or log others.

### 3. drizzle integration
- use drizzle's upsert pattern (`.onConflictDoUpdate`) for idempotency on repeated webhooks.
- map clerk user fields to the db schema using a dedicated mapping function (see `mapClerkUserToDb`).
- always use the `NewUser` type for db inserts/updates for type safety.

### 4. security
- never trust incoming webhook payloads until signature verification passes.
- log and return 401 for invalid signatures.
- log all errors for debugging and monitoring.

### 5. code style
- follow lowercase comment style and comment the "why" for non-obvious logic, per codebase rules.
- keep mapping and verification logic in separate, testable functions.

## event schema and robustness
- clerk webhook events follow a documented schema (see clerk docs), and the official types (`WebhookEvent`, `UserJSON`) match this schema.
- the handler is robust to extra or missing fields: it only maps what is needed for the db, so new fields from clerk will not break the code.
- to use more fields from clerk, add them to the mapping function and update the db schema as needed.

## adding new event types
- to handle more events (e.g., `user.deleted`), add a new case in the switch statement in the webhook handler and implement the corresponding db logic.

## local development and endpoint url
- for local testing, use a tunneling service (like ngrok) to expose your local webhook endpoint to clerk.
- in production, the endpoint url must be a public https url (not localhost).

## error handling
- always log errors and return appropriate http status codes (e.g., 401 for invalid signature, 400 for invalid event, 500 for internal errors).

## example: webhook handler (summary)
```ts
export async function POST(req: Request) {
  const rawBody = await getRawBody(req);
  const isValid = await verifyClerkSignature(req, rawBody);
  if (!isValid) return NextResponse.json({ error: "invalid signature" }, { status: 401 });
  const event = JSON.parse(rawBody.toString()) as WebhookEvent;
  // ... handle event, map user, upsert with drizzle ...
}
```

## see also
- clerk docs: https://clerk.com/docs/
- svix docs: https://docs.svix.com/receiving/verifying-payloads/how
- drizzle docs: https://orm.drizzle.team/docs/

---
> Source: [reliverse/versator](https://github.com/reliverse/versator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
