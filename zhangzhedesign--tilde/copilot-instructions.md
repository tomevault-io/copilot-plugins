## tilde

> **GridDo** 是一款基于 Tauri 2 的轻量级桌面 Todo 看板应用，定位为常驻桌面的高效任务管理工具。采用紧凑的纯黑无边框窗口设计，随时可用，支持语音快速录入。

# GridDo — 项目开发指南

## 项目概述

**GridDo** 是一款基于 Tauri 2 的轻量级桌面 Todo 看板应用，定位为常驻桌面的高效任务管理工具。采用紧凑的纯黑无边框窗口设计，随时可用，支持语音快速录入。

### 核心特性

- **紧凑看板**：主窗口固定宽度 320px，高度自适应任务数量，frameless 设计
- **快速录入**：按 `~` 键唤起独立 Entry 窗口快速添加任务
- **语音输入**：按住 `Alt` 键语音录入，集成离线语音识别（CapsWriter-Offline）
- **任务管理**：支持待办/进行中/已完成三种状态，可设置紧急标记、进度百分比、重复任务（每日/工作日）
- **历史回顾**：按日期分组查看过往任务，支持无限滚动加载
- **任务导出**：按日期范围和关键词筛选，导出为 Markdown 或 CSV

### 目标平台

Windows（Tauri 桌面应用）

---

## 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 桌面框架 | Tauri | v2 |
| 前端框架 | React | v19.1.0 |
| 语言 | TypeScript | ~5.8.3 |
| 构建工具 | Vite | v7.0.4 |
| 样式 | Tailwind CSS | v4.2.2 |
| 状态管理 | Zustand | v5.0.12 |
| 动画 | Framer Motion | v12.38.0 |
| 图标 | Lucide React | v1.8.0 |
| 日期处理 | date-fns | v4.1.0 |
| 数据库 | SQLite | tauri-plugin-sql |
| Rust 异步 | Tokio | v1 |
| 音频采集 | cpal | v0.15 |
| WebSocket | tokio-tungstenite | v0.24 |

---

## 架构说明

### 双窗口架构

| 窗口 | Label | 尺寸 | 特性 |
|------|-------|------|------|
| 主看板 | `main` | 320 x 自适应 | frameless，可置顶，位置记忆 |
| 快速输入 | `entry` | 480 x 72 | frameless，始终置顶，不在任务栏显示 |

- 前端通过 `@tauri-apps/api/window` 的 `getCurrentWindow().label` 判断当前窗口类型，渲染不同组件
- `entry` 窗口默认隐藏，按 `~` 键或点击 "+ New Task" 唤起

### 视图管理

- **无前端路由**（无 React Router）
- 采用 Zustand 的 `view` 状态在 `'board'` 和 `'history'` 间切换
- 多窗口显示/隐藏由 Tauri 后端控制

### 前后端通信

- 前端通过 `invoke` 调用 Rust 命令
- 后端通过 `emit` 发送事件给前端
- 语音数据通过 WebSocket 在 Rust 与本地 CapsWriter 服务之间传输

### Rust 后端职责

- 窗口管理（位置、尺寸、置顶状态）
- 全局快捷键注册（`~` 键切换 entry 窗口）
- 语音服务器生命周期管理（启动/停止/自动重启）
- 音频采集（cpal）与重采样
- WebSocket 客户端（连接 CapsWriter）

---

## 关键文件索引

### Rust 后端

| 文件 | 说明 |
|------|------|
| [src-tauri/src/lib.rs](src-tauri/src/lib.rs) | 核心逻辑：窗口管理、快捷键、语音服务、录音、WebSocket |
| [src-tauri/src/main.rs](src-tauri/src/main.rs) | 程序入口，调用 `griddo_lib::run()` |
| [src-tauri/tauri.conf.json](src-tauri/tauri.conf.json) | Tauri 应用配置：双窗口定义、打包配置 |
| [src-tauri/Cargo.toml](src-tauri/Cargo.toml) | Rust 依赖配置 |

### 前端核心

| 文件 | 说明 |
|------|------|
| [src/main.tsx](src/main.tsx) | React 入口，按窗口 label 渲染 `App` 或 `EntryBox` |
| [src/App.tsx](src/App.tsx) | 根组件，初始化数据库，管理视图切换 |

### 组件

