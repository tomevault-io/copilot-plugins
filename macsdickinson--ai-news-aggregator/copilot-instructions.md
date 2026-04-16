## ai-news-aggregator

> Project-wide rules for AI News Commentator MVP (architecture, commands, coding/git rules, acceptance criteria, guardrails)


- Project: AI News Commentator – AI-driven news curation with elaborations, Q&A, drafting, and publishing.
- MVP scope: feed, item actions, elaborate (150–250 words + 3–5 takeaways + citations + why-it-matters), Q&A, drafts (LinkedIn/Substack), editor + versioning, OAuth publish, roundups + simple scheduler, settings, analytics v0.
- Architecture principles: serverless-first, AWS-native, TypeScript end-to-end, strict attribution.
- Stack: Remix/React (frontend), AWS Lambda/API Gateway (backend; Express for local dev), DynamoDB + OpenSearch + S3, Cognito/Auth0, OpenAI primary + Anthropic fallback, SQS, EventBridge, Kinesis→S3(+QuickSight/PostHog), Secrets Manager + KMS.

Commands (run from repo root):

```bash
npm install
npm run dev            # backend:3001 + frontend:3000
npm run build
npm run test           # all workspaces
npm run test:watch     # backend
npm run lint
npm run type-check
npm run db:migrate     # backend
npm run db:seed        # backend
npm run clean
```

Coding & process:
- TypeScript strict; tests with Vitest for all features; JSDoc for public functions.
- Mandatory validation before commit: `npm run build`, `npm run lint`, `npm run type-check`, `npm run test`.
- Keep PRs focused; avoid unrelated changes.

Git rules:
- Never commit directly to `main`.
- Conventional commits: `feat`, `fix`, `test`, `chore`, `refactor`.
- Include issue key when applicable: `feat: implement onboarding modal (APP-123)`.
- Branch naming: `type/ISSUEKEY-kebab-summary` (e.g., `feat/APP-123-implement-onboarding-modal`).

Acceptance criteria (key):
- Time-to-feed ≤ 120s for new users.
- Elaborate latency: p50 ≤ 4s, p95 ≤ 8s.
- Citation integrity: every elaboration has ≥ 1 valid source link.
- Q&A persistence across refresh; appears in drafts.
- LinkedIn drafts ≤ 3,000 chars; prevent overflow.
- Publish returns URL/status with valid tokens.
- Roundup auto-drafts appear on schedule.

Guardrails:
- Do not bypass paywalls; store metadata + short excerpts only.
- Encrypt tokens; rotate regularly.
- Do not merge failing builds; validations must be green.

References:
@CLAUDE.md
@docs/milestone-1-implementation-plan.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MacsDickinson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
