## nomi-core

> > 这份文件是给接手 `nomi-core` 的 agent 使用的项目维护规则，不是正式用户文档。

# Nomi Project Guide

> 这份文件是给接手 `nomi-core` 的 agent 使用的项目维护规则，不是正式用户文档。

## Repo Ownership

当前三仓职责固定如下：

- `nomi-core`
  - Python 核心仓。
  - 负责 CLI、runtime、agent、session、provider、tools、tasks、cron、channel、remote server、instance channel、config。
- `nomi-desktop`
  - Tauri + React 桌面端仓。
  - 负责 desktop UI、desktop state、Tauri 窗口、桌面本地持久化和构建。
- `nomi-protocol`
  - 共享协议仓。
  - 负责 remote HTTP + SSE contract、schema、版本和共享类型。

当前 Codex 分工：

- 本仓 Codex 默认 owner 是 `nomi-core`。
- desktop 侧由另一个 Codex 负责 `/Users/lixinlv/Doing/nomi-desktop`。
- `nomi-protocol` 默认由 core owner 修改和发版。

## Cross-repo Rules

涉及 desktop 的需求，先判断是不是 core 仓职责：

- UI、前端状态、Tauri、窗口行为、desktop 本地存储问题，不直接改 `nomi-desktop`，交给 desktop owner。
- core 需要配合 desktop 时，先明确接口字段、错误语义和验收方式，再改 core。
- remote 协议变更必须先获得用户明确同意，再改 `nomi-protocol`。
- 不在 core 私自发明会影响 desktop 的 remote wire 语义。

协议变更固定流程：

1. 先写清楚为什么改、改哪些 route/event/schema、core 和 desktop 怎么验收。
2. 用户确认后，先改 `nomi-protocol`。
3. core 依赖新协议完成实现。
4. 通知 desktop owner 升级协议依赖并接入。
5. core / desktop 都用正式 tag 依赖完成验证后，再视为协议联动完成。

固定禁止：

- 不主动修改 `/Users/lixinlv/Doing/nomi-desktop` 代码。
- 不绕过 desktop owner 改桌面端实现。
- 不把 desktop 私有补丁塞进 core。

## Current Product Facts

Nomi 当前是一个以 instance 为核心的个人 AI runtime。

正式运行模型：

```text
InstanceRuntime
  ├─ CLI
  ├─ RemoteAdapter       HTTP + SSE
  ├─ WeixinChannel
  ├─ AgentLoop
  ├─ TaskRunner / CronService
  ├─ SessionManager
  ├─ Provider
  ├─ ToolRegistry
  └─ InstanceChannel
```

当前统一口径：

- 一个 instance root 只允许一个 runtime 进程。
- `remote` 和 `channel` 是挂在 runtime 上的 adapter，不单独持有 runtime。
- `nomi instance start/run/stop/restart/status/log/services` 是后台进程管理入口。
- `nomi remote` 只负责 remote 配置、token 和状态。
- `nomi channel` 只负责 channel 配置、登录和状态。
- 自动任务默认是实例级全局提醒。
- `tasks.json` 是任务定义真源，`cron/jobs.json` 是派生调度状态。
- `TOOLS.md` 是 core 内置模板，不生成到每个 workspace。
- `OPERATIONS.md` 是根目录操作手册，方便部署和排障时直接查看。

## Repo Layout

核心目录：

- `nomi/cli/`
  - 命令入口、交互循环、终端渲染。
- `nomi/runtime/`
  - 统一 runtime 装配、adapter 挂载、service 生命周期。
- `nomi/agent/`
  - AgentLoop、上下文构建、工具循环、记忆、skills、prompt。
- `nomi/remote/`
  - HTTP API + SSE remote server。
- `nomi/channel/`
  - 微信等外部 channel adapter。
- `nomi/instance_channel/`
  - instance 好友关系、邀请码、relation token、实例间聊天。
- `nomi/tasks/`
  - 自动任务定义、执行、全局提醒队列。
- `nomi/cron/`
  - 派生时间触发器。
- `nomi/session/`
  - 会话模型和 JSONL 持久化。
- `nomi/providers/`
  - LLM provider 解析、注册和适配。
- `nomi/config/`
  - 配置模型、路径、实例 root 解析。
- `nomi/bus/`
  - runtime 内部 inbound / outbound 消息总线。
- `nomi/templates/`
  - 内置 prompt、TOOLS、workspace 模板。

