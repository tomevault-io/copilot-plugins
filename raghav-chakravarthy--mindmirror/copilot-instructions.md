## mindmirror

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

MindMirror is a full-stack web application that analyzes a user's AI conversation history (Claude, ChatGPT, Gemini exports) and produces a brutally honest cognitive profile using the Claude API. A 3D brain activation sidebar powered by Meta's TRIBE v2 shows which brain regions activate when the user processes their most common topics.

## Running the App

```bash
# Install dependencies (first time)
npm install

# Option A — run everything at once
bash start.sh

# Option B — run services separately
npm run dev                      # Next.js on :3000
cd brain-service && python main.py  # TRIBE v2 sidecar on :8000
```

Add your Claude API key to `.env.local`:
```
ANTHROPIC_API_KEY=sk-ant-...
TRIBE_SERVICE_URL=http://localhost:8000
```

## Architecture

```
User uploads file(s)
  → DropZone detects platform
  → POST /api/parse → lib/parsers/{claude,chatgpt,gemini}.ts → Conversation[]
  → POST /api/analyze → Claude API (claude-sonnet-4-6) with system prompt → MindMirrorResult JSON
  → sessionStorage → /analysis page renders 8 sections
  → Topic hover → POST /api/brain → Python sidecar → TRIBE v2 → vertex activation → BrainSidebar recolors mesh
```

## Key Files

| File | Purpose |
|---|---|
| `lib/types.ts` | All shared TypeScript types (Conversation, MindMirrorResult, etc.) |
| `lib/system-prompt.ts` | MindMirror system prompt + `buildUserMessage()` |
| `lib/parsers/claude.ts` | Parses Claude JSONL export |
| `lib/parsers/chatgpt.ts` | Parses ChatGPT ZIP → conversations.json (tree traversal) |
| `lib/parsers/gemini.ts` | Parses Gemini Takeout ZIP JSON files |
| `app/api/analyze/route.ts` | Server-side Claude API call; reads `ANTHROPIC_API_KEY` |
| `app/api/brain/route.ts` | Proxies to TRIBE v2 sidecar; returns `{ activation: null }` if sidecar down |
| `components/analysis/BrainSidebar.tsx` | Three.js brain mesh, vertex-colors updated from TRIBE v2 output |
| `brain-service/main.py` | FastAPI sidecar for TRIBE v2 inference |
| `brain-service/model.py` | TRIBE v2 load + inference wrapper; gracefully handles missing package |

## MindMirrorResult Schema (8 fields)

`COGNITIVE_FINGERPRINT` · `TOPICS` · `DEPENDENCY_AUDIT` · `UNCOMFORTABLE_QUESTIONS` · `KNOWLEDGE_EDGE` · `ARCHETYPE` · `VERDICT` · `SHAREABLE_CARD`

Full type definitions in `lib/types.ts`.

## TRIBE v2 Sidecar Setup

TRIBE v2 requires the package from Meta's GitHub:
```bash
git clone https://github.com/facebookresearch/tribev2
pip install -e ./tribev2
cd brain-service && pip install -r requirements.txt
```

The sidecar returns `{ "activation": null }` gracefully if the model isn't installed — the brain viewer shows "Brain map unavailable" instead of crashing.

## Export Format Notes

- **Claude**: `conversations.jsonl` (JSONL) or `.json` — each line is `{ name, chat_messages: [{sender, text}], created_at }`
- **ChatGPT**: ZIP containing `conversations.json` — top-level array, each item has `{ title, create_time, mapping: { [id]: { message: { author, content } } } }`
- **Gemini**: Google Takeout ZIP — `Takeout/Gemini/` folder with JSON files `{ title, create_time, entries: [{role, text}] }`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Raghav-Chakravarthy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
