## record

> Use Conventional Commits for commit messages:

# AGENTS.md

## Git Workflow

Use Conventional Commits for commit messages:

```text
<type>[optional scope]: <description>
```

Use Conventional Commit-style prefixes for branch names:

```text
<type>/<scope-or-short-description>
```

Prefer these scopes when they fit: `capture`, `transcribe`, `record-plugin`, `deps`, `build`, `ci`, and `docs`. Use short, lowercase, hyphen-separated branch names.

Examples:

```bash
feat/capture-window-selection
fix/transcribe-missing-model
docs/record-plugin-validation
feat(capture): add window selection
fix(transcribe): report missing model setup
docs(record-plugin): document smoke validation
```

Never include `codex` in branch names or commit messages.

## Project Context

This repository has native Swift capture and TypeScript transcription runtimes behind the Homebrew-distributed `record` CLI. The agent plugin source lives in `plugins/record`.

## Scope

These instructions apply to the whole repository. Read the nearest package `AGENTS.md` before editing scoped package code.

## Commands

```bash
npm install
npm run setup:capture
npm run setup:transcribe
npm run setup
npm run build:bundle
npm run build:binary:macos-arm64
npm run test:record
npm run doctor
npm run install:local
npm run plugin:smoke
```

Run package CLIs in development:

```bash
npm --workspace capture run capture -- <args>
npm --workspace transcribe run transcribe -- <args>
```

## Context Loading

| Working On | Read First |
| --- | --- |
| Capture CLI | `packages/capture/AGENTS.md` |
| Transcribe CLI | `packages/transcribe/AGENTS.md` |
| Codex plugin | `plugins/record/README.md` and `plugins/record/skills/<skill>/SKILL.md` |
| Claude plugin | `plugins/record/README.md` and `plugins/record/claude/skills/<skill>/SKILL.md` |

## Validation

- For CLI source changes, run the package help command and `npm run build:bundle`.
- For binary/install script changes, run `npm run build:binary:macos-arm64` when feasible.
- For plugin manifest, skill, helper script, or plugin README changes, run `npm run plugin:smoke`.
- Treat `doctor` failures from missing Screen & System Audio Recording permission, missing Whisper, or missing model as environment checks; report them instead of hiding them.

## Structure

- `packages/capture`: native ScreenCaptureKit recording CLI and on-demand app agent.
- `packages/transcribe`: local transcription CLI.
- `scripts/record`: unified installed CLI dispatcher.
- `packaging/marketplace`: marketplace bundled with Homebrew releases.
- `plugins/record`: Codex plugin plus the nested Claude plugin, skills, and helper scripts.
- `dist/`, `node_modules/`, `runs/`, recordings, transcripts, and local config are generated and must stay untracked.

## Boundaries

- Keep capture and transcription runtime code separate behind the `record` command.
- Do not introduce private package registries, local absolute paths, or machine-specific plugin metadata files.
- Do not commit recordings, transcripts, local run output, build artifacts, dependencies, secrets, or environment files.
- Treat recording and transcription artifacts as sensitive by default.

---
> Source: [jlave-dev/record](https://github.com/jlave-dev/record) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
