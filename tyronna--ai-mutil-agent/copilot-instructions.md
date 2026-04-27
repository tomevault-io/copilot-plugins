## ai-mutil-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make install           # Create venv, install deps, install Playwright Chromium
make run TASK="..."    # Run Expo pipeline (requires EXPO_PROJECT_DIR in .env)
make run-no-git        # Run without git commit/PR creation
make run-no-test       # Run without browser screenshot
make web               # Start FastAPI on :8000 + Next.js UI on :3000
make web-reload        # Start web with auto-reload
make test              # Run pytest
make lint              # Python syntax check
make clean             # Remove venv + caches
```

CLI entry point (after `make install`, `source .venv/bin/activate`):
```bash
agent run "Add dark mode toggle"               # Expo pipeline
agent run --dir ~/Projects/my-app "..."        # Explicit project dir
agent game "Add daily reward popup"            # Game pipeline
agent game --workers 2 "..."                   # Override parallel workers
agent serve --port 8000 --reload               # Web UI
```

Run a single test:
```bash
pytest tests/test_orchestrator.py::test_name -v
```

## Environment

Copy `.env.example` to `.env`. Key variables:
- `GITHUB_TOKEN`, `GITHUB_REPO` — for PR creation
- `EXPO_PROJECT_DIR` — path to the target Expo app
- `GAME_PROJECT_DIR` — path to the Mộng Võ Lâm game repo
- `WEBHOOK_URL` — optional Slack/Discord notification
- `MODEL` — override default `gemini-3-flash-preview`

Vertex AI credentials must be in `config/vertex-ai.json` (service account key, not committed).

## Architecture

Two independent pipelines share agents, tools, and the LLM layer:

### Expo Pipeline (`src/orchestrator.py`)
Builds Expo React Native features end-to-end:
1. **AnalyzerAgent** — reads source files, extracts conventions, runs `tsc --noEmit` to detect pre-existing errors
2. **PlannerAgent** — decomposes task into up to 5 ordered subtasks, each targeting specific files
3. **CoderAgent** + **ReviewerAgent** loop — runs in parallel threads (up to 3 workers); Coder writes complete diffs file content (never patches); Reviewer reads from disk and checks TypeScript errors + conventions
4. **TesterAgent** — non-LLM; starts Expo web server, captures Playwright screenshots
5. **GitAgent** → **NotifierAgent** — commit, push, open GitHub PR, send macOS notification

Shared state is `AgentState` (`src/state.py`) — a single mutable object passed through the pipeline.

### Game Pipeline (`src/orchestrator_game.py`)
Builds Phaser 4 / JavaScript features for the Mộng Võ Lâm game:
1. **GameLoader** (`src/context/game_loader.py`) — loads game source into static (cached, ~24K chars) and dynamic (~120K chars) tiers; only static tier goes into Gemini Context Cache
2. **TechExpertAgent** (Gemini Pro) — plans subtasks, test scenarios, and architectural constraints
3. **DevAgent** + **QAAgent** loop — same parallel pattern as Expo pipeline
4. **TechExpertAgent** — final architecture review before commit
5. **GitAgent** → **NotifierAgent** — same as Expo

State is `GameAgentState` (`src/state_game.py`).

### LLM Layer (`src/llm/__init__.py`)
- **Backend**: Vertex AI via `google-genai` SDK with service account auth
- **Models**: `gemini-3-flash-preview` (default, fast) and `gemini-3-pro-preview` (planning/review, via `pro=True`)
- **Retry**: exponential backoff on 429 and 5xx errors
- **Context Cache**: `create_context_cache(content)` caches static context (game source, codebase conventions) for reuse across multiple calls within a subtask. Falls back to full prompt if content is below the 32K token threshold.
- **Thinking tokens**: `thinking_budget=4096` for TechExpert planning; `thinking_budget=1024` for QA static analysis; `thinking_budget=0` for TechExpert review (reads diff, no reasoning needed)
- **Structured output**: pass a Pydantic model as `response_schema` for JSON mode

### Web Server (`src/web/server.py`)
FastAPI app with:
- `POST /run` — starts pipeline in a background thread, returns `session_id`
- `GET /ws/{session_id}` — WebSocket that streams real-time agent progress via `asyncio.Queue`
- `GET /agents` — agent metadata (name, icon, role, description)
- Serves Next.js build from `ui/out/` or falls back to `src/web/static/`

### Web UI Navigation (`ui/app/page.tsx`)
- Main views: `pipeline`, `tasks`, `queue`, `analytics`, `preview`
- Desktop top tabs include all five main views above
- Mobile bottom navigation includes all five main views above (including `preview`)
- Mobile `pipeline` view has sub-tabs: `form` and `feed`

### Key Design Decisions
- **Coder always returns diffs file content** — never patches all. Reviewer reads the file from disk after Coder writes it.
- **TechExpert planning — search-first, no full dump**: `_build_plan_prompt` runs keyword code search before including the dynamic context. The 120K dynamic context is only sent as a fallback when search returns no results, saving ~43K tokens on typical tasks.
- **Subtask parallelization** is safe because the Planner is instructed to assign non-overlapping files per subtask.
- **Convention extraction** is first-class: AnalyzerAgent output flows into every downstream agent's prompt to enforce project-specific patterns.
- **QA receives unified diff** (original → written) instead of full file content. Originals are captured in `subtask.original_files` before first Dev write. This reduces QA prompt from ~10–22K tokens to ~500–2K tokens per subtask.
- **Progress callbacks**: `state.log(msg, agent=name)` appends to `state.messages` and fires an optional `progress_cb` — used by the Web server to push WebSocket updates.

## Game Pipeline Invariants (enforced by QAAgent)

These constraints are architectural rules the DevAgent must follow and QAAgent validates:
- `CombatEngine.js` — pure JavaScript only, zero Phaser imports ever
- Colors via `UI_THEME` from `constants.js` — no bare hex literals like `0x0000ff`
- `SaveManager`: always `load()` → modify → `save()` — never access `localStorage` directly
- All text rendered with `crispText()`, all scene transitions via `gotoScene()`
- Vietnamese text must include full diacritics (e.g., `'Chọn'` not `'Chon'`)
- Combat formula: `final = rawDmg * (DEF_K / (DEF_K + DEF)) * crit`
- Status effects: `{type: 'stun'|..., remaining: N}`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TyronNA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
