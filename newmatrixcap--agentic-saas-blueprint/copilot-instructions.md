## agentic-saas-blueprint

> [One sentence: what this product is and who it's for]

# [PROJECT NAME]

[One sentence: what this product is and who it's for]

Production URL: [URL or "TBD"]
Branch in active development: main
Canonical contact email: [your contact email]

## Stack (locked)

Next.js 15 App Router · TypeScript strict · Tailwind (theme tokens, no literals) · Prisma + Supabase · NextAuth v5 · Stripe · Resend · Vitest · Vercel Cron

## Non-negotiables

1. No hardcoded color literals. Use `text-text-app`, `bg-bg-public`, `text-gold`, etc. Tokens flip on `[data-theme]` in `globals.css` automatically.
2. Light + dark mode work on every component, day one.
3. Mobile-first. Test 375 / 768 / 1280px.
4. Cards with a destination are clickable `<Link>` wrappers with hover lift + gold halo.
5. Admin views show real data with stat tiles, search, filters, pagination.
6. TDD. Failing vitest first, then impl.
7. No em-dashes in source files.
8. PII (SSN, full EIN, card numbers) scrubbed server-side before persistence.
9. Soft-fail on external services. Every Plaid/Stripe/Resend integration sits behind an env-gated wrapper. App builds without third-party keys.

## Workflow

1. Brainstorm → save spec to `docs/superpowers/specs/`
2. Plan → save plan to `docs/superpowers/plans/` (file paths + full code, no placeholders)
3. Subagent execute → task-by-task with TDD + spec review + code-quality review
4. `tsc --noEmit && vitest run && next build` → commit → push to main
5. **Verify deploy fired** via Vercel MCP. Webhook is unreliable; if commit SHA doesn't match latest READY deploy, run `vercel deploy --prod --yes`.
6. Curl `/api/health`. Confirm 200.

## Anti-patterns

- "Hotfix" comments that disable features instead of fixing root cause
- Building single-theme UI with plans to "add light later"
- Component-by-component sweep migrations (use a single CSS specificity layer scoped to `[data-theme='light']`)
- Skipping mobile review
- Bare HTML tables in admin pages
- Plans with placeholders ("TODO", "fill in details")
- Commits without typecheck + vitest first
- Trusting the Vercel auto-deploy webhook without verification

## Behavior

Default to action. Ask one clarifying question only when ambiguity blocks progress. When asked to plan, deliver complete file paths and code blocks. When asked to build, write the failing test, the code, the commit. When asked to ship, push AND verify the deploy AND curl `/api/health`.

Source: https://github.com/NewMatrixCap/agentic-saas-blueprint

---
> Source: [NewMatrixCap/agentic-saas-blueprint](https://github.com/NewMatrixCap/agentic-saas-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
