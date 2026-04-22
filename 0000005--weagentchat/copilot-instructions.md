## weagentchat

> **WeAgentChat (唯信)** is an AI-native social sandbox application — **The first social platform where YOU are the only human center.**

# WeAgentChat (唯信) Project Context

## Project Overview
**WeAgentChat (唯信)** is an AI-native social sandbox application — **The first social platform where YOU are the only human center.** 

Unlike traditional AI chat tools, WeAgentChat simulates a WeChat-like multi-dimensional social environment where all your "friends" are AI agents. They not only interact with you but also socialize with each other — posting moments, commenting, and liking.

### Core Innovations

1. **Dual-Track Long-Term Memory System**
   - **Global Profile:** AI agents automatically update your personality, preferences, and life situation based on conversations.
   - **Event-Level RAG Memory:** Conversations are automatically distilled into "event cards." Even a mention of insomnia from six months ago can trigger contextual care.

2. **Passive Session Management**
   - Say goodbye to the "New Chat" button. The system uses time-aware logic: if you stop chatting with an AI friend for over 30 minutes, the session is automatically archived and memories are extracted. When you speak again, it's a natural, context-aware new beginning.

3. **AI Moments Ecosystem**
   - AI agents post updates, comment on each other, and interact autonomously.

## Quick Start

### One-click Start (Development)
Run in project root:
`scripts\startAll.bat`
This script will simultaneously launch the Backend API and Frontend Dev Server.

### Frontend
1.  Navigate to `front/`: `cd front`
2.  Install dependencies: `pnpm install`
3.  Run dev server: `pnpm dev`
4.  Access: `http://localhost:5173`

### Backend
1.  Navigate to `server/`: `cd server`
2.  Run server: `venv\Scripts\python -m uvicorn app.main:app --reload`
3.  Access docs: `http://localhost:8000/docs`

### Desktop (Electron)
1.  Start backend and frontend servers first (see above).
2.  Run Electron dev mode at root: `pnpm electron:dev`
3.  The app will load from `http://localhost:5173` with backend at `http://localhost:8000`.

### Production Build (Desktop)
1.  Run one-click packaging script: `scripts\package-electron.bat`
2.  Output will be in `dist-electron/`
3.  Or step-by-step:
    *   Build backend with PyInstaller (output to `build/backend`)
    *   Run `pnpm electron:build` at project root

## Tech Stack
### Frontend (`front/`)
*   **Framework:** Vue 3.5+ (Composition API)
*   **Build Tool:** Vite 6
*   **Language:** TypeScript 5
*   **Styling:** Tailwind CSS 3.4
*   **UI Components:**
    *   shadcn-vue (Radix Vue based)
    *   ai-elements-vue (AI-native components)
    *   Lucide Vue Next (Icons)
*   **State Management:** Pinia
*   **Routing:** Vue Router
*   **AI Integration:** Vercel AI SDK (`ai` package)
*   **Markdown & Highlighting:** streamdown-vue, shiki
*   **Animations:** motion-v
*   **Diagrams:** @vue-flow