文档入口：

- `README.md`
  - 项目介绍、快速开始和索引。
- `OPERATIONS.md`
  - 日常操作手册。
- `docs/`
  - 功能文档。
- `AGENTS.md`
  - 本文件，项目维护规则。
- `CLAUDE.md`
  - 源码讲解型 agent 提示词。

## Reading Order

进入任务前按这个顺序建立上下文：

1. `AGENTS.md`
2. `README.md`
3. `OPERATIONS.md`
4. `docs/README.md`
5. 对应模块文档
6. 代码
7. 测试

不要用历史对话替代代码事实。

## Environment Setup

推荐环境：

- Python `>=3.11`
- `uv`

安装依赖：

```bash
uv sync
```

安装开发依赖：

```bash
uv sync --extra dev
```

安装本地命令：

```bash
uv tool install -e . --force
nomi --version
```

## Runtime Paths

默认实例 root：

```text
~/.nomi
```

关键运行态：

- `~/.nomi/config.json`
- `~/.nomi/workspace`
- `~/.nomi/sessions`
- `~/.nomi/tasks`
- `~/.nomi/logs`
- `~/.nomi/weixin`
- `~/.nomi/instance-relations.json`
- `~/.nomi/instance-requests.json`
- `~/.nomi/instance-invite.json`

服务器上的 `~/.nomi` 默认视为用户运行态数据，不得整体删除、覆盖或重置。

## Cleanup Rules

切换分支或排查历史污染时，可以清理本机运行态，但必须保留配置和认证：

固定保留：

- `~/.nomi/config.json`
- `~/.nomi/weixin`
- 明确属于认证或登录态的文件

可清理：

- `~/.nomi/workspace`
- `~/.nomi/sessions`
- `~/.nomi/tasks`
- `~/.nomi/logs`
- `~/.nomi/skills`

清理前如果实例正在运行，先执行：

```bash
nomi instance stop
```

## Build And Test

语法检查：

```bash
uv run python -m compileall nomi tests -q
```

全部测试：

```bash
uv run python -m pytest -q
```

局部测试：

```bash
uv run python -m pytest tests/agent -q
uv run python -m pytest tests/cli -q
uv run python -m pytest tests/remote -q
uv run python -m pytest tests/instance_channel -q
uv run python -m pytest tests/tasks -q
uv run python -m pytest tests/tools -q
```

Ruff：

```bash
uv run ruff check nomi tests
```

## Documentation Rules

- `README.md` 只做项目介绍、快速开始和索引。
- `OPERATIONS.md` 放日常操作和排障命令。
- `docs/` 放正式功能文档。
- 文档只描述当前已落地能力，不写路线图、未落地设计或未实现能力。
- 改代码后同步更新相关文档。
- `docs/` 下源码链接必须以 `../` 开头，并精确到 `#Lstart-Lend`。
- 根目录文档源码链接必须以 `./` 开头，并精确到 `#Lstart-Lend`。

标准格式：

```text
描述 -> ../nomi/agent/loop.py#L87-L213
```

## Code Style

- 函数和方法需要清晰中文 docstring。
- docstring 说明参数和返回值。
- 变量命名清楚，避免无意义缩写。
- 注释使用中文。
- 注释解释“为什么”，不要解释显而易见的“做了什么”。
- 不添加不必要的类、封装和中间层。
- `__init__.py` 只保留最小导出。
- 改动以最小可维护解为准。

## Working Rules

允许：

- 修正文档和代码不同步。
- 清理结构冗余。
- 统一已有实现风格。
- 补齐已有能力缺少的测试。
- 基于真实运行结果解释系统行为。

禁止：

- 擅自改产品方向。
- 恢复已移除能力到默认链路。
- 增加过渡适配实现。
- 增加迁移脚本。
- 为未确认能力提前铺大框架。
- 没确认边界就跨模块大改。
- 用历史对话替代代码事实。

## Decision Priority

遇到冲突时按这个顺序判断：

1. 当前代码事实。
2. 本文件中的项目边界。
3. `README.md` / `OPERATIONS.md` / `docs/`。
4. 测试。
5. 历史对话。

## One-line Principle

**先相信代码事实，按规范做最小修改，把已有能力做清楚，不扩、不绕、不兼容。**

---
> Source: [YS-BW/nomi-core](https://github.com/YS-BW/nomi-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
