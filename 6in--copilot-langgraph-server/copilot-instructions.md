## copilot-langgraph-server

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Copilot LangGraph Chat**

GitHub Copilot を LangGraph の AI プロバイダーとして使う、社内向け汎用チャット Web アプリ。
`ChatCopilot`（`BaseChatModel` のカスタム実装）を通じて Copilot の推論能力を活用しながら、LangGraph のグラフ構造により将来のエージェント化・ツール呼び出し拡張に対応できる設計を目指す。

> **利用コンテキスト:** 当初は個人用ツールとして設計されたが、現在は **社内プロジェクト向けシステム**として開発を進めている。想定ユーザー規模は **200名程度**。リクエスト量は多くないが、マルチユーザー・マルチアプリケーションの運用に耐える設計（ユーザー分離、アプリケーション管理、監査ログ）を整備中。

**Core Value:** Copilot の JSON-RPC ベース SDK を LangChain 互換プロバイダーとして動かし、アプリケーション（Chat / SuperChat）＋ユーザーという単位でスレッドを管理できるチャット UI から使えること。

### Constraints

- **Tech Stack**: Python（LangChain / LangGraph / Copilot SDK） — ドキュメントのサンプルコードが Python ベース
- **Auth**: Device Flow のみ — 非インタラクティブ環境向け PAT 方式は今回対象外
- **SDK 安定性**: Copilot SDK は Technical Preview — 外部インターフェースを薄いラッパーで隔離しておく
- **スケール感**: 200名規模・社内利用 — 高トラフィック対策より運用性（監査ログ・アプリ管理）を優先する
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Runtime
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Python | 3.12 | Runtime | 3.12 is the stable sweet spot: full support from LangGraph, FastAPI, and github-copilot-sdk (>=3.11). 3.13 is supported but less battle-tested in the ecosystem. | HIGH |
### Core AI Framework
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `langgraph` | 1.1.3 | Stateful conversation graph | The project's core orchestration layer. Provides StateGraph, MessagesState, compiled graph with checkpointer support. Production/Stable (status 5 on PyPI). Python 3.10+ required; 3.13 now officially supported. | HIGH |
| `langchain-core` | 1.2.23 | BaseChatModel base class | Required for the `ChatCopilot` custom provider. Provides `BaseChatModel`, `HumanMessage`, `AIMessage`, `ChatResult`, `ChatGeneration`. Do NOT install the full `langchain` package — `langchain-core` is the slim, stable dependency surface needed. | HIGH |
### Custom Provider
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `github-copilot-sdk` | 0.2.0 | GitHub Copilot JSON-RPC client | The SDK bundles platform-specific Copilot CLI binaries into Python wheels — no separate CLI install needed. Communicates via JSON-RPC (not HTTP/OpenAI-compatible), so `ChatOpenAI(base_url=...)` is not an option. A thin `BaseChatModel` wrapper isolates breaking changes. Technical Preview: pin to an exact version. | LOW (Technical Preview, breaking changes possible) |
### Web Backend
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `fastapi` | 0.135.2 | HTTP + WebSocket API server | FastAPI is the standard choice for async Python APIs in 2025 (78.9k GitHub stars vs Flask's 68.4k; ASGI vs WSGI). Native `async def` routes integrate naturally with LangGraph's `ainvoke`. Built-in Pydantic validation and OpenAPI docs. LangGraph + FastAPI is the most documented integration pattern in 2025. | HIGH |
| `uvicorn` | 0.42.0 | ASGI server | Standard ASGI server for FastAPI. Use `uvicorn[standard]` for production (includes `uvloop` and `httptools`). | HIGH |
### Frontend
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| React 19 + ReactDOM | 19.2 | SPA chat UI | Component-based UI with hooks. Serves the primary chat interface at `/app` via Vite dev server or built static assets. | HIGH |
| TypeScript | 5.9 | Type safety | Full type coverage for components, hooks, and API types. | HIGH |
| Vite | 8.0 | Dev server + bundler | Fast HMR in development; `bun run build` produces `frontend/dist/` served by FastAPI. Vite proxy (`/api` → `localhost:8000`) in dev. | HIGH |
| @chatscope/chat-ui-kit-react | 2.1 | Chat UI components | Ready-made chat UI (MessageList, MessageInput, TypingIndicator). Eliminates custom chat layout work. | HIGH |
| react-markdown + remark-gfm + rehype-highlight | 10.1 / 4.0 / 7.0 | Markdown rendering | Renders AI responses with GFM and syntax highlighting inside chat messages. | HIGH |
| Bun | latest | Package manager (Docker) | Used as the package manager and runtime in the `frontend` Docker service. | MEDIUM |
| Jinja2 | 3.x (FastAPI ships with it) | HTML templating | Used for the legacy Vanilla JS UI served at `/`. Zero additional dependency cost since FastAPI already pulls it in. | MEDIUM |
### Persistence / Session State
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `langgraph-checkpoint-sqlite` | 3.0.3 | Conversation thread persistence | SQLite is the right fit for a single-user local tool: zero operational overhead (no Redis server), file-based durability across process restarts, and `AsyncSqliteSaver` integrates cleanly with async FastAPI. `MemorySaver` is in-memory only (lost on restart) — insufficient for a chat app where history should survive. Redis is explicitly ruled out in PROJECT.md ("個人ツールのため不要"). | HIGH |
### Authentication / Security
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `cryptography` | 46.0.6 | Fernet token encryption | Required by the concept doc's `CopilotAuthManager`. Encrypts the `ghu_` OAuth token at rest in `~/.copilot_sdk/token.enc`. Standard, well-maintained library. | HIGH |
| `httpx` | 0.28.1 | Async HTTP for Device Flow | Required for the GitHub Device Flow OAuth calls in `copilot_auth.py`. The concept doc already uses it. Do not add `requests` (sync) alongside `httpx` (async) — pick one. | HIGH |
### Packaging
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `pyproject.toml` | PEP 621 | Project metadata + dependencies | Standard in 2025. Single source of truth. Works with pip, uv, and all modern build backends. `requirements.txt` is a legacy format — no rationale for using it in a new project. | HIGH |
| `uv` | latest | Dependency management + venv | 10–100x faster than pip. `uv.lock` for reproducible environments. `uv add` / `uv sync` replace `pip install`. Not required by users consuming the app, only for development workflow. | MEDIUM |
## LangGraph API Patterns for This Project
### State Definition
# Option A: extend the built-in (recommended for simple chat)
### Graph Compilation
### Thread-based Sessions
## Alternatives Considered
| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Web framework | FastAPI | Flask | Flask is WSGI; async requires workarounds. LangGraph is natively async — FastAPI is the path of least resistance. |
| Web framework | FastAPI | Django | Too heavy for a personal tool with no ORM needs. |
| Frontend | React 19 + TypeScript + Vite | HTMX | React is the chosen frontend for the primary UI at `/app`. HTMX was considered but React + chatscope provides richer chat UI components. |
| Frontend | React 19 + TypeScript + Vite | Next.js | Next.js adds SSR/routing complexity. Vite SPA is sufficient for a single-user personal tool. Vanilla JS legacy UI remains at `/`. |
| State persistence | SQLite (AsyncSqliteSaver) | MemorySaver | MemorySaver is lost on process restart; not suitable for a persistent chat app. |
| State persistence | SQLite (AsyncSqliteSaver) | Redis | Redis requires a running server. Explicitly out-of-scope in PROJECT.md. |
| Packaging | pyproject.toml | requirements.txt | requirements.txt has no metadata, no build system declaration, and no lock file story. pyproject.toml is the PEP 621 standard. |
| LangChain dependency | langchain-core | langchain (full) | Full `langchain` installs many unused integrations. `langchain-core` provides only what is needed: `BaseChatModel`, message types, output parsers. |
| HTTP client | httpx | requests | Project is fully async; `requests` is synchronous and would block the event loop inside `async def`. |
## Installation
# Using uv (recommended)
# Dev dependencies
# Or using pip
## Critical Constraints
## Sources
- LangGraph PyPI: https://pypi.org/project/langgraph/
- langchain-core PyPI: https://pypi.org/project/langchain-core/
- github-copilot-sdk PyPI: https://pypi.org/project/github-copilot-sdk/
- FastAPI PyPI: https://pypi.org/project/fastapi/
- uvicorn PyPI: https://pypi.org/project/uvicorn/
- langgraph-checkpoint-sqlite PyPI: https://pypi.org/project/langgraph-checkpoint-sqlite/
- cryptography PyPI: https://pypi.org/project/cryptography/
- httpx PyPI: https://pypi.org/project/httpx/
- LangGraph Python 3.13 compatibility announcement: https://changelog.langchain.com/announcements/langgraph-is-now-compatible-with-python-3-13
- FastAPI vs Flask 2025 comparison: https://strapi.io/blog/fastapi-vs-flask-python-framework-comparison
- LangGraph + FastAPI integration guide: https://www.zestminds.com/blog/build-ai-workflows-fastapi-langgraph/
- pyproject.toml packaging guide: https://packaging.python.org/en/latest/guides/writing-pyproject-toml/
- GitHub Copilot SDK overview: https://github.blog/news-insights/company-news/build-an-agent-into-any-app-with-the-github-copilot-sdk/
- GitHub Copilot SDK Python deep-wiki: https://deepwiki.com/github/copilot-sdk/6.2-python-sdk
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

### 応答言語

**すべての応答は日本語で行うこと。** GSD ワークフロー（バナー・チェックポイント・Next Up ブロックなど）を含め、ユーザーへの出力は日本語を使用する。コードやコマンド、ファイルパス、技術的な固有名詞はそのまま英語で記載してよい。

### Merge Workflow

「マージして」と指示された場合、マージを実行する前に必ず次を確認する:

> 「`/create-adr` でこのブランチの振り返りを記録しますか？」

- 了解なら `/create-adr` を実行してから PR 作成 → マージへ進む
- 不要なら即マージへ進む

マージ完了後、不要な worktree を削除する:

```bash
git worktree list
# main 以外の worktree があれば削除
git worktree remove <path>          # 変更なしの場合
git worktree remove --force <path>  # 強制削除が必要な場合
git worktree prune                  # 参照だけ残っているゴミを掃除
```
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

### Backend (Python / FastAPI)

```
app/
  api/
    main.py           — FastAPI app factory, lifespan, CORS, static mounts
    models.py         — Pydantic request/response models
    routes/
      auth.py         — Device Flow OAuth + JWT login/logout
      chat.py         — POST /api/chat (enqueue), GET /api/chat/history
      jobs.py         — GET /api/job/{id}, GET /api/job/{id}/stream (SSE)
      me.py           — GET /api/me (GitHub user info)
  auth/
    jwt_utils.py      — JWT HS256 encode/decode, JTI blocklist
    manager.py        — CopilotAuthManager (Device Flow + token encryption)
  graph/
    builder.py        — LangGraph StateGraph (MessagesState) compilation
  jobs/
    job_store.py      — In-memory job result store with asyncio.Queue SSE
    notifier.py       — SSE notification bridge (worker -> client)
    worker.py         — arq worker: process_chat task
  providers/
    copilot.py        — ChatCopilot (BaseChatModel wrapper for Copilot SDK)
```

### Frontend (React 19 + TypeScript + Vite)

```
frontend/
  src/
    App.tsx            — Root component, AuthContext.Provider
    main.tsx           — ReactDOM entry point
    types.ts           — Shared TypeScript types (Thread, Message, etc.)
    api/
      client.ts        — apiFetch wrapper (JWT cookie auth)
    components/
      AuthPanel.tsx    — Device Flow login UI
      ChatApp.tsx      — Main chat layout (sidebar + messages)
      Header.tsx       — App header with user info
      MarkdownMessage.tsx — Markdown rendering with syntax highlighting
      MessageArea.tsx  — Message list + input (chatscope)
      ThreadSidebar.tsx — Thread list with CRUD
    hooks/
      useAuth.ts       — Auth state management
      useChat.ts       — Chat messaging + SSE job polling
      useThreads.ts    — Thread CRUD operations
```

### Infrastructure

- Docker Compose: FastAPI backend + PostgreSQL (langgraph-checkpoint-postgres) + Redis (arq worker queue) + React frontend (Bun/Vite)
- **Primary startup method: `docker compose up`** — do not use direct `uvicorn` or `bun run dev` commands for running the full app
- **開発時アクセス URL: `http://localhost:5173/orochi/`**（Vite dev server）
- Legacy Vanilla JS UI served at `/` via FastAPI StaticFiles (`static/` directory)
- React UI served at `/app` (Vite dev server proxies `/api` to backend in dev; FastAPI serves `frontend/dist/` in production)
- Reverse-proxy URL prefix (e.g. `/orochi`) configured via `APP_PREFIX` (FastAPI) + `VITE_APP_BASE` (Vite); nginx strips the prefix before forwarding — see `docs/nginx.md`

### Key Patterns

- Async-first: all routes are `async def`, arq worker for background jobs
- SSE for job completion notification (not WebSocket)
- JWT HS256 in httpOnly cookie for auth
- ChatCopilot wraps Copilot SDK behind BaseChatModel interface
- LangGraph StateGraph compiled once at startup; checkpointer lifecycle owned by caller
<!-- GSD:architecture-end -->

## Chrome DevTools MCP

`chrome-devtools` MCP（`.mcp.json`）は `http://127.0.0.1:9222` で動作する Chromium に接続する。

### Chromium の起動

chrome-devtools MCP を使う前に、Chromium がリモートデバッグモードで起動していることを確認する。

**起動コマンド（ユーザーが手動実行）:**
```bash
chromium --remote-debugging-port=9222 --no-first-run --no-default-browser-check &
```

**Claude が使う前に確認すべきこと:**

1. 起動確認:
   ```bash
   curl -s http://127.0.0.1:9222/json/version
   ```
2. 接続できない（エラーまたは空レスポンス）場合は、ユーザーに以下を依頼する:
   ```
   ! chromium --remote-debugging-port=9222 --no-first-run --no-default-browser-check &
   ```
   `!` プレフィックスでセッション内実行できる。
3. 起動確認後、chrome-devtools ツールを使用する。

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.

**ブランチ必須:** `/gsd:quick`・`/gsd:execute-phase`・`/gsd:debug`・`/gsd:do` など GSD コマンドで作業を開始する際は、必ず最初にブランチを作成すること。`main` ブランチ上で直接コミットしない。
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/6in) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
