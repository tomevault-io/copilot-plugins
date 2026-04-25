## claude-brain

> 个人异步任务自动化系统 — Notion 输入 → Brain 调度 → Claude Code 执行。v2 新增飞书实时对话能力。

# Claude Brain

个人异步任务自动化系统 — Notion 输入 → Brain 调度 → Claude Code 执行。v2 新增飞书实时对话能力。

## 项目架构

系统有两套并行运行的工作流：

- **v1（Notion 任务流）**：Notion 写需求 → Brain 轮询分发 → CC 子进程执行 → 结果写回 Notion
- **v2（飞书对话流）**：飞书发消息 → Brain 接收 → CC SDK 执行 → 结果回飞书

两套工作流共享同一个 asyncio 事件循环、同一个 SQLite 数据库、同一套日志体系。

职责边界：Brain 管"什么时候做、做完了告诉谁" + 记忆管理；CC 管"怎么做"。

## 关键文件

### v1（Notion 任务流）

| 文件 | 用途 |
|---|---|
| `brain/core/dispatcher.py` | 任务分发（workspace 准备 → inbox 构建 → CC 启动） |
| `brain/core/watchdog.py` | 超时检测 + 进程健康检查 |
| `brain/core/outbox.py` | outbox.json 轮询与结果处理 |
| `brain/core/protocol.py` | inbox/outbox JSON 格式定义 |
| `brain/core/process.py` | CC 子进程启动 |
| `brain/integrations/notion.py` | Notion REST API 客户端 |
| `brain/workspace/manager.py` | workspace git clone/pull/init |
| `brain/workspace/setup.py` | 模板安装 + 上下文注入 |
| `templates/` | CC 角色模板（planner/、executor/、shared/） |

### v2（飞书对话流）

| 文件 | 用途 |
|---|---|
| `brain/channels/base.py` | Channel 抽象接口 + 标准消息格式 |
| `brain/channels/feishu/adapter.py` | 飞书 WebSocket adapter |
| `brain/channels/feishu/client.py` | 飞书 API 客户端（发送/回复/编辑消息） |
| `brain/executor/cc.py` | Claude Agent SDK 封装（query + resume） |
| `brain/session/manager.py` | Session 生命周期（channel→session 映射 + 过期管理） |
| `brain/memory/store.py` | 记忆 SQLite CRUD |
| `brain/memory/retriever.py` | 记忆检索 + context 组装 |
| `brain/memory/extractor.py` | 从 CC 输出提取记忆（Phase A: 规则匹配） |

### 共享

| 文件 | 用途 |
|---|---|
| `brain/main.py` | asyncio 主循环，编排 v1 轮询 + v2 channel adapter |
| `brain/config.py` | 配置加载，导出 CONFIG 和派生常量 |
| `brain/infra/db.py` | SQLite schema、连接工厂（v1 + v2 表） |
| `brain/infra/logger.py` | 分类日志初始化（4 个 logger） |
| `config.yaml` | 运行时配置 |
| `state.db` | SQLite 运行时状态 |

## 分层架构

```
config.py              ← 无 brain 内部依赖（基础层）
    ↑
infra/                 ← 只依赖 config（基础设施层）
integrations/          ← 只依赖 config（外部服务层）
    ↑
workspace/             ← 依赖 config、infra/logger（workspace 层）
core/protocol.py       ← 无依赖（纯数据格式）
channels/base.py       ← 无 brain 内部依赖（接口定义）
    ↑
core/dispatcher.py     ← 依赖 infra + integrations + workspace + protocol
core/watchdog.py       ← 依赖 infra + integrations
core/outbox.py         ← 依赖 infra + integrations + protocol
channels/feishu/       ← 依赖 channels/base + infra/logger
executor/              ← 依赖 infra/logger
session/               ← 依赖 config + infra/logger
memory/                ← 依赖 infra/logger
    ↑
main.py                ← 依赖 core + channels + executor + session + memory
```

约束：v1 模块（core/、integrations/、workspace/）和 v2 模块（channels/、executor/、session/、memory/）互不依赖，只在 main.py 编排层汇合。

## 技术栈

- Python 3.12+
- SQLite（状态管理）
- PyYAML（配置解析）
- requests（Notion REST API 调用）
- lark-oapi（飞书 SDK，WebSocket 长连接 + 消息 API）
- claude-agent-sdk（Claude Code SDK，v2 执行层）
- Claude Code CLI（`claude --print`，v1 执行层）
- Notion MCP（`@notionhq/notion-mcp-server`，Planner CC 使用）

## v1: CC 角色与权限

权限通过 CLI 参数 `--allowedTools` / `--disallowedTools` 硬性控制，不依赖 prompt 约束。配置集中在 `config.yaml` 的 `roles` 字段。

- **Planner CC**：有 Notion 写权限，无 Bash；负责需求拆解
- **Executor CC**：有完整文件和 Shell 工具，无 Notion 权限；负责代码实现

