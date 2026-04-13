## lovie-p2p-payment

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Agent Instructions

## Role
You are building a P2P payment request feature for a fintech app. Follow the spec at `specs/001-p2p-payment-request/spec.md` as the source of truth.

## Rules

### Must Follow
- Read the spec before making any changes
- Amounts are INTEGER cents ($50 = 5000). Never float.
- Row Level Security is on. Don't bypass it.
- Validate on both client AND server
- Use `WHERE status = 'pending'` for all status changes (race condition protection)
- Run tests after changes: `npx playwright test --project=chromium`

### MCP Usage
- Use Supabase MCP (`execute_sql`, `list_tables`) to check database state
- Use Playwright MCP to test features visually in a real browser
- Use Context7 to pull latest docs when unsure about API changes

### Code Patterns
- TypeScript strict — no `any` types
- shadcn/ui for all UI components
- Tailwind for styling
- Supabase server client for API routes (`lib/supabase/server.ts`)
- Supabase browser client for client components (`lib/supabase/client.ts`)
- Toast notifications via Sonner for user feedback

### Testing
- Playwright config: `video: 'on'`, `screenshot: 'on'`, `trace: 'on'`
- Test both accounts: samsungsarz@outlook.com and emreozyorukdev@gmail.com
- All tests in `tests/` directory

### Don't
- Don't use floats for money
- Don't skip server-side validation
- Don't bypass Row Level Security
- Don't hardcode Supabase keys (use env vars)
- Don't commit .env.local (it's in .gitignore)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emreozyoruk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
