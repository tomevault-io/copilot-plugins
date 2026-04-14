## quranpulse-v6-0

> You are an expert React TypeScript developer working on QuranPulse v6.0.

# Cursor Rules - QuranPulse v6.0

You are an expert React TypeScript developer working on QuranPulse v6.0.

## Project Context

QuranPulse v6.0 ("Noor-e-Cyber") - Futuristic Islamic PWA
Phase: PRODUCTION | URL: https://quranpulse.my

## Tech Stack (DO NOT CHANGE)

- React 18 + Vite + TypeScript
- Tailwind CSS v4 + Framer Motion
- Supabase (PostgreSQL + Auth + Edge Functions)
- Zustand (global state) + React Query (server state)
- Jest + React Testing Library

## Code Style

- ALWAYS use functional components with hooks
- ALWAYS use TypeScript strict mode
- ALWAYS use Tailwind CSS for styling
- PREFER Framer Motion for animations
- USE Zustand for global state
- USE React Query for server state

## Verification (MANDATORY!)

Before EVER saying "done", you MUST:

1. Run `npx tsc --noEmit` → ZERO TypeScript errors
2. Run `npm run build` → Build successful
3. Include verification results in response

## NEVER

- Say "done" without running verification commands
- Ask user to "check if it works"
- Delete files without checking .agent/context/PROTECTED_FILES.md
- Leave TypeScript errors unfixed
- Skip testing
- Change locked tech stack without approval

## ALWAYS

- Read AGENTS.md first for full context
- Check .agent/PROJECT_STATUS.md for current phase
- Run build/test checks before completion
- Commit with format: [AGENT:Cursor] type: message

## Commands

```bash
npm run dev          # Dev server (port 5173)
npm run build        # Production build
npm test             # Jest tests
npx tsc --noEmit     # TypeScript check
```

## Project Structure

```text
src/modules/         # Feature modules (quran, iqra, smart-deen)
src/services/        # API services
src/components/      # Shared UI
supabase/functions/  # Edge Functions
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thisisniagahub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
