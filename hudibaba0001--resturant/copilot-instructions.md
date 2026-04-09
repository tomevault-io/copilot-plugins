## resturant

> description: "Embeddable restaurant menu with AI chat + Smart Cart (pickup via Stripe)"

# .cursorrules — Stjarna MVP
# Purpose: Keep Cursor (and contributors) inside a safe, shippable MVP box.
# Scope: Next.js 14 (App Router) on Vercel · Supabase (Postgres + pgvector + Auth/RLS, EU) · Stripe · OpenAI · Sentry · Plausible.

version: 1
project:
  name: Stjarna
  description: "Embeddable restaurant menu with AI chat + Smart Cart (pickup via Stripe)"

# -------------------------------
# 1) PRODUCT GUARDRAILS (DO NOT CHANGE)
# -------------------------------
objectives:
  north_star: "New restaurant to live embed ≤ 15 minutes (onboarding KPI)."
  mvp_features:
    - Menu Creator (sections/items, allergens/tags)
    - Embeddable Menu + AI Chatbot
    - Smart Cart (order code/QR, Stripe pre-pay for pickup)
    - Opening-hours gating
  out_of_scope:
    - POS integrations, delivery, table reservations, subscriptions
    - Complex modifiers/options matrix (keep tags + free-text notes only)
    - "Insights" dashboards beyond basic logs

# -------------------------------
# 2) COMPLIANCE & SECURITY (NON‑NEGOTIABLE)
# -------------------------------
compliance:
  gdpr:
    data_residency: "All prod data in EU (Supabase EU region)."
    pii_minimization: "No diner PII stored. Use pseudonymous session_token."
    right_to_erasure: "Provide SQL helper to delete all session+order rows by restaurant scope if needed."
  audit:
    - "Enable database logging and retention per Supabase defaults."
security:
  rls: "Every table RLS-on; follow has_restaurant_permission() pattern."
  auth:
    provider: "Supabase Auth"
    roles: [owner, manager, editor, viewer]
  keys_and_secrets:
    - "Never expose service-role or Stripe secrets client-side."
    - "Edge (chat) can call Supabase with service-role only from server code."
  logging:
    - "No logs of full chat content or payment info in server console in production."
    - "Sentry allowed for errors; scrub payloads."

# -------------------------------
# 3) RUNTIME & TECH POLICY
# -------------------------------
runtime:
  edge:
    - path: /api/chat
    - reason: latency, cost
  node:
    - path: [/api/orders, /api/stripe/webhook]
    - reason: Stripe signatures/SDK
packages_allowed:
  - "@supabase/supabase-js"
  - "stripe"
  - "zod"
  - "openai"
  - "ai"           # streaming utils if needed
  - "uuid"
  - "qr-code-styling"  # client QR rendering
  - "@vercel/analytics" | "plausible-tracker"
  - "@sentry/nextjs"
packages_blocked:
  - "server-side analytics that export data outside EU"
  - "unmaintained vector/LLM wrappers"

