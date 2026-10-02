## splitter-ai

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# AGENTIC WORKFLOW & BOUNDARIES

You are an expert Next.js 16 senior engineer. I am the architect.
Before writing any code, read every file in `/.context` to understand the
product, the stack rules, and the database schema. Follow the phase order
in `.context/4-build-plan.md` — one phase per commit.

## INFRASTRUCTURE RULES (CRITICAL)

- We do NOT write custom JWT, session, or password-hashing logic.
- We do NOT copy API keys out of web dashboards by hand.
- ALL auth infrastructure is provisioned with the `clerk` CLI.

## CUSTOM SKILLS

When I type one of these commands in chat, stop what you are doing and
execute exactly the associated steps.

### /clerk-setup  (Phase 1 — infrastructure)
1. Tell me to run `clerk init --framework next` in my terminal. This one
   command creates the Clerk application, links the project, installs
   `@clerk/nextjs`, and writes the keys into `.env.local`.
2. If keys are ever missing later, tell me to run `clerk env pull`.
   Keyless mode (no keys in `.env.local`) is only a temporary bridge —
   unclaimed keyless instances expire and break sign-in at runtime. Flag
   it if the project is still keyless, and gate demos on `clerk doctor`
   passing.
3. Scaffold `proxy.ts` with `clerkMiddleware()` protecting `/dashboard`
   and `/groups`, wrap the root layout in `<ClerkProvider>`, and add
   sign-in / sign-up pages. Auth-conditional UI uses Clerk v7's
   `<Show when="signed-in">` / `<Show when="signed-out">` — the
   `SignedIn` / `SignedOut` components from your training data were
   removed in v7.
4. Finish by telling me to run `clerk doctor` to verify the integration.

### /architect-math  (Phase 4 — debt logic)
1. Enter plan mode. Do NOT write application code yet.
2. Based on `.context/3-database-schema.md`, output a markdown walkthrough
   of the debt-minimization algorithm: net balance per member, then
   greedy largest-creditor ↔ largest-debtor matching, with a worked
   example (A owes B $10, B owes C $10 ⇒ A owes C $10).
3. Wait for my explicit approval before implementing.

### /webhook-sync  (Phase 5 — user sync)
1. Write `app/api/webhooks/clerk/route.ts` using `verifyWebhook` from
   `@clerk/nextjs/webhooks` (NOT hand-rolled svix verification).
2. Handle `user.created`, `user.updated`, `user.deleted` with idempotent
   upserts/deletes into the Prisma `User` table.
3. Tell me to expose localhost with a tunnel and register the endpoint
   with `clerk api` (or the Dashboard), then set
   `CLERK_WEBHOOK_SIGNING_SECRET` in `.env.local`.

---
> Source: [kulkarniankita/splitter-ai](https://github.com/kulkarniankita/splitter-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
