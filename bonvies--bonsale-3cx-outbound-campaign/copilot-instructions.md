## bonsale-3cx-outbound-campaign

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Commands

### Development
```bash
pnpm install              # Install all dependencies
pnpm dev                  # Run backend & frontend in dev mode
pnpm backend              # Run backend only (Node.js on port 4020)
pnpm frontend             # Run frontend only (Vite on port 4030)
pnpm type-check           # TypeScript validation across all packages
pnpm lint                 # ESLint check
pnpm lint:fix             # ESLint with auto-fix
```

### Docker (Local Development)
```bash
pnpm docker:up            # Start docker-compose (backend + frontend + redis)
pnpm docker:down          # Stop docker-compose
docker-compose logs -f    # View live logs
```

### Build & Test
```bash
pnpm build                # Build all packages (Turbo orchestrated)
pnpm test                 # Run tests
pnpm clean                # Clean build artifacts
```

### Single Package Commands
```bash
# Backend (apps/backend/)
pnpm -F backend dev       # Backend dev server with auto-reload
pnpm -F backend build     # Compile TypeScript to dist/

# Frontend (apps/frontend/)
pnpm -F frontend dev      # Frontend dev with HMR
pnpm -F frontend build    # Production build
```

## Project Architecture

### Technology Stack
- **Frontend**: React 19 + Vite + TypeScript + Material-UI 7
- **Backend**: Node.js 18+ + Express + TypeScript + WebSocket (ws)
- **Data**: Redis 7 (state & cache) + async-mutex (concurrency)
- **Build**: Turbo monorepo + pnpm workspaces
- **Deployment**: Docker + Docker Compose + GCP Container Registry

### Monorepo Structure
```
apps/backend/           # Express API server
  - src/class/         # Core business logic (Project, ProjectManager, etc.)
  - src/services/      # Redis, API integrations (3CX, Bonsale, xApi)
  - src/routes/        # HTTP endpoints
  - src/websockets/    # WebSocket connection handlers
  - src/shared/        # Backend-only shared utilities (alias: @shared-local/*)
  - src/features/
    - outbound-campaign/   # 3CX 自動外播功能
    - call-schedule/       # 自動語音通知功能
      - services/api/      # Phone API 抽象層（IPhoneApiService）
        - device/          # NewRock / Yeastar 實作
      - services/monitor/  # Call Monitor 抽象層（ICallMonitorService）
        - device/          # NewRock / Yeastar 監控實作
      - services/          # callScheduleService, database
      - routes/            # HTTP 路由
      - types/             # 型別定義

apps/frontend/         # React web dashboard
  - src/components/    # React UI components
  - src/pages/         # Page-level components
  - src/router/        # React Router setup
  - src/theme/         # Material-UI theme

packages/shared-types/ # Shared TypeScript definitions
  - ApiResponse<T>, User, Auth types
  - WebSocket event types
```

### High-Level Architecture

This is a **real-time outbound campaign management system** integrating:
1. **3CX Phone System**: WebSocket connection for live call monitoring/control
2. **Bonsale CRM**: API integration for contact management and configurations
3. **Campaign Dashboard**: Real-time UI for viewing/controlling active campaigns

**Key Pattern**: The `Project` class (src/class/project.ts) is the heart of the system, managing a single campaign's full lifecycle:
- 3CX WebSocket connection (connect → authenticate → listen for calls)
- Call list processing and dialing
- Call state tracking (ringing → connected → completed)
- Bonsale CRM updates on call completion
- Automatic recovery on connection loss

The backend exposes this via WebSocket events, allowing the frontend to view/control projects in real-time.

### State Management
- **In-Memory**: Active Project instances maintain state during operation
- **Redis**: Persistent storage of project state (survives restarts with `AUTO_RECOVER_ON_RESTART=true`)
- **Bonsale**: CRM source of truth for contacts and call outcomes

## Core Components to Know

### Backend

**`Project` class** (`src/class/project.ts`, ~95KB)
- Manages a single outbound campaign lifecycle
- Connects to 3CX, monitors calls, dials contacts, updates CRM
- Complex state machine with concurrency control (mutex)
- Implements: token refresh, call restrictions (time/extension), idle checks, auto-recovery

**`ProjectManager`** (`src/class/projectManager.ts`)
- Registry of active Project instances
- Lifecycle operations: create, stop, remove, recover from Redis

**`WebSocketManager`** (`src/class/webSocketManager.ts`)
- Manages WebSocket server connections and message broadcasting
- Two endpoints: main WS server + Bonsale webhook listener

