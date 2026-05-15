## cc-switch-web

> 本文件为 Claude Code（claude.ai/code）在此仓库中操作时提供指导。

# CLAUDE.md

本文件为 Claude Code（claude.ai/code）在此仓库中操作时提供指导。

## 项目概述

CC Switch 是一个 Tauri 2 桌面应用，用于管理 AI CLI 工具的配置（Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw）。它提供统一的界面来切换 provider、管理所有五个工具的 MCP 服务器、prompts 和 skills，内置 50+ 个 provider 预设和 SQLite 数据存储。

## 本地系统环境
- Ubuntu 20.04
- Node.js 20.20.2
- pnpm 10.13.1
- Rust 1.95.0

## 开发命令

```bash
# 前端
pnpm dev              # 开发模式（热重载，pnpm tauri dev）
pnpm build            # 生产构建（pnpm tauri build）
pnpm dev:renderer     # 仅启动 Vite 渲染进程
pnpm build:renderer   # 仅构建渲染进程

# 类型检查
pnpm typecheck        # TypeScript 类型检查（tsc --noEmit）

# 代码格式化
pnpm format           # 使用 Prettier 格式化
pnpm format:check     # 检查格式

# 前端测试
pnpm test:unit        # 运行 vitest 测试
pnpm test:unit:watch  # 监听模式

# Rust 后端（在 src-tauri/ 目录下运行）
cargo fmt             # 格式化 Rust 代码
cargo clippy          # Lint 检查
cargo test            # 运行所有 Rust 测试
cargo test test_name  # 运行指定测试
cargo test --features test-hooks  # 使用 test-hooks 特性运行测试

# Web 服务器模式（Ubuntu 20.04 兼容）
./run-web.sh                               # 构建并前台启动（Ctrl+C 停止）
./run-web.sh start                         # 后台启动
./run-web.sh stop                          # 停止服务
./run-web.sh restart                       # 重启
./run-web.sh status                        # 查看运行状态
./run-web.sh logs                          # 查看后台日志
cargo build -p web-server                  # 仅编译 web 服务器
cargo run -p web-server                    # 编译+启动（前台）

# 原生桌面窗口
cd native-shell && cargo build --release   # 编译原生窗口包装
cd .. && native-shell/target/release/native-shell  # 启动原生窗口
cargo run --manifest-path native-shell/Cargo.toml  # 或直接编译运行

# .deb 打包（Ubuntu 20.04 发布）
./build-deb.sh                             # 构建 .deb 安装包
./run-web.sh package                       # 同上（通过管理脚本）
sudo dpkg -i cc-switch-web_*.deb           # 安装 .deb
```

## 架构

### 全栈分层架构（Tauri 2）

**前端（React 18 + TypeScript + Vite）**
- 路径别名：`@/` 映射到 `src/`
- 状态管理：TanStack Query v5 管理服务端/缓存状态
- UI：TailwindCSS 3.4、shadcn/ui（Radix 原语）、react-hook-form + zod
- 国际化：react-i18next，语言文件位于 `src/i18n/`（zh/en/ja）
- 拖拽：@dnd-kit
- 编辑器：CodeMirror 6（JSON/Markdown 编辑器）

**后端（Rust + Tauri 2）**
- `src-tauri/src/` 内清晰分层架构：
  - `commands/` — Tauri IPC 命令处理层（薄层，委托给 services）
  - `services/` — 业务逻辑（providers、MCP、proxy、session 等）
  - `database/` — SQLite DAO 层及数据库迁移
- 每个 CLI 工具有独立配置模块：`claude_plugin.rs`、`codex_config.rs`、`gemini_config.rs`、`hermes_config.rs`、`opencode_config.rs`、`openclaw_config.rs`

### 测试

- **前端**：vitest + jsdom + @testing-library/react + MSW（模拟 Tauri IPC）
  - 测试设置文件：`tests/setupGlobals.ts`、`tests/setupTests.ts`
  - MSW handlers 位于 `tests/msw/`
