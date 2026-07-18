## cc-to-antigravity-cli-bridge

> Context for AI assistants working in this repository. `CLAUDE.md` is a symlink to this file. Antigravity CLI (`agy`) reads `AGENTS.md` from the working directory natively.

# AGENTS.md

Context for AI assistants working in this repository. `CLAUDE.md` is a symlink to this file. Antigravity CLI (`agy`) reads `AGENTS.md` from the working directory natively.

## What this project is

A minimal bridge for driving Google's Antigravity CLI (`agy`) from Claude Code (CC). It is the CLI counterpart to the IDE-based `claude-code-antigravity-ide-bridge`, and the third in a series of CC harness bridges (Grok, Codex, Antigravity).

The honest scope is small. As of 2026-07, `agy` has no native memory API and no documented hook schema, so a full bridge (memory sync, hook adapters) is not possible. What works is system-prompt sharing and a research-method prompt prefix that lifts search quality. See `README.md`.

## What's here

- `research-prefix.md`: model-agnostic research-method prompt prefix (works for `agy -p`, `grok -p`, `claude -p`)
- `scripts/ccagy.sh`: wrapper that prepends the prefix and runs `agy -p` from a clean temp cwd
- `README.md` (EN default), `README.zh-TW.md`, `README.zh-CN.md`
- `AGENTS.md` / `CLAUDE.md`: shared system prompt (symlink)

## Key facts

- `agy` runs on your Gemini subscription quota when logged in with Google. Do not set `GEMINI_API_KEY`, which forces API billing instead.
- `agy -p "..."` is the non-interactive one-shot mode; `--model "..."` takes the exact display name from `agy models`.
- System prompt is shared by symlinking `AGENTS.md` to your `CLAUDE.md`.
- No memory sync and no hook-level security: `agy` has neither a memory API nor a documented hook schema as of 2026-07.

## Security notes for AI assistants

- Do not read `.env` files or files named `credentials.json`, `service-account.json`
- Do not read files whose names contain `secret`, `token`, `key`, or `credential`
- Before any `git push`, confirm the diff contains no API key patterns (`sk-ant-`, `AIza`, `ghp_`, etc.)

---
> Source: [yyu0310/cc-to-antigravity-cli-bridge](https://github.com/yyu0310/cc-to-antigravity-cli-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