# -------------------------------
# 4) FILES & BOUNDARIES
# -------------------------------
files_protected:
  - sql/migrations/**  # Only change via explicit migration tasks
  - app/api/stripe/webhook/route.ts  # Check signature; do not log payloads
  - public/widget.js   # Keep no external network beacons other than our APIs
client_server_contracts:
  - "Client never passes price; server is source of truth for cents/currency."
  - "Client may send sessionToken, restaurantId, and itemIds only."

# -------------------------------
# 5) DATA & DB RULES
# -------------------------------
database:
  vector:
    table: item_embeddings
    dim: 1536  # text-embedding-3-small
  pricing:
    field: price_cents  # int, authoritative
  policies:
    public_read:
      - menu_sections (active+verified)
      - menu_items (active+verified)
    private:
      - chat_sessions/messages (staff read-only; widget insert)
      - orders/order_items (staff manage; widget insert)
  rpc:
    - name: match_menu_items
      usage: "Service-role only from server; never from client."

# -------------------------------
# 6) STRIPE POLICY
# -------------------------------
stripe:
  mode:
    phase0: "Single platform Stripe account via Checkout."
    phase1: "Upgrade to Stripe Connect (Standard) per-restaurant; do NOT start until MVP is live."
  webhook:
    events: [checkout.session.completed, checkout.session.expired]
    endpoint: /api/stripe/webhook
  pii: "Never store cardholder data; store Stripe IDs only."

# -------------------------------
# 7) AI POLICY
# -------------------------------
ai:
  provider_primary: OpenAI
  models:
    chat: ${OPENAI_CHAT_MODEL:gpt-4o-mini}
    embed: ${OPENAI_EMBEDDING_MODEL:text-embedding-3-small}
  prompts:
    style: "brief, cautious on allergens, 2–4 suggestions max"
  telemetry: "Do not log prompts/responses beyond minimal success/error metrics."

# -------------------------------
# 8) TASK TEMPLATES (FOR CURSOR)
# -------------------------------
tasks:
  implement_feature:
    steps:
      - "Create/modify files as specified; keep runtimes as declared."
      - "Add Zod validation for any external input."
      - "Touch only allowed packages."
      - "Write/adjust Supabase SQL migrations instead of ad-hoc SQL."
      - "Run typecheck and build; fix trivial issues."
    acceptance:
      - "`pnpm build` passes"
      - "RLS not bypassed in client"
      - "No secret in client bundle (grep for keys)"
  fix_bug:
    steps:
      - "Reproduce with a minimal curl or unit test."
      - "Add fix with comments referencing the reproduction."
      - "Run build; keep snapshots updated."

# -------------------------------
# 9) CI / COMMANDS CURSOR MAY RUN
# -------------------------------
commands:
  install: ["pnpm i", "npm i"]
  lint: ["pnpm lint", "npm run lint"]
  typecheck: ["pnpm typecheck", "tsc -p . --noEmit"]
  build: ["pnpm build", "npm run build"]
  dev: ["pnpm dev", "npm run dev"]
  format: ["pnpm format", "npx prettier -w ."]

# -------------------------------
# 10) PR CHECKLIST (AUTO‑COMMENT)
# -------------------------------
checklist:
  - "No new third-party dependencies outside allowlist"
  - "All server handlers validate input with Zod"
  - "No secrets leaked to client bundle"
  - "RLS policies exist for any new table"
  - "EU region only; no US endpoints added"
  - "Stripe webhook events handled idempotently"
  - "Public API responses contain only non-sensitive fields"

# -------------------------------
# 11) BRANCHING & COMMITS
# -------------------------------
branching:
  naming: ["feat/*", "fix/*", "chore/*", "ops/*"]
commits:
  style: conventional
  examples:
    - "feat(chat): add menu candidate rendering"
    - "fix(orders): enforce single currency per order"

# -------------------------------
# 12) ACCEPTANCE TESTS (MVP DONE)
# -------------------------------
acceptance:
  - "New account to live embed ≤ 15 min"
  - "Chat answers vegan/nut-free/high-protein with 2–4 items"
  - "Pickup order -> Stripe Checkout -> webhook -> order.status=paid"
  - "Closed hours disables ordering and labels menu as Closed"

# -------------------------------
# 13) ESCAPE HATCHES (DESIGN‑TIME)
# -------------------------------
escape_hatches:
  db: "SQL portable to Neon"
  auth: "Adapter boundary to swap to Clerk"
  hosting: "App portable to Cloudflare Pages (ensure Edge compatibility)"
  ai: "aiClient wrapper to swap to Claude/Gemini"

# -------------------------------
# 14) DANGER ZONE (BLOCK)
# -------------------------------
block:
  - "Adding customer PII to chat/orders"
  - "Storing full Stripe payloads"
  - "Replacing RLS with Supabase anon keys on client"
  - "Moving data outside EU"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hudibaba0001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hudibaba0001)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
