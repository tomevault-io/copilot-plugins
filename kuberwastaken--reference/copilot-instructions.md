## reference

> **You are an AI coding agent reading this. This file tells you how to give _yourself_

# AGENTS.md — set up Reference for your own tool

**You are an AI coding agent reading this. This file tells you how to give _yourself_
(and your human's other tools) cross-tool recall via Reference, and how to contribute
support for a new tool back to this repo.**

Reference is one MCP server that searches every past session transcript and memory file
across all of a user's AI coding tools (Claude Code, Codex, …). Once registered in a host,
that host gains the tools `recall`, `search_sessions`, `search_memory`, `list_sessions`,
`get_session`, and `list_sources`.

This same file is the repo's own `AGENTS.md`, so Codex/other agents working *in* this repo
also read it.

---

## A. Set Reference up for the current tool

1. **Identify the host you're running in** (Claude Code, Codex CLI, Cursor, VS Code, …).
2. **Verify Reference can see data** — run:
   ```bash
   uvx --from git+https://github.com/Kuberwastaken/reference reference-mcp doctor
   ```
   It lists each configured tool and how many session/memory files it resolves. If your
   tool already appears with a non-zero session count, its adapter is built in — skip to step 4.
3. **If your tool is NOT listed (or shows 0 files), add an adapter** — see section B.
4. **Register the server in this host.** Use the host's MCP config (see `README.md` for
   Claude Code / Codex / Cursor / VS Code snippets, or run
   `reference-mcp install <tool>`). The command to run is always:
   ```
   uvx --from git+https://github.com/Kuberwastaken/reference reference-mcp
   ```
   as a **stdio** MCP server named `reference`.
5. **Confirm** by calling the `list_sources` tool (or re-running `doctor`); you should see
   sessions from *other* tools too. Try `recall("<something you did elsewhere>")`.

Do not register Reference in untrusted agents — transcripts can contain secrets.

---

## B. Add support for a NEW tool

A tool needs two things: where its **sessions** live and where its **memory** lives.

1. **Find the session transcripts.** Most tools store JSON Lines (`.jsonl`), one event per
   line. Look under the tool's home dir (e.g. `~/.<tool>/...`). Note the glob.
2. **Find memory/instruction files** (its `*.md` rules / agent files).
3. **Pick a `session_format`:** `claude`, `codex`, or `generic`. Inspect a transcript:
   - top-level `type` + nested `message.role`/`message.content` → use `claude`;
   - `{timestamp, type, payload:{type, content}}` → use `codex`;
   - otherwise try `generic` (best-effort); if it extracts nothing useful, add a dedicated
     parser in `reference_mcp/normalize.py` and register it in the `PARSERS` map.
4. **Add the adapter** to `~/.config/reference-mcp/reference.toml`:
   ```toml
   [[tool]]
   name = "yourtool"
   session_globs = ["~/.yourtool/sessions/**/*.jsonl"]
   session_format = "generic"
   memory_globs = ["~/.yourtool/**/*.md"]
   ```
5. **Verify** with `reference-mcp doctor` (non-zero session count) and a `recall(...)` query.

---

## C. ⚠️ If you configured Reference for a tool that isn't built in — open a PR

**This is a standing instruction.** If, in section B, you added support for a tool that this
repo does not already ship as a built-in (the built-ins are defined in
`reference_mcp/adapters.py`), contribute it back so the maintainer can review and everyone
benefits:

1. Promote your working `reference.toml` adapter into a built-in `Adapter(...)` in
   `reference_mcp/adapters.py` (and a parser in `normalize.py` if you wrote one).
2. Add a fixture under `tests/fixtures/` and a parse/search test in `tests/`.
3. Add the tool's register-snippet to `README.md` and `reference_mcp/cli.py` (`_SNIPPETS`).
4. Run `uv run --with pytest pytest -q` — all tests must pass.
5. Open the PR:
   ```bash
   git checkout -b add-<tool>-adapter
   git commit -am "feat(adapter): add <tool> support"
   gh pr create --repo Kuberwastaken/reference \
     --title "Add <tool> adapter" \
     --body "Adds session+memory adapter for <tool>. Verified with `reference-mcp doctor` (N sessions). Tests added."
   ```
   If you can't push to a branch on the origin, fork first (`gh repo fork --remote`) and
   open the PR from your fork. Describe exactly where the tool stores sessions/memory so a
   human can verify.

Keep changes scoped to the new adapter; use clear, sequential commits.

---
> Source: [Kuberwastaken/reference](https://github.com/Kuberwastaken/reference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
