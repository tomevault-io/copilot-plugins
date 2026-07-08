## mediaplayerx

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

MediaPlayerX 是一个基于 Electron + React 的本地优先综合媒体库管理器（图片 + 视频 + 音频）。当前版本 `0.75`，处于 **Electron 实用化 + 质量收敛阶段**。

## 环境要求

- Node.js `22.x` + npm
- **Windows 注意**：不要以管理员权限启动 Electron（如管理员终端执行 `npm run dev:desktop`），否则触发 UAC 权限隔离，拖拽文件到窗口会被系统阻断。

## 常用命令

```bash
# 开发
npm run dev             # 浏览器模式（无 Electron）
npm run dev:desktop     # Electron 开发模式（主入口）

# 构建
npm run build           # tsc + Vite 前端构建
npm run build:electron  # 构建 Electron 主进程
npm run desktop:start   # 完整打包后直接启动 Electron

# 质量检查（提交前按序执行）
npm run format:check    # Prettier 检查
npm run lint            # ESLint（必须 0 warning）
npm run test            # 运行全部测试（vitest run）
npm run test:coverage   # 带覆盖率报告
npm run build           # 确认构建通过
npm run format:write    # Prettier 自动修复

npx madge --circular src electron  # 检查循环依赖（必须归零）

# 测试
npm run test:watch                          # 监听模式
npx vitest run src/path/to/file.test.ts     # 运行单个测试文件

# 完整 CI 质量门禁（对齐 CI 时执行）
npm run quality:ci
npm run build:electron
```

代理配置（按需）：
```bash
MEDIA_PLAYERX_PROXY_SERVER=socks5://127.0.0.1:2080 npm run dev:desktop
```

## 整体架构

应用分为三个进程层，通过严格的 IPC 协议通信：

```
Electron Main Process (electron/)
  └─ FileSystemMediaReadService   # 后端总协调器
       ├─ MediaLibraryDatabase    # SQLite 持久层（WAL 模式）
       ├─ 10+ 专项 Service        # 各司其职（见下文）
       └─ ServiceEventBus        # 内部事件总线
          ↓ IPC（~70 个 channel）
Preload / IPC Bridge
  └─ registerBackendIpcHandlers   # 注册所有 IPC handler
  └─ Zod schema 双向验证          # IPC 边界的类型保证
          ↓
React Renderer Process (src/)
  └─ MediaRepository              # IPC 的前端抽象层
       ├─ RealMediaRepository     # 生产环境，包装 IPC 调用
       └─ MockMediaRepository     # 测试环境，内存实现
  └─ 数据 Pipeline（useApp* hooks）
```

### 后端关键文件

| 文件 | 职责 |
|------|------|
| `electron/fileSystemReadFacade.ts` | 后端总协调器，创建并持有所有服务 |
| `electron/mediaLibraryDatabase.ts` | SQLite 数据库封装，含多个 Store 类 |
| `electron/registerBackendIpcHandlers.ts` | 注册全部 ~70 个 IPC channel |
| `electron/channels.ts` | IPC channel 常量定义（唯一真相源） |
| `electron/facade/` | 将 IPC handler 按领域分组（Library / Management / System） |
| `electron/services/file-system-read/` | 所有专项服务实现 |
| `electron/services/task-orchestrator/` | 任务调度与 Worker 线程管理 |

### 前端关键文件

| 文件 | 职责 |
|------|------|
| `src/contracts/backend.schemas.ts` | IPC 消息的 Zod schema（与后端共用） |
| `src/contracts/backend.types.ts` | 从 schema 推导的 TypeScript 类型 |
| `src/backend-api.d.ts` | preload 注入到 window 的 API 类型声明 |
| `src/features/backend/repository/types.ts` | `MediaRepository` 接口定义 |
| `src/features/backend/repository/realRepository.ts` | 生产 IPC 实现 |
| `src/features/backend/repository/mockRepository.ts` | 测试内存实现 |
| `src/features/app/useAppDataPipeline.ts` | 前端数据 Pipeline 入口 |

