## prisma-review-tool

> PRISMA Review Tool — an open-source CLI + Web + MCP tool for PRISMA 2020 systematic literature reviews. Released at https://github.com/Black-Lights/prisma-review-tool

# Session System Implementation — Context for Agent

## What This Repo Is

PRISMA Review Tool — an open-source CLI + Web + MCP tool for PRISMA 2020 systematic literature reviews. Released at https://github.com/Black-Lights/prisma-review-tool

## Current Architecture

```
prisma_review/        Python CLI module (search, dedup, screen, export, download)
mcp_server/           MCP server (11 tools for AI-assisted screening)
api/                  FastAPI REST backend (wraps prisma_review)
web/                  Next.js 16 + Tailwind v4 frontend (dark glassmorphism theme)
```

## Your Task: Background Pipeline Session System

The pipeline (search → dedup → screen) currently runs **synchronously** — the HTTP request blocks for minutes. This causes:
- Browser shows "loading" indefinitely
- Page reload loses the running state
- No way to cancel mid-pipeline
- Pipeline stepper on dashboard doesn't update in real-time

### What You Need to Build

#### Backend: `api/session.py` (NEW)

`PipelineSession` class that:
- Runs pipeline steps in a `threading.Thread`
- Tracks state: `idle` | `running` | `completed` | `failed` | `cancelled`
- Tracks `current_step` (search/dedup/screen), `progress_message`, `started_at`, `error`
- Stores state in `review_state.json` under a `"pipeline"` key so it persists across restarts
- Has a `cancel_flag` (threading.Event) checked between steps
- Singleton pattern — only one pipeline can run at a time

#### Backend: Modify `api/routes/pipeline.py`

- `POST /api/pipeline/start` — launches pipeline in background thread, returns immediately `{"status": "started"}`
- `GET /api/pipeline/progress` — returns current state (step, status, message, counts). Must be FAST (just reads state, no computation)
- `POST /api/pipeline/stop` — sets cancel flag, pipeline stops after current step completes
- Keep existing individual step endpoints (`/search`, `/dedup`, `/screen`) working synchronously for CLI/direct use
- `POST /api/pipeline/start` accepts optional `steps` param: `["search", "dedup", "screen"]` to run specific steps

#### Frontend: `web/src/components/PipelineProgress.tsx` (NEW)

- Uses `useQuery` with `refetchInterval: 2000` when pipeline is running
- Shows animated stepper:
  - **Done** steps: green CheckCircle2 + count
  - **Running** step: animated spinner (Loader2 icon with `animate-spin`) + "Running..."
  - **Pending** steps: gray Circle
  - **Failed** step: red XCircle + error message
- "Stop Pipeline" button shown when running (uses Modal component for confirmation)
- Auto-stops polling when status is idle/completed/failed

#### Frontend: Modify `web/src/app/page.tsx` (Dashboard)

- Replace the current static pipeline stepper with `<PipelineProgress />`
- "Run All" button:
  - When idle → shows "Run All" (green), calls `POST /api/pipeline/start`
  - When running → shows "Stop" (red), calls `POST /api/pipeline/stop` (with Modal confirmation)
  - When completed → shows "Run Again" with warning Modal
- On page mount, fetch `/api/pipeline/progress` to restore state (survives reload)

### Key Files to Reference

- `api/routes/pipeline.py` — current synchronous pipeline endpoints
- `api/deps.py` — config loading, file lock
- `prisma_review/cli.py` — the actual pipeline logic (cmd_search, cmd_dedup, cmd_screen_rules)
- `web/src/app/page.tsx` — current dashboard with static stepper
- `web/src/components/Modal.tsx` — reusable modal for confirmations
- `web/src/lib/api.ts` — API client functions

### Design Rules

- **Dark glassmorphism theme**: bg-bg-base (#0a0e1a), text-text-primary (#e0e8f0), primary (#7dd3fc)
- Use existing CSS classes: `glass`, `glass-elevated`, `glass-input`, `glow-primary`
- Use `lucide-react` for icons (CheckCircle2, Circle, Loader2, XCircle, Play, Square)
- Use `@tanstack/react-query` for data fetching
- Import API functions from `@/lib/api`
- Import components from `@/components/*`

### State Shape in review_state.json

```json
{
  "search": { ... },
  "dedup": { ... },
  "screen": { ... },
  "eligibility": { ... },
  "pipeline": {
    "status": "running",
    "current_step": "dedup",
    "progress_message": "Deduplicating 2821 papers...",
    "started_at": "2026-03-30T12:00:00",
    "completed_steps": ["search"],
    "error": null
  }
}
```

### Testing

1. Run `npx next build` in `web/` to verify no TypeScript errors
2. Start backend: `uvicorn api.main:app --port 8001` (use 8001 to not conflict with main branch on 8000)
3. Start frontend: `NEXT_PUBLIC_API_URL=http://localhost:8001 npm run dev -- --port 3001`
4. Click "Run All" → pipeline should start in background → stepper animates → completes
5. Refresh page mid-pipeline → stepper should restore running state
6. Click "Stop" → pipeline should stop after current step

### Important

- Do NOT modify files in `prisma_review/` or `mcp_server/` — only `api/` and `web/`
- The web/ directory needs `npm install` first (node_modules not in worktree)
- Config file `config.yaml` is gitignored — copy from main: `cp ../prisma_tool/config.yaml .`
- Output dir `prisma_output/` is gitignored — symlink or copy from main for testing
- Commit and push to `feature/session-system` when done

---
> Source: [Black-Lights/prisma-review-tool](https://github.com/Black-Lights/prisma-review-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
