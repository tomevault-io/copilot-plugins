## pre-payroll-approval-guard

> You're working inside the **WAT framework** (Workflows, Agents, Tools) to build and maintain the Pre-Payroll Approval Guard SaaS app. This architecture separates concerns so that probabilistic AI (like Claude) handles reasoning and orchestration, while deterministic code/tools handle execution. That separation ensures reliability for sensitive tasks like payroll change detection.

# Agent Instructions
You're working inside the **WAT framework** (Workflows, Agents, Tools) to build and maintain the Pre-Payroll Approval Guard SaaS app. This architecture separates concerns so that probabilistic AI (like Claude) handles reasoning and orchestration, while deterministic code/tools handle execution. That separation ensures reliability for sensitive tasks like payroll change detection.

## The WAT Architecture for SaaS Development

**Layer 1: Workflows (The Instructions)**
- Markdown SOPs stored in `workflows/`
- Each workflow defines the objective, required inputs, which tools to use, expected outputs, and how to handle edge cases (e.g., invalid CSV uploads, Supabase query failures).
- Tailored to the app: Focus on features like payroll snapshot comparisons, material change judgements, and UI rendering.
- Written in plain language, as if briefing a solo developer in Khammam, Telangana.

**Layer 2: Agents (The Decision-Maker)**
- This is your role (Claude via Claude Code in VSCode). You're responsible for intelligent coordination.
- Read the relevant workflow, run tools in the correct sequence, handle failures gracefully, and ask clarifying questions when needed (e.g., "Confirm CSV format?").
- You connect intent to execution without trying to do everything yourself.
- Example: To implement the "one-screen review" MVP, read `workflows/payroll_review_ui.md`, generate Next.js code via Claude Code, then deploy to Vercel via GitHub.

**Layer 3: Tools (The Execution)**
- TypeScript processing pipeline in `lib/payroll/` (diff calculation, rule engine, DB persistence).
- 40+ deterministic judgement rules in `lib/rules/` — one category per file, easy to read and modify.
- API integrations (Supabase, OpenAI, LemonSqueezy), CSV parsing, and deployment scripts.
- Credentials and API keys are stored in `.env` (e.g., SUPABASE_URL, OPENAI_API_KEY)—NEVER commit secrets to GitHub.

**Why this matters for our SaaS**: Building a payroll app involves accuracy (e.g., detecting blockers like negative net pay). If each AI step is 90% accurate, success drops fast. Keep rules deterministic (same input → same output always), and use AI only for generating human-readable explanations — never for deciding rule outcomes.

## How to Operate in This Project

**1. Look for existing tools first**
Before building anything new, check `lib/payroll/`, `lib/rules/`, and existing API routes. Only create new files when nothing exists for the task.

**2. Learn and adapt when things fail**
When you hit an error (e.g., Claude Code Git Bash issue):
- Read the full error message and trace.
- Fix the script/tool and retest (if it uses paid APIs like OpenAI, check costs first—aim for <$15/mo).
- Document what you learned in the workflow (e.g., rate limits on Supabase, Vercel timeout behavior).
- Example: Supabase query times out—refactor to use Edge Functions, verify, then update the workflow.

**3. Keep workflows current**
Workflows evolve as you learn (e.g., add Vercel deployment steps). Update them for better methods or constraints, but don't create/overwrite without confirmation unless explicitly told. Preserve them as core instructions.

**4. Integrate the Tech Stack**
- **VSCode + Claude Code**: Use for code generation and debugging (e.g., "Generate Next.js dashboard page").
- **Vercel**: Hosts the Next.js app and all API routes; auto-deploys on git push.
- **Supabase**: PostgreSQL DB (snapshots, diffs, judgements), Auth, RLS—enforce RLS for payroll data security.
- **GitHub**: Version control; triggers Vercel auto-deploy on push.
- **LemonSqueezy**: Billing — subscription management, webhooks, checkout.
- **OpenAI**: GPT-4o-mini for human-readable explanations on the review page. ~$0.01-0.03 per review. Rules are deterministic — AI never decides outcomes.
- **Other Potential Add-ons**: PostHog analytics.

**5. Payroll-Specific Guidelines**
- Prioritize MVP: One-screen UI with verdicts, blockers, and approvals (from original docs).
- Handle sensitive data: Anonymize in tools/workflows; comply with basics (e.g., no real PII in dev).
- Cost Optimization: Keep runtime <$60/mo; use free tiers where possible.

## Stack-Specific Guidance

### Supabase
- Use Postgres features properly (constraints, indexes, views)
- Prefer SQL logic over app-layer logic for data integrity
- Design schemas for auditability and diffs
- Assume multi-tenant B2B from day one

### Processing Pipeline (lib/payroll/)
- `processor.ts` is the single entry point — call this from API routes
- `diff.ts` is pure math — no DB calls, no side effects, easy to test
- `rulesEngine.ts` is deterministic — same input always produces same output
- `persistence.ts` owns all DB writes — never write to payroll tables elsewhere
- To add a rule: add it to the relevant file in `lib/rules/` — no other changes needed

### Frontend / Vercel
- Frontend is primarily:
  - Review
  - Explanation
  - Approval
- Avoid business logic in the UI
- Keep API routes thin

### GitHub
- Code should be commit-ready
- Avoid “example” or “toy” patterns
- Assume future contributors will read this

## The Self-Improvement Loop

Every failure strengthens the app:
1. Identify what broke (e.g., a rule producing false positives, a delta calculation edge case).
2. Fix it in the relevant TypeScript file — `lib/payroll/` or `lib/rules/`.
3. Verify the fix works (test locally with `npm run dev`).
4. Update the workflow SOP with the new approach.
5. Move on with a more robust SaaS.

This loop ensures iterative progress toward $500+/mo WTP.

## File Structure

**What goes where:**
- **Deliverables**: Final outputs to Supabase (data), Vercel (UI), or GitHub (code)—accessible via cloud.
- **Intermediates**: Temporary files that can be regenerated (e.g., test CSVs).

**Directory layout:**
- lib/payroll/     # Processing pipeline: processor, diff, rulesEngine, persistence
- lib/rules/       # Deterministic judgement rules (one file per category)
- pages/           # Next.js pages (e.g., dashboard, upload, review)
- pages/api/       # API routes (upload, approve, review, webhooks)
- components/      # React components
- supabase/        # DB schemas and migrations
- scripts/         # Operational scripts (LemonSqueezy setup, Vercel env, etc.)
- scripts/dev/     # Dev-only scripts (test-trial-expiration, etc.)
- workflows/       # Markdown SOPs
- docs/            # Project documentation
- __archive/       # Archived dev artifacts (not production code)
- .env             # API keys (gitignored)
- CLAUDE.md        # This file (agent instructions)
- README.md        # Project overview

## Mindset

You are building a **real product**, not a demo.
Assume:
- Paying customers
- Real payroll data
- Real consequences

If forced to choose between:
- Fast vs correct → choose correct
- Clever vs clear → choose clear
- Flexible vs safe → choose safe

**Core principle:** Local files are for dev/processing. App lives in cloud (Supabase DB, Vercel hosting). Everything in `.tmp/` is disposable.

## Bottom Line

You (Claude) orchestrate between app goals (workflows) and execution (tools/stack). Focus on building the Pre-Payroll Approval Guard: Read instructions, make smart decisions, call tools, recover from errors, and improve iteratively.Stay pragmatic, cost-conscious, and reliable. Keep learning to hit launch in <4 weeks.

---
> Source: [bharathreddybits/pre-payroll-approval-guard](https://github.com/bharathreddybits/pre-payroll-approval-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
