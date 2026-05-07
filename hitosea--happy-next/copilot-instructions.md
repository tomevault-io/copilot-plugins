## happy-next

> - **Always suggest options when asking questions** — when clarifying requirements, provide concrete recommendations or choices to help the user decide quickly, rather than open-ended questions

## Interaction Rules

- **Always suggest options when asking questions** — when clarifying requirements, provide concrete recommendations or choices to help the user decide quickly, rather than open-ended questions

## Language Preference

- Use Simplified Chinese for technical summaries and key conclusions, unless the user explicitly requests another language

## Repository Overview

Yarn 1.x monorepo (`yarn@1.22.22`).

| Package | Purpose |
|---------|---------|
| **happy-app** | Expo/React Native mobile + web client |
| **happy-cli** | CLI wrapper for Claude Code, Codex, Gemini |
| **happy-server** | Fastify backend (API + Socket.IO) |
| **happy-voice** | LiveKit-based voice gateway |
| **happy-wire** | Shared Zod schemas and wire types |

## Type Checking (run after all changes)

Run `yarn typecheck` in the package you modified. In happy-server the command is `yarn build` (which is `tsc --noEmit`, not a real build step).

## Running a Single Test (Vitest)

```bash
npx vitest run path/to/file.test.ts
```

---
> Source: [hitosea/happy-next](https://github.com/hitosea/happy-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
