## ai-research-assistant

> Rules for agents working in this repo.

# AGENTS.md

Rules for agents working in this repo.

## Tooling

- This repo's code is generated and maintained with DeepSeek-Harness (dsh)
  plus code-review-graph (CRG).
- **Explore graph-first, grep second**: run `build_or_update_graph_tool` with
  `repo_root` set to this repo after every change, and query the knowledge
  graph (semantic search, impact radius, review context) before falling back
  to grep/glob over files.
- Run dsh from your local checkout of deepseek-harness, with the OpenRouter
  patch applied, e.g.:
  `OPENROUTER_API_KEY=... pnpm dsh --profile headless --patch
  openrouter.patch.yml "<task>"`.
- Model routing mandate: use **Flash** (`deepseek/deepseek-v4-flash-0731`) for
  codegen and structure work; use **Pro** for reasoning and anomaly
  investigation. Do not swap these roles casually.

## Do not commit

- Secrets: never commit API keys, `.env`, or anything resembling a
  credential. `.env` is the only place keys live (gitignored).
- Generated reports: `research/reports/*.html` are runtime artifacts and are
  gitignored — never add them. `research/index.html` is generated at runtime;
  do not commit its generated content either.
- The code-review graph database under `.code-review-graph/` is gitignored.

## Before changing anything

- If `MEMORY.md` is present at the repo root, read it first and honor it
  before making changes.

## Behavior

- Follow the SPEC (`SPEC.md`) as the source of truth for structure, endpoints,
  retention semantics, and the Docker layout.
- Shell scripts use `set -euo pipefail` and quote all expansions.
- No secrets anywhere in committed files; comments explain the WHY for
  non-obvious decisions (glibc, DONGHOST_NO_SANDBOX, stripped `/api` prefix,
  `-2` suffix logic).

---
> Source: [rpmalouin/AI-Research-Assistant](https://github.com/rpmalouin/AI-Research-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
