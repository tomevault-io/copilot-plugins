## claude-sonar

> This file tells Claude Code (and any AI coding agent) how to work on this project effectively. If you are a blind or low-vision developer using Claude Code to contribute to claude-sonar, this file is written with you in mind.

# CLAUDE.md — Instructions for AI agents working on claude-sonar

This file tells Claude Code (and any AI coding agent) how to work on this project effectively. If you are a blind or low-vision developer using Claude Code to contribute to claude-sonar, this file is written with you in mind.

## What This Project Is

claude-sonar is a screen reader accessibility plugin for Claude Code. It intercepts tool output via hooks and reformats it into concise, spoken-friendly summaries. The people who use this project may not be able to see your code changes — they will hear them through a screen reader or TTS engine. Keep that in mind with every line you write.

## Project Status

This project was released as a seed for the community. The scaffolding, architecture, and test suite are solid (545 tests, 42 test files), but there may be bugs in edge cases, especially around screen reader output quality. If you are fixing a bug, add a test for it. If you are improving spoken output, test it with a screen reader if you can.

## Build and Test

```bash
npm run build        # Build with tsup (dual ESM/CJS)
npm test             # Run all tests (vitest)
npm run test:coverage # Tests with V8 coverage
npm run lint         # ESLint
npm run typecheck    # tsc --noEmit
```

Always run `npm test` after changes. All 545+ tests should pass. Do not skip this.

## Architecture — The Short Version

The core is a pure pipeline with no I/O:

```
stdin JSON -> parseHookEvent() -> formatToolUse() -> classifySignificance() -> applySignificance() -> buildHookOutput() -> stdout JSON
```

The CLI layer (`src/cli/`) is a thin wrapper that handles stdin, stdout, TTS, and earcons. The pipeline (`src/core/pipeline.ts`) is a pure function: string in, structured result out. This separation is what makes the project testable.

## Key Source Locations

| Directory | What It Contains |
|-----------|-----------------|
| `src/core/pipeline.ts` | Main entry point. Routes hook events to handlers. Start here. |
| `src/core/types.ts` | All hook event type definitions |
| `src/core/significance.ts` | Classifies events as noise, routine, notable, or important |
| `src/core/apply-significance.ts` | Adjusts output based on significance level |
| `src/core/digest.ts` | Accumulates events for end-of-turn summary |
| `src/core/progress.ts` | Tracks tool elapsed time |
| `src/core/history.ts` | JSONL event log per session |
| `src/core/code-summarizer.ts` | Extracts function, class, and interface declarations from code |
| `src/formatters/` | One file per tool (bash.ts, edit.ts, read.ts, etc.) |
| `src/formatters/index.ts` | Formatter registry with conflict detection |
| `src/formatters/types.ts` | FormattedOutput (contextText + ttsText), Formatter interface |
| `src/config/types.ts` | All config interfaces (SonarConfig and sub-interfaces) |
| `src/config/defaults.ts` | Default config values |
| `src/earcon/sounds.ts` | Earcon IDs and OS sound mappings |
| `src/settings/index.ts` | Claude Code settings.json hook management |
| `src/cli/index.ts` | CLI entry point (all commands registered here) |
| `src/output/index.ts` | Builds the final HookJsonOutput JSON for stdout |

## The Two Audiences Rule

Every formatter produces two separate strings:

- **contextText** — read by Claude (the AI). Include full file paths, line numbers, structural details, anything that helps Claude make better decisions on the next step.
- **ttsText** — heard by the human through a screen reader or TTS engine. Must be short, speakable, and useful. No full file paths (just filenames). No special characters. No technical noise.

These serve different audiences. Never conflate them.

## Accessibility Rules for All Output

These are not suggestions. They are requirements.

1. **No emoji.** Screen readers spell out emoji names ("smiling face with open mouth"), which wastes time and interrupts flow.
2. **No ANSI color codes.** Screen readers do not convey color. Escape sequences produce garbled speech.
3. **No Unicode box-drawing characters** (lines, corners, borders). Screen readers announce each one.
4. **TTS text must be speakable.** Use "TypeScript" not "TS". Use periods for pauses. Keep it to 1-2 sentences.
5. **No spinners, progress bars, or animations.** These are meaningless to someone who cannot see them and actively harmful to screen reader output.
6. **Test the actual strings.** Do not just test that a formatter returns something. Test that `contextText` contains the right information and `ttsText` says the right words.

## How to Add a Formatter

1. Create `src/formatters/your-tool.ts` implementing the `Formatter` interface (see `src/formatters/types.ts`)
2. Give it a unique `id` and list the `toolNames` it handles
3. Register it in `src/formatters/index.ts` inside `registerBuiltinFormatters()`
4. Add tests in `tests/formatters/your-tool.test.ts`
5. Test both `contextText` and `ttsText` output

## How to Add a Hook Event

1. Add the event interface to `src/core/types.ts`
2. Add a parser case in `parseHookEvent()` in `src/core/pipeline.ts`
3. Add a handler function and wire it into the switch in `processHookEvent()`
4. Add the event name to `HOOK_EVENT_TYPES` in `src/settings/index.ts`
5. Add tests

## Safety Rules

- The `format` command must ALWAYS output valid JSON to stdout, even on errors. Claude Code expects this.
- Stdin reader has a 5-second timeout and 5MB size limit.
- TTS text is sanitized: `--` flag terminator before text, truncation, control character stripping.
- Config merge uses `structuredClone` (not spread) to avoid prototype mutation.
- Hook detection is structural (checks command prefix), not substring matching on JSON.
- All formatters are wrapped in try/catch at the registry level — a buggy formatter falls back gracefully.

## Test Patterns

- Formatters: test `contextText` and `ttsText` separately with exact string assertions
- Pipeline: use `DEFAULT_CONFIG` as base, override specific fields
- Config: test merge behavior and prototype pollution guards
- Earcons: test selection logic (pure functions), not playback (I/O)

Tests live in `tests/` mirroring the `src/` structure. Run a single file with:

```bash
npx vitest run tests/formatters/bash.test.ts
```

## Changesets

For user-facing changes, create a changeset before submitting a PR:

```bash
npx changeset
```

Internal refactors and test-only changes do not need changesets.

---
> Source: [vylasaven/claude-sonar](https://github.com/vylasaven/claude-sonar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
