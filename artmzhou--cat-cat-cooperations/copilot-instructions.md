## cat-cat-cooperations

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cat Agent Platform (猫猫多Agent协同系统) is a CLI Agent collaboration platform supporting external CLI tools (Claude Code, OpenCode, etc.) with task orchestration and real-time communication.

## Build & Development Commands

### Quick Start

```bash
# Build and run (no external dependencies)
mvn clean package -pl cat-standalone -am -DskipTests
java -jar cat-standalone/target/cat-standalone-1.0.0-SNAPSHOT.jar

# Or run directly with Maven
mvn spring-boot:run -pl cat-standalone

# Or use the startup scripts
run-standalone.bat  # Windows
./run-standalone.sh # Linux/Mac
```

**Features:**
- JSON file storage (no database required)
- Embedded Redis server (optional, for caching)
- Simplified authentication (any username/password works)
- Single process, easy local development

**Access:**
- Frontend: http://localhost:3000
- API: http://localhost:8080/api/v1
- Data Directory: ./data/

### Backend (Java/Spring Boot)

```bash
# Build
mvn clean package -pl cat-standalone -am -DskipTests

# Run tests
mvn test -pl cat-standalone

# Run development server
mvn spring-boot:run -pl cat-standalone
```

### Frontend (Vue 3/TypeScript)

```bash
cd cat-web

# Install dependencies
npm install

# Development server (port 3000)
npm run dev

# Build for production
npm run build

# Lint code
npm run lint
```

## Service Ports

| Service | Port | Description |
|---------|------|-------------|
| cat-standalone | 8080 | Backend API |
| cat-web | 3000 | Frontend dev server |
| Embedded Redis | 6380 | Optional caching |

## Architecture

### Module Structure

```
cat-cat-cooperations/
├── cat-standalone/    # Standalone module - all backend code
│   ├── src/main/java/com/cat/standalone/    # Core services
│   ├── src/main/java/com/cat/cliagent/      # CLI Agent services
│   └── data/                                # JSON data files
└── cat-web/           # Vue 3 frontend SPA
    └── src/
        ├── api/       # API client modules
        ├── views/     # Page components
        └── utils/     # Utilities (WebSocket, etc.)
```

### Backend Layer Architecture

- Controller → Service → JsonFileStore
- Entity classes in `store/entity/`, DTOs in `dto/`
- Global exception handling via `GlobalExceptionHandler`
- Standardized API responses via `ApiResponse<T>`

### Frontend Structure

- Vue 3 Composition API with `<script setup>`
- API modules in `src/api/`
- Pinia stores in `src/stores/`
- WebSocket via `@stomp/stompjs`

### CLI Agent Architecture

**Execution Mode:** Per-request (`--print` mode)
- New process spawned for each request
- Uses `--resume <sessionId>` for conversation continuity
- Output parsed from `stream-json` format

**Startup Behavior:**
- On backend restart, all RUNNING/EXECUTING agents are automatically reset to STOPPED
- This prevents stale status display since process context is lost on restart

**Key Services:**
- `LocalCliAgentService` - Agent CRUD operations
- `LocalCliSessionService` - Session & process management
- `LocalCliProcessService` - Process lifecycle
- `LocalCliTaskExecutionService` - Task execution
- `LocalTokenUsageService` - Token usage tracking
- `LocalCliOutputPushService` - WebSocket output push

### Data Storage

**JSON Files (./data/ directory):**
- `cli_agents.json` - CLI Agent instances
- `cli_agent_templates.json` - CLI Agent templates
- `cli_agent_capabilities.json` - Agent capabilities
- `token_usage_logs.json` - Token usage records
- `cli_agent_output_logs.json` - CLI Agent output logs (每Agent最多100条)
- `tasks.json` - Task entities (backend only, no frontend UI)
- `task_assignments.json` - Task assignments (backend only)
- `task_logs.json` - Task execution logs (backend only)

**Format:** Jackson with Java 8 time support

## Tech Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | Vue 3.4+, TypeScript 5.0+, Vite 5.0+, Element Plus 2.5+ | |
| Backend | Spring Boot 3.2+, Java 17 | |
| Storage | JSON Files | ./data/ directory |
| Cache | Embedded Redis | Port 6380, optional |
| WebSocket | STOMP over SockJS | Real-time output |

## Key API Endpoints

**CLI Agent Management:**
- `GET/POST /api/v1/cli-agents` - Agent list/create
- `GET/PUT/DELETE /api/v1/cli-agents/{id}` - Agent operations
- `POST /api/v1/cli-agents/{id}/actions/start` - Start agent
- `POST /api/v1/cli-agents/{id}/actions/stop` - Stop agent
- `POST /api/v1/cli-agents/{id}/actions/restart` - Restart agent

**CLI Agent Communication:**
- `POST /api/v1/cli-agents/{id}/session/input` - Send input
- `GET /api/v1/cli-agents/{id}/logs` - Get output logs
- WebSocket `/ws` - Real-time output streaming

**CLI Agent Monitoring:**
- `GET /api/v1/cli-agents/monitor/overview` - System overview (used by dashboard)

## Development Workflow

This project uses long-task methodology:
- Feature progress tracked in `feature-list.json`
- See `long-task-guide.md` for workflow details

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtmZhou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