**`redis.ts`** (`src/services/redis.ts`)
- Redis client initialization and cache operations
- Persistence layer for project state

**API Services** (`src/services/api/`)
- `bonsale.ts`: Contact retrieval, campaign config
- `callControl.ts`: 3CX call operations (dial, hangup, transfer)
- `xApi.ts`: Token refresh from 3CX authentication

**Call Schedule Feature** (`src/features/call-schedule/`)
- **`phoneApiService.ts`**: `IPhoneApiService` 抽象層，根據 `TELEPHONE_EQUIPMENT` 環境變數切換設備
  - `device/newRockApi.ts`: NewRock HTTP 1.0 XML API 實作
  - `device/yeastarApi.ts`: Yeastar REST API + 自動 Token 刷新（斷網恢復）
- **`callMonitorService.ts`**: `ICallMonitorService` 抽象層，切換通話監控實作
  - `monitor/device/NewRockCallMonitorService.ts`: HTTP Server 接收 NewRock XML Push（port 4022）
  - `monitor/device/YeastarCallMonitorService.ts`: WebSocket 訂閱 Yeastar CDR 事件（topic 30012）
  - `monitor/callMonitorCore.ts`: 共用業務邏輯（pendingCalls、retry、handleAnswer/Bye）
- **`callScheduleService.ts`**: 排程核心（node-schedule），讀 SQLite 排程並觸發撥號
- **`database.ts`**: better-sqlite3，SQLite 資料庫操作

**切換設備**：只需修改 `.env` 中的 `TELEPHONE_EQUIPMENT=NewRock|Yeastar`，程式碼層不需改動。

### Frontend
- **Router setup** determines page layout
- **Real-time updates** via WebSocket message events
- **Material-UI theme** in `src/theme/` for consistent styling

## Important Patterns & Conventions

### Error Handling
**Error Serialization Issue**: When Error objects are serialized to JSON, they lose properties (stack, name become empty).
- Solution: Check `/docs/ERROR_HANDLING.md` for proper error handling
- Use `error instanceof Error` checks and extract properties before serialization
- Always wrap token refresh failures with proper error context

### WebSocket Protocol
- **Format**: `{ event: string, payload: any }`
- **Server responses**: Include `timestamp` in payload
- **Event examples**: `startOutbound`, `stopOutbound`, `allProjects`
- See `/docs/WEBSOCKET_FORMAT.md` for complete specification

### Concurrency Control
- `async-mutex` protects critical sections in Project class
- Example: Token refresh and call state updates are mutex-protected
- **Important**: Ensure all async operations within mutex blocks complete properly (avoid deadlocks)

### Call Restrictions
- **Time-based**: Campaign only dials during specified UTC time windows
- **Extension cooldown**: Extensions have min interval between consecutive calls
- **Cross-day ranges**: Restrictions can span midnight (e.g., 22:00-06:00)

### TypeScript Paths
- `@/*` → Backend: `apps/backend/src/*`
- `@shared/*` → Backend: `../../packages/shared-types/src/*`
- `@shared-local/*` → Backend: `apps/backend/src/shared/*`
- `@outbound/*` → Backend: `apps/backend/src/features/outbound-campaign/*`
- `@call-schedule/*` → Backend: `apps/backend/src/features/call-schedule/*`
- Frontend imports from `packages/shared-types` directly

**Production Build Note**: Path aliases are resolved at build time by `tsc-alias` (runs after `tsc`). The `build` script is `tsc && tsc-alias` — never just `tsc` alone, or aliases will remain unresolved in `dist/` and cause `Cannot find module` errors at runtime.

## Environment Configuration

