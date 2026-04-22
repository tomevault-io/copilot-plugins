## express

> > **Token-aware**: This file is loaded every session. Keep it concise.

# New-U Peptides — Claude Development Guide

> **Token-aware**: This file is loaded every session. Keep it concise.
> Detailed context is in `.claude/memory/` and injected via hooks.

## Quick Reference
- **Stack:** Express.js + TypeScript, PostgreSQL (AWS Aurora), Stripe, MailerSend, Vercel
- **Module:** ES modules — `.js` extensions required in all imports
- **Build:** `pnpm run build` (TypeScript → `dist/`)
- **Deploy:** Auto on push to `main`, `master`, or `claude/*` (Vercel deploy hook triggers via PostToolUse)

## Code Style
- TypeScript ES2022, Node 20, async/await + try-catch
- camelCase vars/functions, PascalCase types
- `console.error()` for errors, `// ──` section headers
- Rate limit: `rateLimit({ windowMs: 60000, max: 60 })`

## File Map (DO NOT read full files — grep first)
```
src/index.ts          – Routes & middleware (~4900 lines, grep don't read)
src/products.ts       – Product catalog data
src/db.ts             – PostgreSQL (Aurora) pool + schema (initializeDatabase)
src/payment.ts        – Stripe payment intents & webhooks
src/notifications.ts  – Email via MailerSend
src/mailer.ts         – MailerSend email client wrapper
src/saleor.ts         – Saleor GraphQL integration
src/deployment-center.ts – Fulfillment & operator management
src/affiliate.ts      – Affiliate tracking & commissions
src/encryption.ts     – KMS envelope encryption for PII fields
src/coinspaid.ts      – CoinsPaid crypto payments
src/passimpay.ts      – Passimpay crypto payments
src/nowpayments.ts    – NOWPayments crypto payments (invoice, webhook, signature verification)
src/conduit-ucpi.ts   – Conduit UCPI payments
```

## Database (PostgreSQL — AWS Aurora)
- **Engine:** Aurora PostgreSQL Serverless v2 via `pg` driver
- **Auth:** IAM auth (`@aws-sdk/rds-signer` + Vercel OIDC) or standard password
- **Compat layer:** `src/db.ts` wraps `pg` with a mysql2-style API (`pool.execute(sql, [?params])`) — `?` placeholders auto-convert to `$1, $2, ...`
- **SQL dialect:** Write standard PostgreSQL — use `TIMESTAMPTZ`, `TEXT`, `ON CONFLICT`, `NOW()`, `$1` params (or `?` which auto-converts)
- Core tables: `orders`, `checkout_orders`, `order_items`, `customers`, `professor_accounts`, `operators`, `fulfillment_queue`, `sales_ledger`, `affiliate_orders`, `crypto_payments`, `saleor_sync`, `email_leads`
- UUID PKs, connection pooling, always release in `finally`
- Schema defined in `src/db.ts` → `initializeDatabase()`

## Key Routes (in src/index.ts)
- `GET /`, `/shop`, `/checkout`, `/cart` — pages
- `POST /api/create-payment-intent` — Stripe
- `POST /webhooks/stripe`, `/webhooks/coinspaid`, `/webhooks/passimpay`, `/webhooks/nowpayments` — payment callbacks
- `POST /api/nowpayments/create-invoice` — NOWPayments crypto invoice
- `GET /api/payment-check/:orderRef` — lightweight payment status poll (for success page)
- `POST /api/checkout-order` — order submission
- `GET /admin/dashboard` — admin (key-protected)
- `GET /healthz` — health check

## Env Vars
DB (PostgreSQL): `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`, `PGSSLMODE`
DB (IAM auth): `AWS_REGION`, `AWS_ROLE_ARN` (enables RDS IAM + Vercel OIDC, no password needed)
Stripe: `STRIPE_PUBLIC_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
Email: `MAILERSEND_API_KEY`, `FROM_EMAIL`, `NOTIFY_EMAIL`
Encryption: `KMS_KEY_ID` (AWS KMS key ARN — enables PII field encryption)
NOWPayments: `NOWPAYMENTS_API_KEY`, `NOWPAYMENTS_IPN_SECRET`
Admin: `ADMIN_KEY`, `SESSION_SECRET`
Google OAuth: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`

## Patterns
```typescript
// Route
app.post('/path', apiLimiter, async (req, res) => {
  try { /* logic */ res.json({ success: true }) }
  catch (err) { console.error(err); res.status(500).json({ error: 'Internal server error' }) }
})
// DB (compat layer auto-converts ? → $1,$2,... for PostgreSQL)
const conn = await pool.getConnection()
try { await conn.execute('SELECT ... WHERE id = ?', [param]) } finally { conn.release() }
// Or without connection: await pool.execute('INSERT INTO t (id) VALUES (?)', [id])
// Page
app.get('/page', (req, res) => res.sendFile(path.join(__dirname, '..', 'pages', 'page.htm')))
```

## Before Committing
1. `pnpm run build` — verify TypeScript compiles
2. `git status` — confirm expected files
3. No `.env` or secrets in commits

## Anti-Patterns
- Read `src/index.ts` in full (4900+ lines, ~50k tokens) — always grep first
- Read `src/products.ts` in full or anything in `dist/`
- Use CommonJS `require()` — this project uses ES modules
- Forget `.js` extensions in TypeScript imports
- Make temporary fixes — find root causes, senior developer standards
- Over-engineer simple fixes — balance elegance with pragmatism
- Add features, refactor code, or "improve" beyond what was asked
- Commit `.env`, credentials, or secrets

## Working Practices

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction: update `.claude/memory/decisions.md` with the lesson
- Write rules that prevent the same mistake repeating
- decisions.md is auto-injected every session — lessons persist automatically

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

<!-- CLAUDE.md v2 | 2026-03-28 | optimized per alirezarezvani + jezweb patterns -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/new-u-peptides) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