### Backend (`server/`)
*   **Language:** Python 3.11+
*   **Framework:** FastAPI
*   **Agent Framework:** [OpenAI Agents](https://github.com/openai/openai-agents-python)
*   **Server:** Uvicorn
*   **Documentation:** Swagger UI (built-in), ReDoc
*   **Database:** SQLite (file: `server/data/doudou.db`) + SQLAlchemy + sqlite-vec (for vector search)
*   **Data Validation:** Pydantic v2
*   **Utilities:** python-multipart (for form data)
*   **Structure:** Layered Architecture (API -> Service -> Models/Schemas)
*   **API Prefix:** `/api`

#### Provider 适配规则
为降低多模型接入的维护成本，provider 级差异被集中到一个轻量规则模块：
*   **规则集中位置：** `server/app/services/provider_rules.py`
*   **职责范围（provider 级）：**
    *   base_url / model_name 归一化（如 Gemini）
    *   是否走 LiteLLM
    *   是否支持 `reasoning_effort` 这类协议级参数
    *   Gemini `thought_signature`、DeepSeek `reasoning` 注入等特殊要求
*   **模型能力来源（model 级）：** 仍由数据库中的模型配置决定（工具调用/思考/视觉/联网等）



### Desktop (`electron/`)
*   **Framework:** Electron 32+
*   **Builder:** electron-builder 24
*   **Process Management:** tree-kill (for graceful backend shutdown)
*   **Backend Packaging:** PyInstaller (onedir mode)
*   **Architecture:**
    *   Main process spawns Python backend as child process
    *   Preload script injects backend port via `contextBridge`
    *   Splash screen during backend initialization
    *   Health check polling before showing main window

### About ai-elements-vue

[ai-elements-vue](https://www.ai-elements-vue.com/) is a component library built on top of [shadcn-vue](https://www.shadcn-vue.com/), specifically designed for building AI-native applications. It provides pre-built, customizable components including:

- **Chat Components**: `conversation`, `message`, `prompt-input`,`more...`
- **Reasoning Display**: `chain-of-thought`, `reasoning`,`more...`
- **Tool Visualization**: `tool`, `confirmation`,`more...`
- **Workflow**: `canvas`, `node`, `edge`,`more...`
- **Utilities**: `code-block`, `loader`, `suggestion`,`more...`
- **More**: check `front/src/components` folder, find more components and usage.
**GitHub**: [vuepont/ai-elements-vue](https://github.com/vuepont/ai-elements-vue)

**使用文档**: 当需要使用 ai-elements-vue 组件时，必须先调用 `context7` 查询组件的使用方法，然后按照返回的使用方法进行实现。


## Current Status & Structure
The project is currently in the **active development phase**.

*   **Root Directory:** `e:\workspace\code\DouDouChat`

---

### 🎨 Frontend (`front/`)
Vue 3 frontend implemented with a focus on WeChat's aesthetic.

#### 📁 `src/` (Core Source)
*   **`components/`**: UI logic and views.
    *   `ai-elements/`: AI-native components (Reasoning, Tool, Canvas, etc.) from `ai-elements-vue`.
    *   `ui/`: Base UI primitives (via shadcn-vue, e.g., HoverCard, Dialog, Button).
    *   `common/`: Common reusable components (e.g., `AvatarUploader.vue`, `ToolCallsDetail.vue`).
    *   `ChatArea.vue`: Main message terminal (supports SSE events & reasoning).
    *   `GroupChatArea.vue`: Group chat main area (supports SSE events & 接力讨论/原自驱).
        *   `GroupChatArea.css`: Scoped styles for `GroupChatArea.vue`.
    *   `ChatDrawerMenu.vue`: WeChat-style drawer for chat settings and actions.
    *   `GroupChatDrawer.vue`: Group chat drawer (members, info, actions).
    *   `Sidebar.vue`: Session list and search.
    *   `IconSidebar.vue`: Vertical icon menu (WeChat style).
    *   `SettingsDialog.vue`: Management of LLM, Memory, and System settings.
    *   `ProfileDialog.vue`: User profile management.
    *   `SetupWizard.vue`: First-time configuration onboarding.
    *   `AssistantWizard.vue`: AI-guided friend creation wizard.
    *   `FriendGallery.vue`: Friend library (preset personas gallery).
    *   `FriendComposeDialog.vue`: Friend edit/create dialog.
    *   `GroupComposeDialog.vue`: Group create/edit dialog.
    *   `EmojiPicker.vue`: WeChat-style emoji selection.
    *   `AboutDialog.vue`: About dialog with version info.
    *   `UpdateNotifyDialog.vue`: Update notification dialog.
    *   `WindowControls.vue`: Custom window controls (minimize/maximize/close).
    *   `ToastContainer.vue`: Toast notification container.
    *   `GroupAutoDriveConfigDialog.vue`: 接力讨论（原自驱）配置对话框。
*   **`stores/`**: Pinia state management.
    *   `session.ts`: 会话状态入口与编排层（仅 state/computed + 组合 action）。
    *   `session.fetch.ts`: 拉取/分页/同步相关逻辑。
    *   `session.sessions.ts`: 会话管理相关逻辑（切换/删除/新会话）。
    *   `session.stream.friend.ts`: 单聊 SSE 流式处理（发送/重生成/撤回）。
    *   `session.stream.group.ts`: 群聊 SSE 流式处理与 typing 状态管理。
    *   `session.stream.group.auto_drive.ts`: 接力讨论 SSE 流式状态与控制逻辑。
    *   `friend.ts`: Persona/Friend metadata and state.
    *   `group.ts`: Group Chat metadata and state.
    *   `llm.ts` & `embedding.ts`: Global config synchronization with backend.
    *   `settings.ts`: System-wide settings (e.g., memory expiration).
    *   `memory.ts`: Long-term memory and recall state.
    *   `thinkingMode.ts`: Global setting for LLM reasoning display.
*   **`api/`**: Strongly typed REST & SSE clients.
    *   `base.ts`: Base API configuration and Electron port handling.
    *   `chat.ts`, `friend.ts`, `friend-template.ts`, `group.ts`, `llm.ts`, `embedding.ts`, `settings.ts`, `memory.ts`, `health.ts`, `upload.ts`.
*   **`composables/`**: Reusable Vue Composition API logic.
    *   `useChat.ts`: Chat interaction logic.
    *   `useToast.ts`: Toast notification management.
    *   `useUpdateCheck.ts`: Version update checking.
*   **`types/`**: Frontend shared types.
    *   `chat.ts`: Message/ToolCall/GroupTypingUser 类型定义。
*   **`utils/`**: Shared utilities.
    *   `chat.ts`: `parseMessageSegments` 与 `INITIAL_MESSAGE_LIMIT`。
*   **`lib/`**: Utility functions (e.g., `utils.ts` for Tailwind/CSS classes).

#### 接力讨论（原自驱）相关文件
Frontend:
- `front/src/components/GroupChatArea.vue`: 入口与状态展示（接力讨论工具栏、状态栏、控制按钮）。
- `front/src/components/GroupAutoDriveConfigDialog.vue`: 接力讨论配置面板（模式、参与成员、流程）。
- `front/src/components/GroupChatArea.css`: 接力讨论状态条与按钮样式。
- `front/src/api/group-auto-drive.ts`: 接力讨论接口客户端（start/pause/resume/stop/state/stream/interject）。
- `front/src/stores/session.stream.group.auto_drive.ts`: 接力讨论 SSE 流式状态与控制逻辑。
- `front/src/types/chat.ts`: 接力讨论相关类型（`AutoDriveMode`/`AutoDriveState`/`AutoDriveConfig`）。
注：接口路径与内部命名仍沿用 `auto-drive`/`auto_drive`。

#### 📁 Configuration
*   `vite.config.js`, `tailwind.config.js`, `components.json` (shadcn config).

---

### ⚙️ Backend (`server/`)
FastAPI backend with a modular service-oriented architecture.

#### 📁 `app/` (Application Logic)
*   **`api/endpoints/`**: FastAPI routers.
    *   `chat.py`: Real-time SSE streaming.
    *   `profile.py` & `friend.py`: User profile and AI persona management.
    *   `friend_template.py`: Preset friend templates API.
    *   `group_chat.py`: Group chat messaging and management API.
    *   `group_auto_drive.py`: 群聊接力讨论（原自驱）API（start/stream/pause/resume/stop/state）。
    *   `upload.py`: File upload API (avatars, etc.).
    *   `settings.py`: System configuration API.
    *   `llm.py` & `embedding.py`: AI model provider management.
    *   `health.py`: Health check and onboarding status.
*   **`services/`**: Business logic layer.
    *   `chat_service.py`: LLM orchestration, message persistence, and memory RAG.
    *   `recall_service.py`: Multi-step memory recall and agent orchestration.
    *   `group_chat_service.py`: Group chat core logic.
    *   `group_service.py`: Group metadata management.
    *   `group_auto_drive_service.py`: 群聊接力讨论（原自驱）编排与状态流转。
    *   `group_chat_shared.py`: Shared utilities for group chat.
    *   `llm_service.py` & `embedding_service.py`: Model provider abstraction.
    *   `friend_service.py`: Persona and friendship management.
    *   `friend_template_service.py`: Preset friend template management.
    *   `persona_generator_service.py`: AI-powered persona generation.
    *   `provider_rules.py`: Lightweight rule module for provider-level differences (base_url normalization, reasoning injection, etc.).
    *   `reasoning_stream.py`: Reasoning stream handling.
    *   `memo/`: Memory system bridge.
        *   `bridge.py`: Interface to the embedded Memobase SDK.
        *   `constants.py`: Memory system constants and configuration.
        *   `default_profile_config.py`: Default configuration for user profile memory extraction.
    *   `settings_service.py`: Config defaults and DB persistence.
*   **`models/`**: SQLAlchemy ORM definitions (SQLite target).
    *   `chat.py`, `friend.py`, `friend_template.py`, `group.py`, `system_setting.py`, `llm.py`, `embedding.py`.
*   **`schemas/`**: Pydantic data validation and serialization.
    *   `chat.py`, `friend.py`, `friend_template.py`, `group.py`, `group_auto_drive.py`, `llm.py`, `embedding.py`, `memory.py`, `sse_events.py`, `system_setting.py`, `persona_generator.py`.
*   **`db/`**: Database initialization and session management.
    *   `init_db.py`: Database initialization logic.
    *   `init.sql`: Core schema initialization.
    *   `init_persona_templates.sql`: Preset friend templates data.
    *   `session.py`: Database session management.
    *   `base.py`: SQLAlchemy base configuration.
    *   `types.py`: Custom database types.
*   **`core/`**: Core system configuration and `logging.py`.
*   **`vendor/`**: Third-party modules embedded as SDKs.
    *   **`memobase_server/`**: The core Memory Engine (Event Extraction, RAG).

#### 接力讨论（原自驱）相关文件
Backend:
- `server/app/api/api.py`: 接力讨论路由挂载（`group_auto_drive`）。
- `server/app/api/endpoints/group_auto_drive.py`: 接力讨论 API（start/stream/pause/resume/stop/state）。
- `server/app/services/group_auto_drive_service.py`: 接力讨论主流程与状态机。
- `server/app/schemas/group_auto_drive.py`: 接力讨论请求/响应模型。
- `server/app/models/group.py`: `group_auto_drive_runs` 数据表模型。
- `server/app/prompt/auto_drive/`: 接力讨论提示词（目录名仍为 `auto_drive`）。
- `server/alembic/versions/9c4b1a2d3e5f_add_group_auto_drive.py`: 接力讨论数据表迁移。

#### 📁 Infrastructure
*   **`alembic/`**: Production-ready database migrations.
*   **`data/`**: Storage for `.db` files.
    *   `doudou.db`: Primary application data.
    *   `memobase.db`: Memory/Vector storage.
*   **`logs/`**: Backend log files.
    *   `app.log`: Application runtime logs (rotated daily).
*   **`tests/`**: Pytest suite.
    *   `test_chat_api.py`, `test_chat_persistence.py`: Chat functionality tests.
    *   `test_friend_api.py`: Friend management tests.
    *   `test_llm_api.py`: LLM configuration tests.
    *   `test_memo_bridge.py`, `test_memory_extraction_complex.py`: Memory system tests.
    *   `test_persona_generator.py`: AI persona generation tests.
    *   `test_profile_api.py`: User profile tests.
    *   `test_recall_service.py`: Memory recall tests.
    *   `test_session_archive.py`, `test_session_selection.py`: Session management tests.
    *   `test_sse_simple.py`, `test_sse_timing.py`: SSE streaming tests.

---

### 🖥️ Desktop (`electron/`)
Electron wrapper for packaging the app as a standalone desktop application.

#### 📁 Core Files
*   **`main.js`**: Main process entry point.
    *   Manages backend lifecycle (spawn, health check, shutdown)
    *   Creates splash window during startup
    *   Creates main window after backend is ready
    *   Handles port allocation and data directory injection
*   **`preload.js`**: Context bridge for frontend-backend communication.
    *   Exposes `window.__BACKEND_PORT__` and `window.__BACKEND_URL__`
    *   Exposes `window.WeAgentChat` object with backend info
*   **`splash.html`**: Loading screen shown during backend startup.

#### 📁 Configuration (Root Directory)
*   **`package.json`**: Electron dependencies and scripts.
    *   `pnpm electron:dev` - Run in development mode
    *   `pnpm electron:build` - Build production installer
    *   `pnpm electron:pack` - Package without installer (for testing)
*   **`electron-builder.yml`**: Packaging configuration.
    *   App ID: `com.weagent.chat`
    *   Output: `dist-electron/`
    *   Backend bundled as `extraResources`

#### 📁 Build Artifacts
*   **`build/backend/`**: PyInstaller output (bundled Python backend)
*   **`dist-electron/`**: Final packaged application

#### 🔧 Environment Variables
*   `DOU_DOUCHAT_DEV_SERVER_URL`: Override dev server URL (default: `http://localhost:5173`)
*   `DOU_DOUCHAT_BACKEND_PORT`: Override backend port in dev mode (default: `8000`)
*   `DOU_DOUCHAT_BACKEND_EXE`: Explicit path to backend executable
*   `WeAgentChat_DATA_DIR`: Data directory for production (auto-set to AppData)

---

### 📄 Documentation & Planning (`dev-docs/`)
*   **`prd/`**: High-level requirements and visual identity.
*   **`userStroy/`**: Business logic and feature requirements (e.g., `passive_session_memory.md`).
*   **`coding/`**: Granular implementation plans (Divided by Epics: `epic_01/`, `epic_02/`, `epic_03/`, `epic_04/`).
*   **`swagger-api/`**: API definitions (Legacy/Reference).
*   **`troubleshooting/`**: Guides for common dev issues.
*   **`tutorial/`**: User tutorials and configuration guides (includes PDF documentation).
*   **`scripts/`**: Development and deployment scripts.
*   **`tests/`**: Test-related documentation.
*   **`temp/`**: Temporary documents (e.g., `electron_packaging_plan.md`).

---

### 🌐 Website (`website/`)
Project landing page and promotional assets.
*   `index.html`: Main landing page.
*   `styles.css`: Page styling.
*   `main.js`: Interactive scripts.
*   `assets/`: Screenshots and media resources.

### 📁 Static Assets (`server/static/`)
*   **`avatars/`**: Pre-generated avatar images and presets for friends.

---

## Development Roadmap
1.  Core chat functionality with WeChat-style UI
2.  Dual-track memory system implementation
3.  AI Moments & Dynamic feed system
4.  Passive session management
5.  Mobile adaptation (PWA)

## Conventions & Notes
*   **Directory Naming:** The physical directories are `front` and `server`.
*   **Language:** The documentation and primary communication for this project are in Chinese (zh-CN).
*   **pnpm:** The `pnpm` package manager is used for dependency management. It is recommended to use `pnpm` instead of `npm` or `yarn`.
*   **Backend Environment:**
    *   **Virtual Environment:** A virtual environment is located at `server/venv/`.
    *   **Run Server:** Execute `server\venv\Scripts\python -m uvicorn app.main:app --reload` within the `server` directory to start the backend with auto-reload.
    *   **Database Operations:** 使用 `sqlite3` 命令（已配置全局环境变量）直接操作数据库文件（如 `sqlite3 server/data/doudou.db`）。
    *   **Database Migrations (Alembic):**
        *   **Automatic Update:** The server automatically applies the latest migrations on startup (`init_db.py` calls `alembic upgrade head`).
        *   **Generate Migration:** Run `scripts\gen_migration.bat` in the project root to generate a new migration script after modifying SQLAlchemy models.
        *   **Manual Operations:** See `server/ALEMBIC_SETUP.md` for detailed Alembic commands.
    *   **UI Design:** **所有的 UI 界面必须高度参考微信 (WeChat) 的视觉风格和交互体验。** 这包括但不限于：
    *   配色方案（如微信绿、浅灰色渐变背景等）。
    *   布局（侧边栏、对话列表、聊天窗口的排布）。
    *   交互细节（点击反馈、对话气泡样式等）。
*   **Prompt Management (LLM Prompts):**
    *   **禁止硬编码：** 所有用于 LLM 调用的 Prompt 文本**必须**维护在 `server/app/prompt/` 目录下的独立文件中，**严禁**在代码中硬编码 Prompt 字符串。
    *   **加载方式：** 使用 `server/app/prompt/loader.py` 中的 `load_prompt(category, name)` 函数加载 Prompt 文件。
    *   **Prompt 分类目录：**
        *   `chat/`: Chat-related prompts (system prompts, context templates).
        *   `persona/`: Persona generation prompts.
        *   `memory/`: Memory-related prompts (e.g., summary hints).
        *   `recall/`: Memory recall agent prompts.
        *   `tests/`: Test prompts.
*   **Unit Testing:** Run tests using `server\venv\Scripts\python -m pytest server/tests`.
*   **Logging:** Backend logs are output to the console and saved to `server/logs/app.log`, with daily rotation and 30-day retention.

---

# Memobase SDK (Memory System)

"双轨长期记忆系统" (Dual-Track Long-Term Memory System) 现在作为嵌入式 SDK 集成在主后端服务中，为 LLM 应用提供持久化、上下文感知的记忆能力。

-   **Integration:** Embedded SDK (`server/app/vendor/memobase_server`)
-   **Runtime:** 主 FastAPI 进程内运行 (Managed by `server/app/main.py` lifespan)
-   **Database:** `server/data/memobase.db` (SQLite + sqlite-vec)
-   **Configuration:** 统一通过主项目 `server/app/core/config.py` 管理

### Configuration (Environment Variables)

需要在 `.env` 或环境变量中配置记忆系统专用的 Key：

*   `MEMOBASE_LLM_API_KEY`: 用于提取记忆的 LLM API Key
*   `MEMOBASE_LLM_BASE_URL`: (可选) LLM Base URL
*   `MEMOBASE_ENABLE_EVENT_EMBEDDING`: 是否启用向量检索 (Default: `True`)
*   `MEMOBASE_EMBEDDING_API_KEY`: 用于向量化的 Embedding API Key
*   `MEMOBASE_EMBEDDING_BASE_URL`: (可选) Embedding Base URL

### Architecture
此模块不再作为独立服务 (`mem-system`) 运行。
*   **Bridge Layer**: `server/app/services/memo/bridge.py` 负责将主配置注入 SDK 并封装调用。
*   **Background Worker**:  主服务启动时自动挂载后台任务，用于异步处理记忆提取和归档。

---
> Source: [0000005/WeAgentChat](https://github.com/0000005/WeAgentChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
