## agent-session-parser

> `agent-session-parser` is a standalone TypeScript library for parsing AI coding agent session transcripts. It supports Claude Code (JSONL) and Gemini CLI (JSON) formats. Ported from the session parsing internals of `github.com/entireio/cli`.

# CLAUDE.md

## Project overview

`agent-session-parser` is a standalone TypeScript library for parsing AI coding agent session transcripts. It supports Claude Code (JSONL) and Gemini CLI (JSON) formats. Ported from the session parsing internals of `github.com/entireio/cli`.

## Architecture

```
src/
  index.ts          # Public API — re-exports everything
  types.ts          # Shared agent-agnostic types (TokenUsage, Event, AgentSession, etc.)
  utils.ts          # Shared utilities (stripIDEContextTags, deduplicateStrings)
  chunking.ts       # Format-aware chunking for large transcripts (>50MB)
  parsers/
    claude/
      types.ts      # Claude Code JSONL types (TranscriptLine, MessageType, ContentType, etc.)
      parse.ts      # JSONL parser (parseFromString, parseFromBytes, etc.)
      extract.ts    # Extraction (files, prompts, tokens, subagents)
      hooks.ts      # Hook input types for Claude Code lifecycle events
      index.ts      # Re-exports
    gemini/
      types.ts      # Gemini CLI JSON types (GeminiTranscript, GeminiMessage, etc.)
      parse.ts      # JSON parser (parseTranscript, parseTranscriptFromBytes, etc.)
      extract.ts    # Extraction (files, prompts, tokens)
      hooks.ts      # Hook input types for Gemini CLI lifecycle events
      index.ts      # Re-exports
```

The two parsers are namespaced: `import { claude, gemini } from "agent-session-parser"`.

Each parser follows the same pattern:
- **types.ts** — agent-specific message/tool/hook types
- **parse.ts** — parse raw transcript content into typed structures
- **extract.ts** — extract files, prompts, tokens, and other data from parsed transcripts

## Common commands

```bash
npm test             # Run tests (vitest)
npm run test:watch   # Run tests in watch mode
npm run build        # Build with tsup (ESM + CJS + DTS)
npm run typecheck    # Type-check without emitting (also aliased as `npm run lint`)
```

## Key conventions

- **Pure library, no side effects.** No file I/O, no network calls. Callers provide raw content strings or Uint8Arrays.
- **Subagent transcript loading is callback-based.** Functions like `calculateTotalTokenUsage` and `extractAllModifiedFiles` take a `(agentId: string) => string | null` callback so the caller controls I/O.
- **Malformed input is skipped, not thrown.** JSONL lines that fail `JSON.parse` are silently dropped. This is intentional — transcripts may be incomplete or corrupted.
- **Deduplication preserves order.** `extractModifiedFiles` and `deduplicateStrings` return items in first-seen order.
- **Streaming dedup for Claude.** `calculateTokenUsage` deduplicates by `message.id`, keeping the row with the highest `output_tokens` (streaming produces duplicate rows per API call).

## Adding a new agent parser

1. Create `src/parsers/<agent>/` with `types.ts`, `parse.ts`, `extract.ts`, `hooks.ts`, `index.ts`
2. Add `export * as <agent> from "./parsers/<agent>/index.js"` to `src/index.ts`
3. Add tests in `src/parsers/<agent>/__tests__/`
4. If the format needs chunking support, add cases to `src/chunking.ts`

## Testing

Tests live alongside source as `__tests__/*.test.ts` files (e.g. `src/parsers/claude/__tests__/extract.test.ts`). The project uses vitest. Run `npm test` before committing.

---
> Source: [alexngai/agent-session-parser](https://github.com/alexngai/agent-session-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