- **后端**：标准 `cargo test`，可选 `test-hooks` 特性

### 关键领域模块

- **Providers** — CRUD、切换、回填、排序、50+ 预设
- **MCP** — 跨 4 个应用的统一 MCP 服务器管理、双向同步、Deep Link 导入
- **Prompts/Skills** — Markdown 编辑器、GitHub 仓库安装、跨应用同步
- **Proxy** — 本地代理热切换、格式转换、自动故障转移、断路器
- **Session Manager** — 浏览/搜索/恢复所有应用中的对话历史
- **WebDAV 同步** — 通过 WebDAV 服务器进行云同步
- **用量/费用追踪** — 消费金额、请求数、Token 数及趋势图表

### 数据存储

- **数据库**：`~/.cc-switch/cc-switch.db`（SQLite）
- **设置**：`~/.cc-switch/settings.json`（设备级别）
- **备份**：`~/.cc-switch/backups/`（自动轮换，保留 10 份）
- 设计模式：单一数据源（SSOT）、双层存储、原子写入（临时文件 + 重命名）、互斥锁保护数据库

## 提交前检查

确保以下命令通过：`pnpm typecheck`、`pnpm format:check`、`pnpm test:unit`。


## 编译开发要求
1. 如果编译失败需要安装环境则需要记录在CLAUDE.md文档内
2. **Ubuntu 20.04 (Focal) 不支持编译 Tauri 2** — 需要 Ubuntu 22.04+，因为：
   - 缺少 `libwebkit2gtk-4.1-dev`（Tauri 2 必需，20.04 仅有 4.0 版本）
   - 缺少 `glib-2.0 >= 2.70`（20.04 仅提供 2.64）
   - 缺少 `libsoup-3.0-dev`
   - 解决方案：升级系统到 22.04+，或使用 Docker 容器编译
3. 前端开发可在 20.04 上独立运行：`pnpm dev:renderer`（Vite 开发服务器，地址 http://localhost:3000）
4. **原生桌面窗口（native-shell）**：
   - 独立 crate，不依赖 Tauri 2
   - 需要 `libwebkit2gtk-4.0-dev`（Ubuntu 20.04 有，无需升级系统）
   - 编译依赖：`sudo apt install libgtk-3-dev libwebkit2gtk-4.0-dev`
   - 使用 `web-view` Rust 包装，链接 WebKit2GTK 4.0（非 4.1）
   - 构建：`cd native-shell && cargo build --release`
   - 构建的 `.deb` 包包含 `cc-switch-web`（原生窗口，终端输入或双击启动）和 `cc-switch-web-server`（HTTP 服务器）两个二进制
5. **web-server 模式（Ubuntu 20.04 推荐）**：
   - `src-tauri/` 中的 `desktop` feature 将所有 Tauri 依赖设为可选
   - `web-server/` 是一个独立 workspace 成员，以 HTTP 服务器模式运行 CC Switch
   - 前端所有 Tauri API 调用通过 Vite alias 重定向到 `src/lib/api/tauri-mock/`，通过 HTTP 与后端通信
   - 后端 REST API 通过 `POST /api/invoke` 统一入口路由 100+ 个命令
   - `run-web.sh` 启动脚本，支持 start/stop/restart/status/logs 子命令
   - web-server 不可用的功能：系统托盘、开机自启、最小化到托盘、原生窗口控制、Deep Link、自动更新
   - web-server 中通过浏览器兼容层（tauri-mock）实现的部分功能：SQL 备份导入导出、ZIP 文件安装（使用浏览器文件选择器上传）
   - web-server 需特别注意的 API 兼容问题：返回简单类型（string/bool）的命令不能包装为 JSON 对象，必须返回原始值；返回数组的命令不能返回对象


  ## 仓库介绍文档
  `README.md`

---
> Source: [greluoqixi/cc-switch-web](https://github.com/greluoqixi/cc-switch-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
