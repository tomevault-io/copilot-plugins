## pi-tool-repair-layer

> <!-- Last validated: 2026-06-04 by cali-agents-md-validator -->

# Agent Guidelines

<!-- Last validated: 2026-06-04 by cali-agents-md-validator -->
<!-- Do not restructure or delete sections. Update individual values in-place when they change. -->

## Core Principles

- **Cache preservation is non-negotiable.** The extension follows the 4-rule cache-safety pattern: (1) static cutoff, (2) one-shot per (kind, key), (3) byte-deterministic, (4) stable position. Pre-execution repairs have zero cache impact. Post-execution modifications to `tool_result.content` are allowed IF they follow all 4 rules — the write-directory-fallback (Phase 6) does. Guidance is queued and injected via the `context` event side channel — shallow-copied messages array, push only. Full contract: `docs/cache-safety.md`.
- **Keep this file under 70-100 lines (sweet spot), 150 hard limit.** Every line competes for the agent's context budget. Move deep details to `docs/<topic>.md`.
- **Only repair primary/builtin tools.** External extension tools (`agent_browser`, `web_search`, `fetch_content`) get generic guidance via `getToolHelp` — no field-level or arg-level fixes.
- **Validate-then-repair.** Pure repair functions only fix structural arg issues (types, nulls, arrays), never content fields (`command`, `code`, `oldText`, `newText`).
- **No external runtime dependencies.** TypeScript + Vitest only.

---

## Project Overview

**Project type:** pi coding-agent extension
**Primary language:** TypeScript 5.7 (strict)
**Key dependencies:** Vitest 3.1.1 (testing only)
**Architecture deep-dive:** See `docs/architecture.md`

---

## Commands

| Command | Description |
|---------|-------------|
| `npm test` | Run all tests (Vitest) |
| `npx vitest run` | Run tests once |
| `npx vitest --watch` | Watch mode |
| `npx vitest run repairs.test.ts -t "structural integrity"` | Verify extracted architecture survived |
| `npx vitest run repairs/test-coverage.test.ts` | Verify every repair export is tested |
| `npx vitest run repairs/catalog-drift.test.ts` | Verify catalog matches source |
| `npx vitest run extension-integration.test.ts` | Verify handler wiring + lifecycle |
| `npm run setup:hooks` | One-time install of pre-commit hook (core.hooksPath) |

---

## Releases

- **Stay at 0.x.x until user explicitly says stable.** No 1.x.x tag or release without user confirmation.
- First 1.0.0 only when user says "stable" or "release 1.0".
- Once at 1.x.x, semver rules apply normally (fix → patch, feat → minor, breaking → major).
- Use `-alpha` suffix. Drop suffix only for stable releases.
- Create annotated tags; changelog per [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## Don'ts

- **Never modify `tool_result.content`** outside the 4-rule pattern — breaks LLM prefix cache. Modifications are allowed if they are static-cutoff, one-shot, byte-deterministic, and stable-position. The write-directory-fallback (Phase 6) follows this pattern.
- **Never touch content fields** (`command`, `code`, `oldText`, `newText`, `text`, `content`) — structural repairs only.
- **Never add external runtime dependencies** — TypeScript + Vitest only.
- **Never add repairs for external extension tools** (`agent_browser`, `web_search`, `fetch_content`) — generic `getToolHelp` only.
- **Never rename or delete a repair function** without updating `docs/repair-catalog.md` and checking test coverage.
- **Never commit code without tests** — every repair function needs colocated tests in `repairs/*.test.ts`.

---

## Architecture (summary)

3-layer handler: `tool_call` (pre-execution repair + validate) + `tool_result` (analytics + guidance queue) + `context` (side-channel injection). 8 field-level repairs via `repairDispatchers` lookup table + 1 inline (null-like-to-undefined in `repairObjectFieldsWithTrace`); 3 classification predicates; 8 constant sets. Full module map + execution order: `docs/architecture.md`.

---

## References

- `docs/cache-safety.md` — Cache-safety contract (the "why" of the 3-layer design)
- `docs/architecture.md` — Deep architecture: module map, 9 repairs, dispatch table
- `docs/repair-catalog.md` — Source of truth for repair function signatures
- `repairs/test-coverage.test.ts` — Guard: every export has a test
- `repairs/catalog-drift.test.ts` — Guard: catalog ⇄ source consistency
- `extension-integration.test.ts` — Fake `ExtensionAPI` + lifecycle test
- `scripts/hooks/pre-commit` — Pre-commit hook (vitest, opt-in via `npm run setup:hooks`)
- `/skill:cali-agents-md-validator` — Validate this file against 13 rules
- `/skill:cali-post-execution-check` — Verify work after multi-step tasks
- `cali-agents-md-generator` — Generate or refactor AGENTS.md (if this file gets stale)
- `cali-skill-validator` — Validate skill files (if you create new skills)
- `cali-skill-creator` — Create new skills from scratch (if needed)

---
> Source: [calionauta/pi-tool-repair-layer](https://github.com/calionauta/pi-tool-repair-layer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
