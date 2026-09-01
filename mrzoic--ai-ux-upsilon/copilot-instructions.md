## ai-ux-upsilon

> When working on this codebase, follow these guidelines:

# Agent Guidelines

When working on this codebase, follow these guidelines:

- Keep the existing architecture intact — do not redesign the UI or rewrite working systems
- Never hardcode API keys or secrets — always use environment variables
- The `.env` file must never be committed — use `.env.example` for templates
- Supabase integration is in `src/lib/supabase.ts` — all DB calls go through that file
- AI provider routing is in `src/server.ts` and `src/lib/gemini.ts`
- The main chat UI is in `src/routes/index.tsx`

---
> Source: [MrZoic/ai-ux-upsilon](https://github.com/MrZoic/ai-ux-upsilon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
