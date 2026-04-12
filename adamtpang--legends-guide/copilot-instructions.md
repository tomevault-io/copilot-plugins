## legends-guide

> legends.guide v2 — streaming chat with 10 founders, compare feature, source citations. Builds cleanly.

@AGENTS.md

# Project Status

## What's Built
legends.guide v2 — streaming chat with 10 founders, compare feature, source citations. Builds cleanly.

### Architecture
- **Next.js 16 + Tailwind CSS v4 + TypeScript** (App Router)
- **Claude API** (claude-sonnet-4-5-20250514) for chat via `@anthropic-ai/sdk` with streaming
- **No database** — knowledge chunks embedded directly in system prompts (sufficient for 10 founders)
- **Modular AI config** — model/provider swappable via `AI_CONFIG` in figures.ts

### File Map
- `src/lib/figures.ts` — 10 founder definitions with deeply researched system prompts, knowledge chunks with source citations, and `AI_CONFIG` for model swapping
- `src/app/page.tsx` — Landing page: dark grid of 10 FigureCards + Compare CTA
- `src/app/chat/[figure]/page.tsx` — Streaming chat UI: header, message thread, real-time token streaming, typing indicator
- `src/app/compare/page.tsx` — Compare page: pick 2 founders, ask same question, parallel streaming responses side-by-side
- `src/app/api/chat/route.ts` — POST route: SSE streaming via Anthropic SDK stream(), returns text/event-stream
- `src/components/FigureCard.tsx` — Card with gradient avatar, name, era, hook
- `src/components/ChatMessage.tsx` — Message rendering with citation parsing and display

### The Founding 10
1. John D. Rockefeller — wealth, discipline, monopoly (Titan by Chernow)
2. Steve Jobs — product, taste, focus (Isaacson biography)
3. Jeff Bezos — Day 1, customer obsession (Everything Store, Invent and Wander)
4. Elon Musk — first principles, speed (Isaacson, Vance biographies)
5. Jensen Huang — NVIDIA, suffering, conviction (The Nvidia Way)
6. Peter Thiel — monopoly, contrarian thinking (Zero to One)
7. Charlie Munger — mental models, inversion (Poor Charlie's Almanack)
8. Benjamin Franklin — self-improvement, reinvention (Autobiography, Isaacson)
9. Sam Walton — retail, hustle, culture (Made in America)
10. Naval Ravikant — leverage, specific knowledge, happiness (Almanack)

## Deploy Checklist
- [ ] Set ANTHROPIC_API_KEY in Vercel env vars
- [ ] `npx vercel --prod`
- [ ] Point legends.guide domain to Vercel

## Future (v2+)
- [ ] Add real portrait images (update `portrait` field in figures.ts)
- [ ] ElevenLabs TTS voice clones per founder
- [ ] Supabase + pgvector RAG when scaling to 50+ founders
- [ ] Web search integration for real-time knowledge
- [ ] Conversation persistence (database)
- [ ] Auth for premium features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamtpang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adamtpang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