### 前端数据流 Pipeline

`useAppDataPipeline` 按顺序组合四个 hook：

1. **`useAppRuntimeSources`** — 采集所有外部数据源（settings、import state、repository bootstrap）
2. **`useAppReadAndNavigation`** — 根据用户导航动作计算当前可见内容
3. **`useAppDisplayAndEffects`** — 从导航与媒体数据推导 UI 状态
4. **`useAppViewComposition`** — 组装最终渲染树

### `src/features/` 目录结构

```
app/          # 应用级 Pipeline hooks
backend/      # Repository 抽象层与 mappers
import/       # 导入流程 UI 与状态
layout/       # 顶层布局组件
management/   # 文件管理操作（删除/移动/重命名）
media/        # 图片/视频/音频媒体展示
metadata/     # 元数据查询与编辑（nhentai/ehentai 链路）
music-visualizer/ # 音乐可视化
perf/         # 性能基准与 bench 工具
search/       # 搜索功能
shared/       # 跨 feature 共享工具
sidebar/      # 侧边栏树形导航
subtitles/    # 字幕生成（sherpa-onnx）
theme/        # 主题系统（Style x Palette）
```

## 关键设计约定

### IPC 协议
- 所有 IPC 消息通过 Zod schema 在两端验证，修改 IPC 协议需同时更新 `electron/channels.ts`、`src/contracts/backend.schemas.ts` 和 `registerBackendIpcHandlers.ts`。
- 区分两种模式：`registerIpcQuery`（只读）和 `registerIpcCommand`（有副作用）。
- IPC 边界层级顺序：`contracts → preload → ipc → main/repository`，不允许跨层直接依赖。

### 资源治理
- `TaskResourceGovernor` 通过 CPU/GPU token 机制防止后台任务耗尽资源，新增 Worker 任务需接入此机制。

### 测试策略
- 前端测试使用 `MockMediaRepository`，无需启动 Electron 进程。
- 测试文件使用 jsdom 环境（`vitest` + `@testing-library/react`）。
- 集成测试命名后缀为 `.integration.test.tsx`，焦点测试为 `.focus.test.tsx`。

### 编码规范
- 所有注释、文档、commit message 使用**中文**。
- 全面使用 TypeScript，禁止 `any`。
- 使用 ES Modules（`import`/`export`），不用 CommonJS。
- 质量门禁：`lint` 0 warning，`madge --circular` 0 循环依赖，`npm audit --audit-level=high` 0 high/critical。
- 提交格式（Conventional Commits）：`<type>(<scope>): <subject>`，分支命名：`<type>/<topic>`。

## 文档体系（SSOT）

`docs/` 根目录为单一事实源，历史文档归档至 `docs/archive/`，代码与文档冲突时以当前代码行为为准再同步修订。

| 改动类型 | 需同步更新的文档 |
|----------|-----------------|
| 需求变更 | `docs/03-requirements-v1.md` |
| 架构/模块边界变更 | `docs/04-architecture-v1.md` |
| UI/交互/快捷键变更 | `docs/05-interaction-v1.md` |
| 后端/IPC 接入规则变更 | `docs/06-backend-integration-guardrails.md` |
| i18n/a11y 规则变更 | `docs/07-i18n-aria-guardrails.md` |
| Theme token/风格规则变更 | `docs/08-theme-system-v2.md` 与 `docs/09-theme-brainstorm-entry.md` |
| Shader 运行时/迁移规则变更 | `docs/12-music-visualizer-shader-entry.md` 与 `docs/13-music-visualizer-shader-migration-playbook.md` |
| 文档目录结构变更 | `docs/02-DOCS_INDEX.md` 与 `docs/01-README.md` |

**UI 槽位表维护约束**：新增/修改/删除 UI 槽位时，`docs/10-ui_definition.md` 与 `docs/11-token_design.md` 必须同步操作。

---
> Source: [Darkstarrd-dev/MediaPlayerX](https://github.com/Darkstarrd-dev/MediaPlayerX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
