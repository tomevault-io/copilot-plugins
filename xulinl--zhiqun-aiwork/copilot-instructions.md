## zhiqun-aiwork

> This workspace now contains multiple app modules, including the FastAPI platform backend under `apps/platform-server`. Update this guide whenever a section becomes out of date.

# Repository Guidelines

## Project Status

This workspace now contains multiple app modules, including the FastAPI platform backend under `apps/platform-server`. Update this guide whenever a section becomes out of date.

## Project Structure & Module Organization

Current notable modules:

- `apps/platform-server` - Python 3.12 + FastAPI platform infrastructure service.
- `apps/desktop` - Electron/React desktop client.
- `apps/agent-server` - agent orchestration server.
- `apps/memory-service` - Python + FastAPI memory/knowledge base service (Milvus + MinIO + Redis). Local infra is managed by its own `compose.dev.yml`.
- `docs/` - documentation and design notes.

## Build, Test, and Development Commands

Platform server commands:

- `docker compose -f apps/platform-server/compose.dev.yml up -d` - start local MySQL and Redis (host ports 13306/16379 to avoid clashing with other local stacks).
- `conda activate CoderWorks; cd apps/platform-server; python -m pip install -e ".[dev]"` - install platform backend dependencies in the shared Conda environment.
- `cd apps/platform-server; uvicorn coderworks_platform.main:app --reload --host 0.0.0.0 --port 8080` - run the FastAPI backend.
- `cd apps/platform-server; python -m pytest` - run platform backend tests.
- `cd apps/platform-server; python -m ruff check .` - run platform backend lint checks.

Memory service commands:

- `docker compose -f apps/memory-service/compose.dev.yml up -d` - start local Milvus (etcd + MinIO + standalone, ports 2379/9000/9001/19530/9091) and Redis (6379). Port 6379 must be free: a native Windows Redis service (`C:\Program Files\Redis`) also binds it and should be stopped/disabled (`Stop-Service Redis` in an elevated shell) so the repo-managed container owns it.
- `cd apps/memory-service; uvicorn memory_service.main:app --reload --host 0.0.0.0 --port 8430` - run the memory service (env prefix `CODERWORKS_MEMORY_`, settings in `src/memory_service/config.py`).

Desktop dev commands:

- Containers are started manually via the per-app compose commands above; the backend services are pulled up automatically.
- `cd apps/desktop; npm run electron:dev` - one-shot dev startup: `scripts/start-backends.cjs` idempotently starts four backend services (auth-server 44130, agent-server 44120, platform-server 8080, memory-service 8430) using the CoderWorks conda Python for the Python services (override with `CODERWORKS_PYTHON`), then launches vite + Electron. Missing containers are warned about, not auto-started.

## Coding Style & Naming Conventions

- Use spaces for indentation, 2 per level, unless the chosen language dictates otherwise.
- Name files, functions, and variables descriptively; prefer `lower-kebab-case` for filenames and `camelCase` for code identifiers.
- Keep line length near 100 characters.
- Run the project's linter or formatter on every change once one is adopted.

## Testing Guidelines

The platform backend uses pytest and httpx. Keep tests under `apps/platform-server/tests`, with names like `test_*.py`.

## Commit & Pull Request Guidelines

There is no Git history to derive conventions from yet. Recommended practice:

- Write commit messages in the imperative mood, prefixed with a type: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, or `chore:`.
- Keep the subject line under 72 characters; add a body for context when needed.
- Open pull requests with a clear description, a link to any related issue, and screenshots or output for UI or behavioral changes.

## Agent-Specific Instructions

- For frontend UI, always check shadcn/ui first. If shadcn provides the component, use the local wrapper under `apps/desktop/src/components/ui/` instead of rebuilding it with raw HTML or a custom base component. Extend behavior through standard variants, sizes, class names, and design tokens; create a custom component only when shadcn has no suitable capability, and document why reuse is not possible.
- For hover help text, always use the local Tooltip component under `apps/desktop/src/components/ui/tooltip.tsx`; do not use native `title` attributes for user-facing hover hints. Tooltip content should rely on the component's default top placement unless a specific layout need requires another side.
- All enabled interactive controls must show a pointer cursor, including shadcn/Base UI triggers, options, menu items, tabs, checkboxes, and switches that are not native buttons. Disabled controls must use the not-allowed cursor. Enforce this globally through semantic roles and `data-slot`, not with page-specific patches.
- Keep shadcn/Base UI controls compact globally: Button, Input, and Select Trigger use `rounded-md` with 12px text; standard text Buttons use 12px horizontal padding while icon-only Buttons are excluded. Select Content uses `rounded-md`, and Select Item uses `rounded-sm` with 12px text. Apply this through global `data-slot` rules; do not enlarge individual pages to `rounded-lg` or 14px except for an explicitly documented primary input surface, round icon button, or segmented capsule.
- Chat-adjacent overflowing content areas and task panels must use the same immersive scrollbar as the conversation area: `custom-scroll` on the scroll container with `<ScrollbarOverlay />` as its first child. Keep scrollbar colors on `--scrollbar-thumb` / `--scrollbar-thumb-hover`; do not introduce page-specific native scrollbar styling for these panels.
- 大模型对话功能必须全链路同步修改：任何涉及对话文案、运行状态、深度思考、Token/上下文统计、工具调用、消息操作区、流式事件或错误处理的改动，都必须同时检查普通聊天链路和专家团链路。普通聊天重点检查 `ChatPage` / `MessageList`，专家团重点检查 `ExpertTeamView` / `MemberHubView` / `MemberThreadView` / `ExpertMessageCard` 及其对应的 orchestration 数据流；不能只改一个入口后视为完成。若两条链路的数据来源不同，必须明确适配层和缺失数据的展示规则，禁止用占位统计冒充真实数据。完成前至少运行桌面端 TypeScript 检查，并在最终说明已覆盖的入口和仍待补齐的后端字段。

This workspace has the following MCP servers enabled for agent-assisted development:

- **playwright** - browser automation (navigate, click, screenshot)
- **github** - GitHub API access (issues, PRs, repos, search)

Keep `CLAUDE.md` and `AGENTS.md` in sync; when project structure or commands change, update both.

### Electron compilation

- `cd apps/desktop; npm run electron:compile` regenerates `electron/main.js`, `preload.js`, and `auth-store.js` from TypeScript. Desktop type checking alone does not emit these files.
- `scripts/start-electron.cjs` compiles before launching Electron; `npm run electron:build` compiles before packaging. Main/preload changes require restarting Electron after compilation.

---
> Source: [xulinl/zhiqun-aiwork](https://github.com/xulinl/zhiqun-aiwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
