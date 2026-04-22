## jubilee-ai-final

> Recruiting intelligence platform for college football coaches. Built by Onarjae, CEO, affiliated with Assumption University CT. Beta product — 95%+ accuracy required before any coach demo.

# Jubilee.ai — CLAUDE.md

## What is Jubilee.ai?
Recruiting intelligence platform for college football coaches. Built by Onarjae, CEO, affiliated with Assumption University CT. Beta product — 95%+ accuracy required before any coach demo.

## Tech Stack
React + Vite, Tailwind, shadcn/ui, Wouter, TanStack Query v5, Express 5, Drizzle ORM, Neon PostgreSQL, OpenAI gpt-4.1-nano, Resend, Vapi + ElevenLabs + Deepgram + Twilio, Railway hosting.

## Active Features — DO NOT BREAK
- Travel Booking — COMPLETE
- Athlete Stats — COMPLETE
- Recruiting Packet PDFs — COMPLETE
- Territory Intelligence — IN PROGRESS
- AI Email Recruiting — IN PROGRESS
- AI Booking Agent — IN PROGRESS
- AI Film Analysis — Coming Soon page only
- HS Recruiting Studio — Coming Soon page only

## Anti-Hallucination Rules
- Never invent API keys, URLs, or env variables
- Never assume a library exists — check package.json first
- Never guess file paths — verify with ls or find before referencing
- Never modify files outside current task scope
- If unsure — stop and ask Onarjae before proceeding
- Never smash sections together or reformat unrelated code while fixing a bug

## Environment Variables — Never Hardcode
DATABASE_URL, SERPAPI_KEY, SESSION_SECRET, OPENAI_API_KEY, RESEND_API_KEY, DEMO_PACK_FALLBACK, VAPI credentials, Twilio credentials, REPLIT_AUTH_*

## Terminal Rules
- 1 terminal: routine fixes, small tasks
- 2 terminals: 2 independent bugs, different file boundaries
- 3 terminals: demo-critical only, verify no file overlap first
- 3 terminals = 3x token cost — use sparingly

## 95% Accuracy Standard
Every fix must have 4 sections:
1. CONTEXT — what is broken and where
2. ROOT CAUSE — what to investigate
3. FIX — what done looks like
4. VERIFY — exact steps to confirm fixed

Never start Phase 2 without reviewing Phase 1 findings first.
Always run a Review Agent pass after parallel terminal sessions.

## Git Rules
- Never commit to main without permission
- One fix per commit — no bundling unrelated changes
- Always describe what changed and why in commit messages

## Formatting Rules
- Always leave blank lines between sections
- Never smash two sections on the same line
- Keep this file clean — it is Claude Code's operating manual

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onarjae) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
