## ai-coding-pm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Backend**

```bash
cd backend && uv sync              # install dependencies
uv run pytest                      # run all backend tests
uv run pytest tests/test_main.py::test_name  # run a single test
uv run uvicorn app.main:app --host 127.0.0.1 --port 8000  # run locally
```

**Frontend**

```bash
cd frontend && npm ci              # install dependencies
npm run dev                        # run locally
npm run test:unit                  # unit tests (vitest)
npm run build                      # static export to frontend/out/
npm run test:e2e:integration       # Playwright tests (requires app running)
```

**Docker (full stack)**

```bash
./scripts/start-mac.sh             # start
./scripts/stop-mac.sh              # stop
```

## Architecture

The app is a local kanban board with AI chat. FastAPI serves both the API and the Next.js static export.

```
backend/app/
  main.py        # app factory (create_app), all API routes
  models.py      # Pydantic domain models: Board, Card, Column, ChatReply, AiStructuredReply
  repository.py  # BoardRepository — all SQLite access
  ai.py          # OpenRouterClient, build_chat_messages

frontend/src/
  lib/kanban.ts                  # BoardData type and pure board logic (moveCard, createId)
  components/AppShell.tsx        # auth gate, login form, session lifecycle
  components/KanbanBoard.tsx     # main board UI, backend API calls
  components/AiChatSidebar.tsx   # chat sidebar, sends/receives messages
```

**Data flow**

- On login, the frontend calls `POST /api/chat/{username}/session` to reset the backend chat session.
- The board is fetched from `GET /api/board/{username}` and saved via `PUT /api/board/{username}`.
- Chat messages are sent to `POST /api/chat/{username}/messages`, which calls OpenRouter and returns both an AI reply and an updated board (if the AI chose to mutate it).
- The frontend replaces the board state in-place when a chat reply includes a board update.

**Board invariants (enforced by `Board.validate_consistency`)**

- Exactly five columns with fixed ids in order: `col-backlog`, `col-discovery`, `col-progress`, `col-review`, `col-done`.
- Column ids are unique; each card appears in exactly one column; each card dict key matches its `id` field.
- Board mutations from AI are validated against these rules before being persisted.

**Storage**

SQLite at `backend/data/app.db`. Board states are immutable append-only snapshots. `boards.current_board_state_id` points to the latest. `chat_messages` rows reference the `board_state_id` visible at that turn.

**AI**

OpenRouter model `openai/gpt-oss-120b`. The AI is instructed to return JSON matching `AiStructuredReply`: `{"reply": "...", "board_update": null | {"kind": "replace_board", "board": {...}}}`. The backend validates the full returned board before persisting it.

**Auth**

Frontend-only gate: hardcoded credentials `user` / `password`. Session state lives in `sessionStorage`. The backend identifies users by the `username` path parameter (always `"user"` for the MVP).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evokateur)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/evokateur)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