## v1: 通信协议

Brain 与 CC 通过 workspace 中的 JSON 文件通信：
- `inbox.json`：Brain 写入完整任务上下文，CC 读取
- `outbox.json`：CC 写入执行结果，Brain 轮询读取
- Status token：`TASK_DONE` / `TASK_BLOCKED` / `TASK_PROGRESS`

## v2: 消息流

```
飞书消息 → FeishuAdapter → _dispatch_message()
  ├─ /help /reset /status  → 即时响应（不进队列）
  ├─ /btw <task>           → 即时回复 + 后台 CC（不阻塞队列）
  └─ 普通消息               → per-channel 队列 → _handle_chat()
       → add emoji reaction（思考指示器）
       → 获取 per-channel workspace
       → 查找/创建 session（自动 resume）
       → 组装记忆 context
       → executor.cc.execute()（Claude Agent SDK）
       → 回复 markdown 卡片
       → remove reaction
       → 提取记忆存入 DB
```

## v2: 记忆系统

Brain-owned 记忆系统，独立于 CC 的 per-project 记忆。

- 存储：SQLite `memories` 表
- 检索：关键词匹配 content + tags，按 importance + recency 排序
- 注入：通过 `--append-system-prompt` 将相关记忆注入 CC context
- 提取：Phase A 用规则匹配从 CC 输出提取事实（后续升级为 LLM 提取或 MCP tools）

## 远程开发模式

通过 `config.yaml` 的 `remote` 配置段启用，支持通过 Tailscale 等组网方案从远程设备访问。

## 开发规范

- Brain 是确定性调度器，不包含业务推理逻辑
- v1: 同 project 串行，跨 project 并行；v2: per-channel 串行处理
- 最大并发 CC 进程数由 `config.yaml` 的 `scheduler.max_concurrent` 控制
- v1 和 v2 模块互不依赖，只在 main.py 汇合
- Channel adapter 是纯 I/O 层，不含业务逻辑
- CC 不知道 Brain 的存在（workspace + system prompt + 消息）

## 质量规则（本项目开发时遵守）

### 编码流程

1. **编码前**：了解现有代码结构，确认改动范围。如果项目有测试，先跑一次确认基线通过
2. **编码中**：每完成一个独立功能点，运行相关测试确认无回归
3. **编码后**：运行完整测试（如有），确认全部通过
4. **提交前**：检查 git diff，确认没有遗漏文件或调试代码

### 执行策略（TDD 优先）

项目有测试框架时，**必须**先写测试再写实现：

- **新功能**：
  1. 先写测试，描述期望行为（测试应失败）
  2. 实现功能，让测试通过
  3. 清理、重构（测试仍通过）
- **Bug 修复**：
  1. 先写复现测试，锁定当前错误行为
  2. 修复 bug，复现测试变为通过
  3. 确认其他测试无回归
- **重构**：
  1. 先确认现有测试全部通过
  2. 重构代码
  3. 再次确认测试全部通过，不改变外部行为

项目没有测试框架时（如本项目当前状态），用以下替代验证：
- `uv run ruff check` — lint 检查
- `uv run python -c "from brain.xxx import yyy"` — import 验证
- 架构层边界检查 — v1/v2 不交叉 import

### 提交规范

- 格式：`type(scope): description`（Conventional Commits）
- 每个独立改动一个 commit，不混合不相关变更
- 改动涉及新功能/API 变更时，同步更新相关文档（README、CLAUDE.md）

### 版本号规则（必须遵守）

每次功能新增或重要 bug 修复后，**必须**更新版本号：
1. 修改 `pyproject.toml` 中的 `version`
2. 运行 `uv sync` 更新 `uv.lock`
3. 将 pyproject.toml + uv.lock 一起提交

版本格式：`0.major.patch`
- patch +1：bug 修复、小改动
- major +1：新功能、行为变更

不更新版本号 = 其他设备无法通过 `uv tool upgrade ccbrain` 拉到最新代码。

### 验证要求（必须执行，不是建议）

每次改动后，**必须**完成以下验证再提交：
1. `uv run python -c "from brain.xxx import yyy"` — 确认 import 无报错
2. 功能验证 — 实际运行改动涉及的功能路径
3. CLI 命令改动 — 逐个测试受影响的子命令
4. 新增配置项 — 同步更新 `config.example.yaml` 和 `brain/data/config.example.yaml`
5. 飞书相关改动 — `ccbrain restart` 后在飞书发消息验证

如果跳过验证直接提交，后续在其他设备或 launchd 环境中会出问题（import 错误、运行时崩溃等）。

### 上下文恢复

如果对话历史不完整（context compaction），请：
1. 阅读本文件了解项目架构
2. `git log --oneline -20` 查看近期改动
3. 查看 `docs/dev-status.md` 了解当前状态和待办
4. 继续未完成的工作，不要重做已完成的部分

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linvie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