| 文件 | 说明 |
|------|------|
| [src/components/Board.tsx](src/components/Board.tsx) | 看板主容器：标题栏、任务列表、底部操作栏、动态窗口高度 |
| [src/components/TaskItem.tsx](src/components/TaskItem.tsx) | 任务行：状态圆点、文字编辑、进度环、悬停操作 |
| [src/components/EntryBox.tsx](src/components/EntryBox.tsx) | 双模式输入框（Standalone/Inline）+ 语音输入状态机 |
| [src/components/HistoryPage.tsx](src/components/HistoryPage.tsx) | 历史任务浏览页，按日期分组，无限滚动加载 |
| [src/components/DateGroup.tsx](src/components/DateGroup.tsx) | 历史页中的日期分组展示容器 |
| [src/components/ProgressDial.tsx](src/components/ProgressDial.tsx) | 圆形进度转盘，支持拖拽/滚轮/点击边缘设置进度 |
| [src/components/PieProgress.tsx](src/components/PieProgress.tsx) | 饼形进度指示器（SVG），用于任务行小进度展示 |
| [src/components/HoverActions.tsx](src/components/HoverActions.tsx) | 悬停时的快捷操作：紧急标记、循环标记、删除 |
| [src/components/ExportModal.tsx](src/components/ExportModal.tsx) | 任务导出弹窗，支持 Markdown/CSV 格式 |
| [src/components/DatePicker.tsx](src/components/DatePicker.tsx) | 自定义日期选择器，基于 date-fns |
| [src/components/Toast.tsx](src/components/Toast.tsx) | 底部提示条，支持撤销删除操作 |

### 业务逻辑与数据

| 文件 | 说明 |
|------|------|
| [src/hooks/useTasks.ts](src/hooks/useTasks.ts) | 任务 CRUD Hook：加载、排序、循环任务生成、逾期迁移 |
| [src/services/db.ts](src/services/db.ts) | SQLite 数据库操作：tasks 表、daily_snapshots 表、重复任务调度 |
| [src/services/export.ts](src/services/export.ts) | Markdown/CSV 导出逻辑 |
| [src/services/storage.ts](src/services/storage.ts) | 本地存储辅助 |
| [src/stores/appStore.ts](src/stores/appStore.ts) | Zustand 全局状态：视图、弹窗、Toast、任务展开状态 |
| [src/types/index.ts](src/types/index.ts) | TypeScript 类型定义：Task、DailySnapshot、DateGroupData |

### 工具与样式

| 文件 | 说明 |
|------|------|
| [src/utils/date.ts](src/utils/date.ts) | 日期处理工具（getTodayString、getDateLabel 等） |
| [src/utils/format.ts](src/utils/format.ts) | 格式化辅助（formatPercent、generateId） |
| [src/utils/window.ts](src/utils/window.ts) | 窗口相关工具（requestBoardResize 自定义事件） |
| [src/styles/globals.css](src/styles/globals.css) | 全局样式：Tailwind 入口 + CSS 变量 + 自定义滚动条 |

---

## Tauri 命令与事件

### 前端调用命令（invoke）

| 命令 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `save_window_position` | `x: f64, y: f64` | `()` | 保存主窗口位置到 `window-config.json` |
| `load_window_position` | - | `Option<WindowConfig>` | 加载主窗口位置配置 |
| `set_always_on_top` | `value: bool` | `()` | 设置置顶状态并持久化 |
| `load_always_on_top` | - | `bool` | 加载置顶状态配置 |
| `show_entry_window` | - | `()` | 显示并居中 entry 窗口 |
| `hide_entry_window` | - | `()` | 隐藏 entry 窗口，通知主窗口刷新 |
| `start_voice_server` | - | `Result<(), String>` | 启动 CapsWriter 语音服务器 |
| `stop_voice_server` | - | `Result<(), String>` | 停止语音服务器 |
| `toggle_recording` | - | `Result<bool, String>` | 切换录音状态 |

### 后端事件（emit）

| 事件名 | 触发时机 | 数据 |
|--------|----------|------|
| `voice-ready` | 语音服务器启动完成 | - |
| `voice-result` | 语音识别完成 | `string`（识别文本） |
| `voice-error` | 语音识别出错 | `string`（错误信息） |
| `refresh-tasks` | entry 窗口隐藏后 | - |
| `entry-window-shown` | entry 窗口显示时 | - |
| `entry-window-hidden` | entry 窗口隐藏时 | - |

---

## 数据模型

### Task

```typescript
interface Task {
  id: string;
  content: string;
  status: 'pending' | 'in_progress' | 'completed';
  urgency: 'normal' | 'urgent';
  progress: number;        // 0-100
  asset_path: string | null;
  created_at: number;
  completed_at: number | null;
  scheduled_date: string;  // YYYY-MM-DD
  overdue_days: number;
  recurring_type: 'none' | 'daily' | 'workday';
}
```

### 排序规则

紧急 > 循环 > 普通 > 逾期 > 已完成

### 数据库表

- `tasks`：任务主表
- `daily_snapshots`：每日统计（总任务数、完成数、完成率）

---

## 开发规范

### 样式

- 使用 Tailwind CSS 原子类，避免自定义 CSS
- 主题：纯黑背景（`bg-black`）+ 白色文字，所有颜色硬编码，**无主题切换系统**
- 窗口动态高度通过 Tauri API 实时调整

### 状态管理

- 全局状态优先使用 Zustand（[src/stores/appStore.ts](src/stores/appStore.ts)）
- 局部状态使用 `useState`/`useReducer`
- 窗口显隐状态通过 Tauri 命令控制，不在前端状态维护

### 数据操作

