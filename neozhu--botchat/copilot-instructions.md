## botchat

> This repository is a chat-first dashboard application built around expert personas, multi-session chat, streaming AI responses, file attachments, and readable markdown/code output.

# AGENTS.md

## Purpose

This repository is a chat-first dashboard application built around expert personas, multi-session chat, streaming AI responses, file attachments, and readable markdown/code output.

When working in this repo, optimize for:

- preserving the existing two-panel chat UX
- reusing current `botchat`, `ai-elements`, and `ui` components before adding new abstractions
- keeping server/data behavior aligned with the current Supabase-backed flow

## Tech Stack

- `Next.js 16` with App Router
- `React 19`
- `AI SDK 6` with `@ai-sdk/react` and `@ai-sdk/openai`
- `Supabase` for data storage, expert/session/message persistence, and attachment storage
- `Tailwind CSS v4`
- `Radix UI`
- `TypeScript`

## Architecture Overview

- [`app/page.tsx`](D:\github\botchat\app\page.tsx) is the main server entry. It loads initial bootstrap data and renders the dashboard.
- [`lib/botchat/bootstrap.ts`](D:\github\botchat\lib\botchat\bootstrap.ts) prepares initial sessions, experts, active state, and initial messages for first paint.
- [`components/botchat/dashboard.tsx`](D:\github\botchat\components\botchat\dashboard.tsx) is the main client orchestrator. It owns session switching, `useChat`, optimistic UI updates, attachment upload flow, and message persistence.
- [`app/api/chat/route.ts`](D:\github\botchat\app\api\chat\route.ts) streams model output through AI SDK and pulls the active expert system prompt from Supabase.
- `app/api/sessions/*`, `app/api/messages/sync/route.ts`, `app/api/experts/*`, and `app/api/attachments/upload/route.ts` form the main application API surface.

## Main UI Areas

### 1. Sessions Sidebar

- Primary file: [`components/botchat/sessions-panel.tsx`](D:\github\botchat\components\botchat\sessions-panel.tsx)
- Purpose: show chat sessions, active selection, relative timestamps, deletion, and access to settings.
- Important behavior: this is not just navigation; it also exposes the expert system dialog from the footer area.

### 2. Chat Panel

- Primary file: [`components/botchat/chat-panel.tsx`](D:\github\botchat\components\botchat\chat-panel.tsx)
- Purpose: render the conversation timeline, empty state, composer, attachment previews, expert shortcuts, and assistant message display.
- Important behavior: keep the current chat-first interaction model intact. The app is designed around one focused conversation surface, not a dashboard grid.

### 3. Expert Settings Dialog

- Primary file: [`components/botchat/expert-settings-dialog.tsx`](D:\github\botchat\components\botchat\expert-settings-dialog.tsx)
- Purpose: create, edit, duplicate, delete, search, reorder, and AI-generate expert prompts.
- Important behavior: this is the control center for expert definitions. Changes here affect session behavior and chat system prompts.

### 4. AI Rendering Primitives

- Primary area: [`components/ai-elements`](D:\github\botchat\components\ai-elements)
- Purpose: reusable AI-centric presentation pieces such as conversation wrappers, message rendering, code blocks, reasoning/tool UI, prompts, and related display elements.
- Guidance: if a new chat rendering pattern fits this shared layer, extend `ai-elements`; if it is product-specific, keep it under `components/botchat`.

### 5. Shared UI Primitives

- Primary area: [`components/ui`](D:\github\botchat\components\ui)
- Purpose: Radix-based building blocks and local design system primitives.
- Guidance: prefer composing these primitives instead of introducing parallel component patterns.

## Data Model and Flow

- Experts, sessions, and messages are stored in Supabase.
- Attachments are uploaded through [`app/api/attachments/upload/route.ts`](D:\github\botchat\app\api\attachments\upload\route.ts) and stored in a public Supabase Storage bucket named `chat-attachments`.
- The dashboard fetches bootstrap data on the server, then manages most interaction state client-side.
- New or updated messages are persisted through [`app/api/messages/sync/route.ts`](D:\github\botchat\app\api\messages\sync\route.ts).
- Chat generation is streamed from [`app/api/chat/route.ts`](D:\github\botchat\app\api\chat\route.ts), with the system prompt selected from the active expert or session.

## Key Directories

- [`app`](D:\github\botchat\app): App Router pages, layout, globals, API routes
- [`components/botchat`](D:\github\botchat\components\botchat): product-specific chat UI
- [`components/ai-elements`](D:\github\botchat\components\ai-elements): reusable AI presentation layer
- [`components/ui`](D:\github\botchat\components\ui): low-level UI primitives
- [`lib/botchat`](D:\github\botchat\lib\botchat): domain types, bootstrap, server data helpers, expert seeds
- [`lib/ai`](D:\github\botchat\lib\ai): AI provider/model helpers
- [`lib/supabase`](D:\github\botchat\lib\supabase): Supabase client setup
- [`supabase/schema.sql`](D:\github\botchat\supabase\schema.sql): schema and RLS baseline

## Working Rules for AI Agents

- Prefer small, targeted changes that preserve the current server-first bootstrap plus client-orchestrated chat model.
- Once the user has clearly specified the implementation goal, proceed directly with the code changes instead of asking for repeated confirmation before each edit.
- Only stop to ask follow-up questions when the requirement is genuinely ambiguous, the change would materially expand scope, or a destructive action is required.
- Do not replace `useChat` flow in [`components/botchat/dashboard.tsx`](D:\github\botchat\components\botchat\dashboard.tsx) unless the task specifically requires it.
- Do not hardcode model names in feature code. Follow the existing model helper in [`lib/ai/openai.ts`](D:\github\botchat\lib\ai\openai.ts).
- When changing expert behavior, inspect both the expert dialog and the related expert API routes before editing.
- When changing message rendering, verify whether the change belongs in `components/botchat/chat-panel.tsx` or in a reusable `components/ai-elements/*` component.
- Keep the current sidebar plus main chat panel layout unless the task explicitly asks for a layout redesign.
- Preserve attachment support and message persistence behavior when editing composer or send flows.
- Follow the existing TypeScript and Tailwind patterns already in the file you are touching.

## Environment Notes

Expected local environment variables include:

- `OPENAI_API_KEY`
- `OPENAI_MODEL`
- `PUBLIC_SUPABASE_URL`
- `PUBLIC_SUPABASE_PUBLISHABLE_DEFAULT_KEY`

Supabase assumptions in this repo:

- database schema comes from [`supabase/schema.sql`](D:\github\botchat\supabase\schema.sql)
- storage bucket name is `chat-attachments`
- current setup is intentionally simple and README describes anon-friendly local setup

## Common Commands

- `bun dev`
- `bun run build`
- `bun start`
- `bun lint`

## Verification Expectations

Before claiming completion, run the narrowest relevant verification possible:

- UI or TypeScript changes: `bun run build` if feasible
- lint-sensitive changes: `bun lint`
- focused prompt logic changes: run or update the existing test at [`app/api/experts/generate/prompt.test.mts`](D:\github\botchat\app\api\experts\generate\prompt.test.mts)

If you could not run verification, say so explicitly.

---
> Source: [neozhu/botchat](https://github.com/neozhu/botchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
