## llm-speaker-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---
description: Use Bun instead of Node.js, npm, pnpm, or vite.
globs: "*.ts, *.tsx, *.html, *.css, *.js, *.jsx, package.json"
alwaysApply: false
---

## Project Overview

This is an LLM conversation system that enables multiple AI participants to have natural conversations with each other. The system manages speaking turns, conversation history, and participant personas.

### Architecture

- **index.ts**: Main entry point with a simple infinite conversation implementation using structured responses via Zod schemas
- **src/ConversationSystem.ts**: High-level conversation orchestrator that manages the full conversation flow, turn counting, and participant coordination
- **src/SpeakingRightManager.ts**: Core engine that handles LLM API calls, prompt building, turn management, and response parsing
- **src/types.ts**: Core type definitions for participants, messages, and conversation state
- **src/schemas.ts**: Zod schemas for structured LLM responses and validation, including SpeakerName enum

The system uses two architectural approaches:
1. **Simple approach (index.ts)**: Direct LLM calls with structured responses using `generateObject` from AI SDK
2. **Class-based approach (src/)**: More sophisticated turn management with `ConversationSystem` and `SpeakingRightManager` classes, better error handling, and conversation state management

## Development Commands

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Bun automatically loads .env, so don't use dotenv.

## APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`.
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`.
- `Bun.redis` for Redis. Don't use `ioredis`.
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`.
- `WebSocket` is built-in. Don't use `ws`.
- Prefer `Bun.file` over `node:fs`'s readFile/writeFile
- Bun.$`ls` instead of execa.

## Testing

Use `bun test` to run tests.

```ts#index.test.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

## Frontend

Use HTML imports with `Bun.serve()`. Don't use `vite`. HTML imports fully support React, CSS, Tailwind.

Server:

```ts#index.ts
import index from "./index.html"

Bun.serve({
  routes: {
    "/": index,
    "/api/users/:id": {
      GET: (req) => {
        return new Response(JSON.stringify({ id: req.params.id }));
      },
    },
  },
  // optional websocket support
  websocket: {
    open: (ws) => {
      ws.send("Hello, world!");
    },
    message: (ws, message) => {
      ws.send(message);
    },
    close: (ws) => {
      // handle close
    }
  },
  development: {
    hmr: true,
    console: true,
  }
})
```

HTML files can import .tsx, .jsx or .js files directly and Bun's bundler will transpile & bundle automatically. `<link>` tags can point to stylesheets and Bun's CSS bundler will bundle.

```html#index.html
<html>
  <body>
    <h1>Hello, world!</h1>
    <script type="module" src="./frontend.tsx"></script>
  </body>
</html>
```

With the following `frontend.tsx`:

```tsx#frontend.tsx
import React from "react";

// import .css files directly and it works
import './index.css';

import { createRoot } from "react-dom/client";

const root = createRoot(document.body);

export default function Frontend() {
  return <h1>Hello, world!</h1>;
}

root.render(<Frontend />);
```

Then, run index.ts

```sh
bun --hot ./index.ts
```

For more information, read the Bun API docs in `node_modules/bun-types/docs/**.md`.

## Project-Specific Commands

```bash
# Install dependencies
bun install

# Run the main conversation system
bun run index.ts

# Run with hot reload during development
bun --hot index.ts

# Run tests (when available)
bun test
```

## Key Concepts

### LLM Participants
Each participant has:
- `id`: Unique identifier  
- `name`: Display name (must match `SpeakerName` enum in schemas.ts)
- `persona`: Character description that influences responses
- `model`: OpenRouter model identifier (e.g., "google/gemini-2.5-pro")

### Conversation Flow
1. System sets initial topic and first speaker
2. Current speaker generates response using their persona and conversation history
3. Speaker nominates next participant to speak
4. Turn management transfers speaking rights
5. Process repeats until conversation ends

### Environment Setup
Requires `OPENROUTER_API_KEY` environment variable. Bun automatically loads .env files.

### Implementation Notes

#### Two Conversation Systems
- **index.ts system**: Uses `generateObject` with structured Zod schemas for consistent response parsing. Implements infinite conversation loop with structured turn management.
- **src/ system**: Uses `generateText` with manual response parsing. Provides more granular control over conversation state through class-based architecture.

#### Response Parsing Patterns
- **Structured approach (index.ts)**: Uses `ConversationResponseSchema` for type-safe responses
- **Manual parsing (SpeakingRightManager)**: Uses regex patterns to extract response, next speaker, and conversation end signals from text

#### Turn Management
- Current speaker selection via `setSpeaker()` method
- Automatic fallback to random speaker selection on invalid nominations
- History management with configurable limits (20 messages in index.ts, 10 in SpeakingRightManager)
- Conversation summary generation using GPT-4o-mini for cost efficiency

#### Adding New Participants
When adding participants, ensure:
1. `name` matches one of the values in `SpeakerName` enum in schemas.ts
2. `model` uses valid OpenRouter model identifiers
3. `persona` provides clear character instructions for consistent behavior

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akazdayo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akazdayo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