- 所有数据库操作通过 [src/services/db.ts](src/services/db.ts) 集中管理
- 使用 tauri-plugin-sql 的 `execute` 和 `select` 方法
- 业务逻辑（排序、循环任务生成、逾期迁移）在 [useTasks.ts](src/hooks/useTasks.ts) 中处理

### 语音功能开发

- 语音服务依赖本地 CapsWriter-Offline 资源文件
- 开发前确认 `src-tauri/resources/capswriter/` 目录存在
- 录音状态由 Rust 原子变量控制，避免并发竞态

---

## 已知问题与限制

### 打包与资源

- `bundle.resources` 当前为 `[]`，**打包后语音资源路径可能失效**
- 建议验证 `tauri build` 后 `resource_dir()` 在生产环境是否可读取

### 窗口与显示器

- 位置记忆使用绝对屏幕坐标，多显示器切换时窗口可能出现在屏幕外
- `entry` 窗口不在任务栏显示，用户可能丢失窗口，需考虑托盘图标

### 语音功能

- CapsWriter 文件缺失时语音服务器启动失败
- `encode_wav` 函数已废弃，产生编译 warning
- 语音服务器崩溃无自动重启机制

### 快捷键

- 全局快捷键 `Backquote`（`~` 键）可能被其他应用占用
- 若提示 `HotKey already registered`，需结束所有 griddo 进程后重试

### UI 不完善项

- `in_progress` 状态已定义但无独立视觉，仅在展开进度环时自动进入
- 长任务文字在 320px 宽度下被截断，需悬停查看 title 提示
- 历史页打开时窗口高度骤变至 520px，视觉跳跃感明显

### 未开发功能

- 附件上传（`asset_path` 字段已预留）
- 浅色主题切换
- 多显示器位置记忆优化
- 语音服务器自动重启

---

## 常见开发任务速查

### 启动开发环境

```bash
npm run tauri dev
```

### 生产构建

```bash
npm run tauri build
```

### 添加 Rust 命令

1. 在 [src-tauri/src/lib.rs](src-tauri/src/lib.rs) 中定义 `#[tauri::command]` 函数
2. 在 `invoke_handler` 中用 `generate_handler!` 注册
3. 前端通过 `invoke('command_name', args)` 调用

### 添加前端组件

1. 在 [src/components/](src/components/) 下新建 `.tsx` 文件
2. 在父组件或 [App.tsx](src/App.tsx) 中引用
3. 使用 Tailwind 原子类编写样式

### 修改数据库 Schema

1. 在 [src/services/db.ts](src/services/db.ts) 的 `initDatabase()` 中调整 `CREATE TABLE` 语句
2. 在 [src/types/index.ts](src/types/index.ts) 中更新类型定义
3. 如需数据迁移，编写 `ALTER TABLE` 或重建表逻辑

### 调试语音功能

1. 检查 Rust 控制台日志，确认 CapsWriter 进程启动状态
2. 确认 `resources/capswriter/CapsWriter-Offline/` 目录存在
3. 监听 `voice-ready`、`voice-error` 事件排查连接问题
4. 检查默认音频输入设备是否可用

---

## 依赖关系与构建要求

### 必需环境

- [Node.js](https://nodejs.org/) 和 npm
- [Rust](https://rustup.rs/) 工具链（cargo）
- [Tauri v2 CLI](https://v2.tauri.app/reference/cli/)：`cargo install tauri-cli`
- Windows：需安装 [Microsoft C++ 生成工具](https://visualstudio.microsoft.com/visual-cpp-build-tools/)

### 语音功能额外依赖

- 本地 CapsWriter-Offline v2.4 资源文件（放在 `src-tauri/resources/capswriter/`）
- 系统默认音频输入设备

### 前端依赖安装

```bash
npm install
```

---

## 项目文件结构

```
griddo/
├── index.html                  # 入口 HTML
├── package.json                # 项目配置与依赖
├── vite.config.ts              # Vite 配置（端口 1423）
├── tsconfig.json               # TypeScript 配置
├── src/
│   ├── main.tsx                # React 入口
│   ├── App.tsx                 # 根组件
│   ├── components/             # React 组件
│   ├── hooks/                  # 自定义 Hooks
│   ├── services/               # 数据服务层
│   ├── stores/                 # Zustand 全局状态
│   ├── types/                  # TypeScript 类型
│   ├── utils/                  # 工具函数
│   └── styles/                 # 全局样式
├── src-tauri/
│   ├── Cargo.toml              # Rust 依赖
│   ├── tauri.conf.json         # Tauri 配置
│   ├── build.rs                # 构建脚本
│   ├── src/
│   │   ├── main.rs             # Rust 入口
│   │   └── lib.rs              # 核心逻辑
│   ├── capabilities/           # 权限配置
│   ├── icons/                  # 应用图标
│   └── resources/              # 打包资源（语音服务等）
└── README.md
```

---
> Source: [zhangzhedesign/tilde](https://github.com/zhangzhedesign/tilde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
