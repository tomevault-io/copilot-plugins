## opsamplehtml

> - **What it is:** AI-powered clinical documentation demo platform for neurology outpatient practices ("Sevaro Ambulatory" demo). Live at app.neuroplans.app.

# Sevaro Clinical (OPSAmplehtml / "OpieSample")

- **What it is:** AI-powered clinical documentation demo platform for neurology outpatient practices ("Sevaro Ambulatory" demo). Live at app.neuroplans.app.
- **Type:** code repo
- **Stack / tools:** Next.js 15 (App Router, TypeScript), AWS RDS (PostgreSQL), AWS Cognito (OAuth+PKCE), AWS Bedrock (Claude Sonnet 4.6), OpenAI Realtime API, Deepgram Nova-3, Twilio, Tailwind CSS.
- **How to run / test:** `npm run dev` (local dev server, port 3000); `npm run build`; `npm run lint`; `npm run test` (vitest); `npx tsc --noEmit` for type check.
- **Key files / structure:**
  - `src/app/` — Next.js App Router routes (`/physician`, `/dashboard`, `/ehr`, `/wearable`, `/triage`, `/consult`, `/follow-up`, `/sdne`)
  - `src/lib/` — shared logic (db-query, cognito, ASR lexicon, RDS CA bundle)
  - `docs/` — plans, handoffs, changelog
  - `qa/` — test runbook, structured test cases, release checklist
  - `packages/feedback` — local `@sevaro/feedback` package
- **Conventions:** See CLAUDE.md — push-to-deploy on `main` (AWS Amplify, no Vercel), documentation ships with every code commit, patient-data flows must be described generically per clinical-safety rules.
- **Current focus / handoff notes:** as of 2026-07-05, most recent commits are on `feat/neuro-faq-voice-poc` — a patient/caregiver FAQ voice
  feature (Polly generative voice engine, FAQ-only scope with red-flag 911 intercept). Prior main-line work shipped ASR vocabulary biasing
  across 5 voice surfaces, RDS TLS validation, and an AI Historian Realtime API upgrade. See `docs/CHANGELOG.md` for full history.

<!-- Read by Claude Code, Claude Cowork, and OpenAI Codex. Auto-generated 2026-07-05 (Fable run); edit freely. -->

---
> Source: [blondarb/OPSAmplehtml](https://github.com/blondarb/OPSAmplehtml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
