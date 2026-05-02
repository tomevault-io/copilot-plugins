## ai-publish

> Short operating manual for humans and coding agents working on ai-publish.

# AGENTS.md

Short operating manual for humans and coding agents working on ai-publish.

## What this repo is

ai-publish is an AI-assisted release authoring tool that generates:

- a deterministic, Keep a Changelog–style changelog entry
- consumer-facing release notes
- a next version recommendation

It is designed to be **auditable** and **prompt-injection resistant**.

## Non-negotiable invariants (do not break)

1. **Diff-only authority**: the only authority for “what changed” is `git diff <base>..HEAD`.
2. **Index, don’t ingest**: parse the diff into hunks and store hunks out-of-band.
3. **No full diff API**: never expose an API that returns the entire patch.
4. **No full file context API**: repo context must be slice-based with strict budgets.
5. **Binary-safe**: binary changes are represented as metadata only.
6. **Evidence-backed output**: every model bullet must reference explicit evidence node IDs.
7. **Determinism**: for the same inputs, ordering must be stable (sections, files, evidence IDs).

If a change conflicts with these, redesign it.

## Golden path (how it works)

1. **Index the diff** (authoritative): `indexDiff({ base })`
   - Writes `.ai-publish/diff-index/<baseSha>..<headSha>/manifest.json` (metadata only)
   - Writes `.ai-publish/diff-index/<baseSha>..<headSha>/hunks/<hunkId>.patch` (one file per hunk)
2. **Build evidence** from the manifest: `buildEvidenceFromManifest(manifest)`
3. **LLM pipeline (3-pass)**
   - Mechanical: enumerate factual notes from metadata
   - Semantic: request bounded evidence/context and produce evidence-backed notes
   - Editorial: produce the final structured model / release notes
4. **Validate** before writing output: `validateChangelogModel(model)`

## LLM pass contract (agent notes)

Changelog and release notes generation intentionally follows a **3-pass** LLM contract:

1. **Pass 1 (Mechanical)**: metadata-only inputs → “mechanical notes” (no patch text).
2. **Pass 2 (Semantic)**: the model may request _bounded_ context via tools.
   - Diff content is only accessible via `getDiffHunks(hunkIds)` and must be **hunk-ID addressed**.
   - Tooling is **gated and budgeted** (byte caps); requests beyond the remaining budgets must fail.
   - Commit messages, when provided, are **untrusted context** and must never be treated as evidence.
3. **Pass 3 (Editorial)**: structured final output.
   - Changelog bullets must reference explicit evidence node IDs and must validate against the evidence set.
   - Release notes must include explicit `evidenceNodeIds`; “markdown with zero evidence” must be rejected.

Version bump recommendation is **deterministic** from the changelog model, and the LLM is only used to provide a human-readable justification. The LLM is required to echo the exact computed `nextVersion`; mismatches must fail.

CLI and programmatic entrypoints follow this structure.

## Core modules (where to look)

- Diff indexing + retrieval: `src/diff/`
  - `indexDiff()` → shells out to `git diff` and indexes hunks
  - `getDiffHunks()` → bounded retrieval by hunk ID (default `maxTotalBytes = 256 KiB`)
- Evidence + changelog model + render: `src/changelog/`
  - Evidence nodes are **file-level** and contain `hunkIds`
  - Renderer: `renderKeepAChangelogMarkdown()`
- Pipelines: `src/pipeline/`
  - `runChangelogPipeline()` / `runReleaseNotesPipeline()` / `runVersionBumpPipeline()`
- LLM clients + schemas: `src/llm/`
  - Supported providers: `azure` and `openai` (CLI + programmatic)
  - Structured outputs via JSON schema
- CLI: `src/cli.ts`
- Programmatic API: `src/programmatic.ts`
- Bounded repo context tools (HEAD-only): `src/repo/`
- Tests: `test/` (Vitest; temp git repos as fixtures)

## On-disk diff index contract (important)

Each stored hunk file is UTF-8 text and begins with:

- `file: <path>`
- optional `oldFile: <path>`
- an `@@ ... @@` header line (or `@@ meta @@`)
- then bounded diff lines

Hunks are truncated deterministically and include the marker:

`\ No newline at end of truncated hunk (ai-publish)`

Rename-only / binary / other hunkless changes produce a metadata-only pseudo-hunk (`@@ meta @@`) so downstream output still has evidence.

## Bounded context rules

Repo context (snippets/searches/meta) is allowed **only** to interpret impact. It is never evidence of what changed.

Commit messages are **untrusted**. They may be provided as optional bounded context for `base..HEAD` but must never be treated as evidence.

## Determinism checklist

When you change behavior:

- Keep stable ordering: sort by path, ID, or other explicit keys.
- Avoid timestamps/randomness/filesystem order.
- Keep output section ordering fixed.

## CLI + provider reality

- `--llm` is required for `changelog`, `release-notes`, and `prepublish`.
- Valid values are `azure` and `openai`.
- `postpublish` does not use the LLM.

## Test expectations

- Add/adjust Vitest tests for any behavioral changes.
- Deterministic/offline tests should run without network access.
- Azure-backed LLM tests exist but are **local-only** (require Azure env vars and may make network calls):
  - `npm run test:llm-eval`
  - `npm run test:llm-generate`

## Linting + typecheck

- Lint: `npm run lint` (ESLint v9 flat config in `eslint.config.js`; scoped to `src/**/*.ts`).
- Auto-fix: `npm run lint:fix` (use for safe mechanical fixes; avoid broad rule disables).
- Typecheck: `npm run typecheck`.

## Logging

To keep machine-readable stdout parseable, logs go to stderr.

Useful env vars:

- `AI_PUBLISH_LOG_LEVEL`: `silent|info|debug|trace`
- `AI_PUBLISH_TRACE_TOOLS=1`: logs bounded tool usage (counts + budgets)
- `AI_PUBLISH_TRACE_LLM=1`: logs LLM request/response metadata

---
> Source: [emilmuller/ai-publish](https://github.com/emilmuller/ai-publish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
