## preprompt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Activate venv first
source .venv/bin/activate

# Run all tests
python -m pytest

# Run a single test file
python -m pytest tests/test_classifier.py -v

# Install in editable mode (includes dev deps)
pip install -e ".[dev]"

# Start the MCP server
python -m mcp_server.server

# CLI tools (after pip install)
preprompt-history [--limit N] [--intercepted-only]
preprompt-stats
preprompt-test-classifier
preprompt-memory
preprompt-optimize "your prompt here"   # or pipe: echo "..." | preprompt-optimize
preprompt-optimize --raw "prompt"        # prints optimized text only
preprompt-watch                          # live tail of ~/.preprompt/activity.log
preprompt-clip                           # read clipboard, optimize, write back (macOS)
preprompt-feedback                       # rate recent optimizations (builds accept/reject stats)
# Note: stats and history auto-run first-time API key wizard via cli/setup.py
preprompt-install                        # one-command setup (API key + hooks)
preprompt-update                         # upgrade to latest PyPI version + re-register hooks
preprompt-update-context

# One-command install + global hook registration (auto-detects Claude Code, Cursor, Windsurf, Zed)
bash scripts/install.sh

# Per-IDE registration
python scripts/install_cursor.py
python scripts/install_windsurf.py
python scripts/install_zed.py
```

## Architecture

PrePrompt is an MCP server that intercepts prompts before they reach the LLM, scores them with a local heuristic classifier, optionally rewrites vague/complex ones via Claude Haiku, and logs everything to a local SQLite DB.

### Request flow

```
User types prompt
    → .claude/hooks/pre_prompt.py  (UserPromptSubmit hook, subprocess)
        → classify_prompt()        (score 0–100, no API call)
        → if score < 38: passthrough
        → else: optimize() via Haiku API
        → write JSON sidecar → ~/.preprompt/pending/<uuid>.json
        → print optimized prompt to stdout
    → Claude Code sends optimized prompt to LLM
    → (next MCP tool call)
        → flush_pending_hook_events()  reads sidecars → SQLite
```

The hook **never touches the DB directly** — this avoids SQLite lock contention with the running MCP server. The sidecar files are the IPC mechanism.

### Key modules

| File | Responsibility |
|------|---------------|
| `mcp_server/classifier.py` | Heuristic scorer. `OPTIMIZATION_THRESHOLD = 38`. No API calls. |
| `mcp_server/optimizer.py` | Calls Claude Haiku to rewrite prompt. |
| `mcp_server/tools.py` | MCP tool `optimize_prompt()` — entry point from Claude Code. Calls `flush_pending_hook_events()` first. |
| `mcp_server/extractor.py` | Extracts stack facts (language, framework, etc.) from prompts → `stack_memory`. |
| `storage/db.py` | SQLite (WAL mode). Tables: `prompt_history`, `stack_memory`, `sessions`. `_get_connection()` is the long-lived write conn; `get_read_connection()` opens fresh read conns for CLI/history. |
| `cli/commands.py` | CLI entry points. Always calls `flush_pending_hook_events()` before reading history. Includes `optimize_cmd` for standalone CLI optimization. |
| `.claude/hooks/pre_prompt.py` | Hook subprocess. Resolves project root via `__file__` (not cwd). Writes sidecars, never imports `storage.db`. |
| `scripts/install_windsurf.py` | Registers MCP in `~/.codeium/windsurf/mcp_config.json`. |
| `scripts/install_zed.py` | Registers MCP in `~/.config/zed/settings.json`. |
| `.github/workflows/publish.yml` | Publishes to PyPI on `git tag v*` via OIDC trusted publisher. |
| `preprompt.skill.md` | Claude Skill file — manual PrePrompt scoring/optimization for tools without MCP. |

### Data directory

All runtime data lives in `~/.preprompt/`:
- `history.db` — SQLite database
- `pending/*.json` — sidecar files written by hook, flushed by MCP server

### Classifier scoring

Positive: ambiguity verbs (+25 max), multi-requirement density (12 pts/hit, max 30), turn depth (+5/turn after turn 2, max 15), code task without format hint (+15).  
Negative: short prompt (−20), "what is/are/does" opener (−15), already structured (−15), conversational opener (−25).

### SQLite type conventions

- Timestamps stored as ISO 8601 TEXT; parsed back to `datetime` via `_coerce_row()` on read.
- Booleans stored as INTEGER (0/1); coerced back to `bool` on read.
- All writes followed by `conn.commit()`.

### Session identity

One session per `hostname + calendar day`. Session key format: `{hostname}-{YYYY-MM-DD}`.

### Landing page

`docs/index.html` — GitHub Pages site (Tailwind CDN, Space Grotesk + JetBrains Mono, dark theme). Sections: hero terminal, interactive classifier demo, how it works, stack memory CLI mockup, session replay, stats bar, open source CTA, beta signup, FAQ. All interactivity is vanilla JS — no build step.

## Mandatory after every task

After completing any task in this repository, you MUST always:

1. Run `pytest -v` and confirm all tests still pass

2. Run `preprompt-update-context` to update CONTEXT.md

3. Update CLAUDE.md if any architecture, file map, or interface changed

4. Commit with a clear descriptive message authored by Yashdeep Tehlan:

   ```
   git -c user.name="yashdeeptehlan" -c user.email="0yashdeeptehlan@gmail.com" commit -m "your message here"
   ```

5. Push to main

Never skip these steps. Never use co-authored-by or mention AI tools in commit messages.

---
> Source: [yashdeeptehlan/preprompt](https://github.com/yashdeeptehlan/preprompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
