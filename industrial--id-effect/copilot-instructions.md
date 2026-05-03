## llm-git-context

> tool-llm-git-context — vault JSONL logs, session-init, Cursor hooks; CLI-only mutations, devenv-wrapped commands, binary for hooks.


# LLM git context (`tool-llm-git-context`)

Persist **Cursor hook payloads** under **`.tool-llm-git-context/vault/Logs/`**. Default hooks write **`{UTC_YYYYMMDDHHMMSS}-{conversation_id}.jsonl`** (per chat, ordered by timestamp prefix). Legacy **`session-init`** paths use **`<stem>.jsonl`**. The monorepo is located via **`git rev-parse --show-toplevel`** (for `--repo` default only); the vault is **not** a git repository unless you add one yourself.

## When to run what

| Situation | Action |
|-----------|--------|
| **Fresh clone** | `devenv shell -- cargo run -p tool-llm-git-context -- init` (once — creates `vault/` + `vault/Logs/`). |
| **Install / refresh Cursor hooks** | `devenv shell -- cargo run -p tool-llm-git-context -- setup-cursor` (merges `.cursor/hooks.json`, writes `llm-git-context-append.sh`). |
| **New persisted LLM session** (new `events_log` path + `current-session.json`) | `devenv shell -- cargo run -p tool-llm-git-context -- session-init` |
| **LLM handoff to sync session log → vault notes** | `devenv shell -- cargo run -p tool-llm-git-context -- session-handoff --log <session.jsonl>` — path is a dated `vault/Logs/*-<conversation_id>.jsonl` or legacy `events_log` from `session-init`; prints a brief + TOON for context. The **LLM inspects the session and writes/updates Markdown under the vault** (not treating stdout as the deliverable); do not hand-edit JSONL. |
| **Append one JSON line manually** (debug) | `append-event` with stdin JSON containing **`conversation_id`** (optional `--repo`; writes under `vault/Logs/`) |
| **Validate stdin JSON only** (no write; CI / fixtures) | `devenv shell -- bash -c 'echo '"'"'<json>'"'"' \| cargo run -p tool-llm-git-context -- verify-event'` — same schema as append; exit 0 if valid |

## Rules for agents

1. **Shell:** Always `devenv shell --` for `cargo run -p tool-llm-git-context` (see `shell.mdc`).
2. **`current-session.json`** — Written by **`session-init`** (optional metadata + legacy `events_log`). Default hooks use **`append-event`** and do **not** require this file.
3. **Session JSONL** under **`vault/Logs/`** — **Append-only.** Never use `apply_patch` / `Write` / `StrReplace` on it. Hooks run **`append-event`** (no git commits from this tool).
4. **Hooks need a binary** — Cursor does not use devenv PATH. Ensure `target/debug/tool-llm-git-context` exists (`cargo build -p tool-llm-git-context`) or set **`LLM_GIT_CONTEXT_BIN`** to an absolute executable. The hook script resolves debug/release binaries under the monorepo root automatically.
5. **Vault vs monorepo** — Obsidian notes live under **`.tool-llm-git-context/vault/`** (same tree as `Logs/`). Do not confuse the vault folder with the host app repo root unless you open the monorepo as the vault on purpose.
6. **Implementation details** — `crates/tool-llm-git-context/README.md` and source under `crates/tool-llm-git-context/src/`.

## Related

- **Slash command:** `.cursor/commands/llm-git-context.md` (`/llm-git-context`)
- **Skill (discoverable):** `.cursor/skills/tool-llm-git-context/SKILL.md`
- **Activate bootstrap:** `.cursor/commands/activate.md` includes this tool in the session checklist

---
> Source: [Industrial/id_effect](https://github.com/Industrial/id_effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