Create `.env` at root with:
```
# 3CX Integration
HTTP_HOST_3CX=https://your-3cx-host:5015
WS_HOST_3CX=wss://your-3cx-host:5015
ADMIN_3CX_CLIENT_ID=xxx
ADMIN_3CX_CLIENT_SECRET=xxx

# Application
HTTP_PORT=4020
NODE_ENV=development

# Bonsale CRM
BONSALE_HOST=https://your-bonsale-host
BONSALE_X_API_KEY=xxx
BONSALE_X_API_SECRET=xxx

# Redis
REDIS_URL=redis://localhost:6379

# Features
AUTO_RECOVER_ON_RESTART=true
IS_STARTIDLECHECK=true

# Idle Check Intervals (ms)
IDLE_CHECK_INTERVAL=300000
MIN_IDLE_CHECK_INTERVAL=30000
MAX_IDLE_CHECK_INTERVAL=300000
IDLE_CHECK_BACKOFF_FACTOR=1.5

# Feature Flags
ENABLE_OUTBOUND_CAMPAIGN=true   # 自動外播功能
ENABLE_CALL_SCHEDULE=true        # 自動語音通知 (Call Schedule)

# ── Call Schedule 專屬（ENABLE_CALL_SCHEDULE=true 時必填）──
# 電話設備選擇（必填，無預設值）
TELEPHONE_EQUIPMENT=NewRock      # NewRock 或 Yeastar

# OM 事件監聽 Port（接收設備 Push 通知）
OM_MONITOR_PORT=4022

# 外播主叫分機號碼
OM_CALL_FROM_EXTENSION=9038

# NewRock 設定（TELEPHONE_EQUIPMENT=NewRock 時必填）
NEW_ROCK_API_HOST=http://your-newrock-host
NEW_ROCK_API_PATH=/xml

# Yeastar 設定（TELEPHONE_EQUIPMENT=Yeastar 時必填）
YEASTAR_API_HOST=https://your-yeastar-host
YEASTAR_API_PATH=/openapi/v1.0
YEASTAR_USERNAME=your-username
YEASTAR_PASSWORD=your-password
```

## Development Workflow

1. **Start development environment**: `pnpm docker:up` (backend + frontend + redis)
2. **Or run individually**: `pnpm backend` + `pnpm frontend` in separate terminals
3. **Watch changes**: Turbo caching auto-rebuilds dependencies
4. **Check types**: `pnpm type-check` catches TypeScript errors early
5. **Lint before commit**: Git hooks run `lint-staged` via Husky

### Debugging Tips
- **Backend logs**: Check `docker-compose logs backend` for real-time logs
- **Redis state**: Use `redis-commander` (port 8081) to inspect Redis data
- **Frontend console**: Browser DevTools > Console for client-side errors
- **WebSocket events**: Monitor network tab for WebSocket frames

## Testing

- Backend uses Jest (minimal test setup currently)
- No dedicated frontend test suite yet
- Focus on type safety (`pnpm type-check`) as primary quality gate

## Deployment

### Local Docker
```bash
docker-compose up --build
```

### Production (GCP)
1. Build and push images to GCP Container Registry:
   ```bash
   docker build --platform linux/amd64 -f apps/backend/Dockerfile \
     -t gcr.io/PROJECT/bonsale_3cx-outbound-campaign_backend:latest .
   docker push gcr.io/PROJECT/bonsale_3cx-outbound-campaign_backend:latest
   ```
2. Deploy via GCP Cloud Run or Kubernetes using `docker-compose.prod.yml` as reference

See `/README.md` for comprehensive deployment guide (in Chinese).

## Key Files Reference

| File | Purpose |
|------|---------|
| `apps/backend/src/app.ts` | Express app setup, route mounting, WebSocket initialization |
| `apps/backend/src/class/project.ts` | Campaign lifecycle, 3CX integration, call processing |
| `apps/backend/src/class/projectManager.ts` | Project registry and lifecycle |
| `apps/backend/src/services/redis.ts` | Redis client, state persistence |
| `apps/frontend/src/main.tsx` | React entry point |
| `packages/shared-types/src/index.ts` | Shared types and interfaces |
| `docs/WEBSOCKET_FORMAT.md` | WebSocket protocol specification |
| `docs/ERROR_HANDLING.md` | Error serialization patterns |
| `apps/backend/src/features/call-schedule/services/api/phoneApiService.ts` | IPhoneApiService 抽象層，設備切換點 |
| `apps/backend/src/features/call-schedule/services/callMonitorService.ts` | ICallMonitorService 抽象層，監控切換點 |
| `apps/backend/src/features/call-schedule/services/monitor/callMonitorCore.ts` | 通話監控共用邏輯（retry、pendingCalls） |

## Notes for Future Development

- **Project class is complex**: Consider breaking into smaller focused classes as it grows
- **Error handling**: Always check `/docs/ERROR_HANDLING.md` before serializing Errors
- **Token refresh**: Mutex deadlock prevention is critical (see TokenManager)
- **Scaling**: Redis persistence enables horizontal scaling of campaign instances
- **Type safety**: Leverage shared-types to ensure frontend/backend consistency
- **Adding a new phone device**: Implement `IPhoneApiService` in `services/api/device/`, implement `ICallMonitorService` in `services/monitor/device/`, then add both to the switch in `phoneApiService.ts` and `callMonitorService.ts`
- **Call Schedule DB**: Uses SQLite (`better-sqlite3`), not Redis — schema lives in `database.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bonvies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
