## codex-prompts

> - Structure: numbered prompts in `prompts/`; slugged meta artifacts in `.prompts/{slug}-{purpose}/` (create the folder if missing).

# Repository Guidelines

Read this first:
- Structure: numbered prompts in `prompts/`; slugged meta artifacts in `.prompts/{slug}-{purpose}/` (create the folder if missing).
- Practices: use `rg` for search, `apply_patch` for single-file edits, default to ASCII, avoid destructive git commands.
- Naming: zero-padded numeric prefixes for runnable prompts; slugged names (no numbers) for meta flows.
- Metadata: meta outputs include `## Metadata` with status/confidence/dependencies/open questions/assumptions.

## Project Structure & Module Organization
- Root contains `prompts/` with Codex-oriented helper prompts.
- `prompts/create-prompt.md` — meta-prompt to generate task-specific prompts.
- `prompts/run-prompt.md` — runner/orchestrator prompt for executing stored prompts.
- `prompts/create-meta-prompt.md` — Codex adaptation of the Claude meta-prompt creator; uses descriptive slugs (no numeric prefixes) for research/plan/do prompt folders under `.prompts/`.
- Slugged meta prompts live under `.prompts/{slug}-{purpose}/`; keep them separate from numbered files in `prompts/`. Create the `.prompts/` subfolders as needed.
- Add new task prompts under `prompts/` using zero-padded numeric prefixes when generating runnable tasks (e.g., `001-feature.md`), leaving meta files unnumbered. Meta-prompt outputs should use descriptive slugs (avoid `NNN-*` prefixes).
- Before creating new prompts, skim existing ones (numbered and slugged) to avoid duplication and to reuse relevant context.

## Build, Test, and Development Commands
- No build tooling or runtime stack in this repository; it is prompt content only.
- If you add supporting scripts, note them in this file and in any prompt that relies on them (paths, invocation).

## Coding Style & Naming Conventions
- Markdown throughout; keep sections lean and scannable.
- Use descriptive, hyphenated filenames; zero-pad numeric prefixes for runnable prompts (see `prompts/create-prompt.md` for numbering guidance).
- Follow practices noted inside `prompts/create-prompt.md` and `prompts/run-prompt.md` (e.g., prefer `rg` for search, `apply_patch` for single-file edits, avoid destructive git commands, default to ASCII).

## Testing Guidelines
- No automated tests exist for this prompt repository.
- If no tests apply for a prompt, state that explicitly; when suggesting downstream commands, align with the target repo and note sandbox limits.
- When adding prompts that instruct running tests in downstream projects, specify exact commands and paths in those prompts.

## Commit & Pull Request Guidelines
- Use concise, lowercase commit messages with a clear type prefix where appropriate (e.g., `chore: add agent guide`, `docs: update create-prompt`).
- One focused change per commit; avoid bundling unrelated edits.
- PRs should summarize changes, note any assumptions, and call out any skipped verification (if applicable to downstream usage).

## Agent-Specific Instructions
- For meta-prompt workflows, avoid XML in prompt content and use Markdown metadata sections instead (`## Metadata` with `### Confidence/Dependencies/Open Questions/Assumptions`).
- Name generated prompt folders/files with descriptive slugs (no `NNN-*` prefixes).
- Keep prompts concise: minimal intake rounds, explicit file paths, verification steps that note sandbox limits, and no leftover placeholders.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaronlovi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aaronlovi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
