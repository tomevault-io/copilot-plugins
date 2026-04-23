## whaleclaw

> - 运行时: Python 3.12 (项目内嵌独立 Python，路径 `./python/bin/python3.12`)

# WhaleClaw — 项目编码指南

- 项目名: WhaleClaw
- 仓库根目录: 当前目录
- 运行时: Python 3.12 (项目内嵌独立 Python，路径 `./python/bin/python3.12`)
- 包管理: pip + pyproject.toml
- Web 框架: FastAPI + uvicorn
- 异步运行时: asyncio

## 项目结构

```
whaleclaw/                    # Python 包根目录
  __init__.py
  entry.py                    # 主入口 (uvicorn 启动)
  version.py                  # 版本号
  config/                     # 配置管理 (Pydantic v2 模型)
    __init__.py
    schema.py                 # 配置 schema 定义
    loader.py                 # 配置加载/合并/校验
    paths.py                  # 路径常量 (~/.whaleclaw/)
  gateway/                    # Gateway WebSocket 控制面
    __init__.py
    app.py                    # FastAPI 应用工厂
    ws.py                     # WebSocket 协议处理
    protocol.py               # WS 消息协议定义
    middleware.py              # 中间件 (认证/CORS/日志)
  agent/                      # Agent 运行时
    __init__.py
    loop.py                   # Agent 主循环 (消息 -> LLM -> 工具 -> 回复)
    context.py                # Agent 上下文 (会话状态/工具注册)
    prompt.py                 # PromptAssembler 分层提示词组装器 (静态/动态/延迟三层)
  providers/                  # LLM 模型提供商适配层
    __init__.py
    base.py                   # Provider 抽象基类
    anthropic.py              # Anthropic Claude
    openai.py                 # OpenAI GPT
    deepseek.py               # DeepSeek
    qwen.py                   # 通义千问
    zhipu.py                  # 智谱 GLM (glm-4.7 / glm-5)
    minimax.py                # MiniMax (海螺 AI)
    moonshot.py               # 月之暗面 Kimi
    google.py                 # Google Gemini
    nvidia.py                 # NVIDIA NIM (免费模型)
    router.py                 # 模型路由 + failover
  channels/                   # 消息渠道
    __init__.py
    base.py                   # Channel 抽象基类/协议
    webchat/                  # WebChat 渠道
      __init__.py
      handler.py
    feishu/                   # 飞书渠道
      __init__.py
      bot.py                  # 飞书 Bot 事件处理
      client.py               # 飞书 API 客户端
      card.py                 # 卡片消息构建
      media.py                # 媒体上传下载
      webhook.py              # Webhook 安全验证
      tools/                  # 飞书工具集
        docx.py               # 文档操作
        wiki.py               # 知识库操作
        drive.py              # 云盘操作
        bitable.py            # 多维表格操作
        perm.py               # 权限管理
  routing/                    # 消息路由
    __init__.py
    router.py                 # 路由引擎
    rules.py                  # 路由规则
  sessions/                   # 会话管理
    __init__.py
    manager.py                # 会话生命周期管理
    store.py                  # 会话持久化 (SQLite)
    context_window.py         # 上下文窗口管理
  tools/                      # 内置工具
    __init__.py
    base.py                   # Tool 抽象基类
    registry.py               # 工具注册表
    bash.py                   # Bash 命令执行
    file_read.py              # 文件读取
    file_write.py             # 文件写入
    file_edit.py              # 文件编辑
    browser.py                # 浏览器控制 (Playwright)
  plugins/                    # 插件系统
    __init__.py
    sdk.py                    # 插件 SDK (WhaleclawPluginApi)
    loader.py                 # 插件发现/加载
    registry.py               # 插件注册表
    evomap/                   # EvoMap 内置插件 (GEP-A2A 协作进化市场)
      __init__.py
      plugin.py               # EvoMap 插件主入口
      client.py               # A2A 协议客户端 (hello/publish/fetch)
      identity.py             # 节点身份管理 (sender_id 持久化)
      publisher.py            # 资产打包发布 (Gene + Capsule + EvolutionEvent)
      fetcher.py              # 资产拉取 + 本地缓存
      bounty.py               # 赏金任务 (claim/complete)
      hasher.py               # SHA256 content-addressable ID 计算
      models.py               # Pydantic 数据模型 (Gene/Capsule/EvolutionEvent/Task)
      config.py               # EvoMap 配置 (hub_url/webhook/sync_interval)
  skills/                     # 技能系统
    __init__.py
    manager.py                # 技能发现/安装/管理
    parser.py                 # SKILL.md 解析 (含 YAML frontmatter)
    router.py                 # SkillRouter 按需路由 (关键词匹配，不调用 LLM)
    summary.py                # AGENTS.md 摘要生成器
  memory/                     # 记忆系统
    __init__.py
    base.py                   # Memory 抽象基类
    vector.py                 # 轻量记忆存储 (JSON 持久化 + 关键词检索)
    manager.py                # 记忆编排 (Recall/Capture/Organizer/全局风格)
    summary.py                # 自动摘要
  media/                      # 媒体处理
    __init__.py
    pipeline.py               # 媒体处理管道
    transcribe.py             # 音频转文字
    vision.py                 # 图片理解
  security/                   # 安全
    __init__.py
    pairing.py                # DM 配对机制
    sandbox.py                # Docker 沙箱
    auth.py                   # 认证 (Token/密码)
  cli/                        # CLI 命令
    __init__.py
    main.py                   # Typer 应用入口
    gateway_cmd.py            # gateway 子命令
    agent_cmd.py              # agent 子命令
    message_cmd.py            # message 子命令
    config_cmd.py             # config 子命令
    doctor_cmd.py             # doctor 诊断命令
  tui/                        # 终端 UI
    __init__.py
    progress.py               # 进度条/spinner (Rich)
    table.py                  # 表格输出
  web/                        # WebChat 前端静态资源
    static/
  utils/                      # 通用工具
    __init__.py
    log.py                    # structlog 日志配置
    async_helpers.py          # 异步工具函数
    text.py                   # 文本处理
  types.py                    # 全局类型定义
tests/                        # 测试目录 (与 src 结构镜像)
  conftest.py
  test_config/
  test_gateway/
  test_agent/
  test_providers/
  test_channels/
  test_tools/
  test_sessions/
docs/
  specs/                      # 各阶段功能 SPEC
```

## 技术栈

| 领域 | 选型 | 说明 |
|------|------|------|
| Web 框架 | FastAPI + uvicorn | 异步 HTTP + WebSocket |
| 数据校验 | Pydantic v2 | 配置/协议/API schema |
| CLI | Typer + Rich | 命令行界面 + 终端美化 |
| 数据库 | SQLite (aiosqlite) | 会话/记忆持久化 |
| 日志 | structlog | 结构化日志 |
| 测试 | pytest + pytest-asyncio | 单元/集成/E2E 测试 |
| HTTP 客户端 | httpx | 异步 HTTP 请求 (调用 LLM API) |
| WebSocket | websockets | WS 客户端/服务端 |
| 记忆存储 | SimpleMemoryStore (JSON) | 当前默认记忆后端，可扩展到向量库 |
| 浏览器控制 | Playwright | 浏览器工具 (Phase 5) |
| 前端 | Vue 3 + Vite | WebChat SPA (Phase 3) |
| 桌面应用 | Tauri | 桌面端包装 (Phase 8) |

## 编码规范

### 语言与风格

- 语言: Python 3.12+，全面使用类型注解
- 格式化: Ruff (formatter + linter)，运行 `ruff check` + `ruff format` 保持一致
- 类型检查: pyright strict 模式
- 禁止 `# type: ignore` 除非有充分理由并附注释说明
- 禁止 `Any` 类型，修复根因而非绕过
- 异步优先: 所有 I/O 操作使用 `async/await`，不在异步上下文中使用阻塞调用

### 命名约定

- 文件名: `snake_case.py`
- 类名: `PascalCase`
- 函数/方法: `snake_case`
- 常量: `UPPER_SNAKE_CASE`
- 私有成员: `_leading_underscore`
- 模块级别导出: 通过 `__all__` 显式声明
- 产品名: **WhaleClaw** (文档/UI 标题)；`whaleclaw` (CLI 命令/包名/配置键/路径)

### 文件组织

- 单个文件不超过 500 行；超过时拆分为子模块
- 测试文件与源文件同名，前缀 `test_`，放在 `tests/` 对应目录
- 每个包必须有 `__init__.py`，通过它控制公开 API
- 配置/常量集中在 `config/` 目录，不散落在业务代码中

### 注释原则

- 只为非显而易见的逻辑添加注释
- 不写叙述性注释 (如 "导入模块"、"定义函数")
- 复杂算法/业务逻辑/权衡取舍必须注释
- 公开 API 必须有 docstring (Google 风格)

### 错误处理

- 自定义异常继承自 `WhaleclawError` 基类
- 不捕获裸 `Exception`，捕获具体异常类型
- 面向用户的错误信息用中文，内部日志用英文
- 工具执行失败返回结构化错误，不抛异常到 Agent 循环外

### 依赖管理

- 所有依赖在 `pyproject.toml` 的 `[project.dependencies]` 中声明，锁定主版本
- 开发依赖放 `[project.optional-dependencies.dev]`
- 不手动编辑 `requirements.txt`，由 `pip-compile` 生成
- 新增依赖前确认是否有更轻量的替代方案

### 提示词优化原则 (与 OpenClaw 的核心区别)

WhaleClaw 采用分层按需加载架构，严格控制每轮 system prompt 的 token 消耗:

- **不全量注入**: 禁止将 AGENTS.md / TOOLS.md / 全部 SKILL.md 塞入 system prompt
- **工具走原生参数**: 工具 JSON Schema 通过 LLM API 的 `tools` 参数传递，不占 prompt token
- **技能按需路由**: SkillRouter 根据用户消息关键词匹配 0~2 个技能，不匹配则不注入
- **记忆有预算**: `MemoryManager.recall(max_tokens=N)` 严格控制记忆注入量
- **全局风格常驻**: 从长期记忆提炼的 `style_directive` 以低 token 成本每轮注入，且用户本轮明确要求优先
- **AGENTS.md 摘要化**: 仅首轮注入精简摘要 (~500 tokens)，后续轮次不重复
- **预算分配**: PromptAssembler 根据模型 max_context 自动分配各层 token 预算
- **缓存优化**: 静态层标记 `cache_control` 支持 Anthropic prompt caching

目标: 每轮 system prompt 控制在 ~1200 tokens (OpenClaw 约 8000+)

## 配置系统

- 用户配置文件: `~/.whaleclaw/whaleclaw.json`
- EvoMap 数据: `~/.whaleclaw/evomap/` (节点身份/资产缓存/同步状态)
- 凭证存储: `~/.whaleclaw/credentials/`
- 会话数据: `~/.whaleclaw/sessions/`
- 工作区: `~/.whaleclaw/workspace/`
- 插件目录: `~/.whaleclaw/plugins/`
- Agent 摘要: `~/.whaleclaw/workspace/AGENTS.summary.md` (自动生成)
- 技能目录: `~/.whaleclaw/workspace/skills/`
- 记忆文件: `~/.whaleclaw/memory/memory.json`
- 日志目录: `~/.whaleclaw/logs/`

配置优先级 (高 -> 低):
1. 命令行参数
2. 环境变量 (`WHALECLAW_` 前缀)
3. 用户配置文件 (`~/.whaleclaw/whaleclaw.json`)
4. 项目配置文件 (`./whaleclaw.json`)
5. 默认值

## Gateway WebSocket 协议

消息格式: JSON over WebSocket

```json
{
  "type": "message|tool_call|tool_result|stream|error|ping|pong",
  "id": "unique-message-id",
  "session_id": "session-uuid",
  "payload": { ... },
  "timestamp": "2026-02-22T12:00:00Z"
}
```

核心消息类型:
- `message`: 用户/Agent 文本消息
- `tool_call`: Agent 发起工具调用
- `tool_result`: 工具执行结果
- `stream`: 流式文本片段
- `error`: 错误通知
- `ping`/`pong`: 心跳

## CLI 命令结构

```
whaleclaw
  gateway
    run [--port PORT] [--bind HOST] [--verbose]    # 启动 Gateway
    --install-daemon                                 # 安装为系统服务
  agent
    --message TEXT [--thinking LEVEL] [--model MODEL]  # 单次 Agent 调用
  message
    send --to TARGET --message TEXT                  # 发送消息
  config
    set KEY VALUE                                    # 设置配置
    get KEY                                          # 读取配置
  channels
    login                                            # 渠道登录
    status [--probe]                                 # 渠道状态
  doctor                                             # 诊断检查
  onboard                                            # 引导设置向导
```

## 测试规范

- 框架: pytest + pytest-asyncio
- 覆盖率目标: 70% (lines/branches/functions)
- 命名: `test_<module>.py` 中的 `test_<function_name>` 或 `TestClassName`
- 异步测试: 使用 `@pytest.mark.asyncio` 装饰器
- 运行: `pytest` (全部) / `pytest tests/test_xxx/` (单模块) / `pytest --cov` (覆盖率)
- Mock: 使用 `unittest.mock` 或 `pytest-mock`，优先 per-instance mock
- 不在测试中使用真实 API key，除非标记为 `@pytest.mark.live`
- E2E 测试文件以 `_e2e` 后缀标识

## 提交规范

- 提交信息格式: `<scope>: <description>` (如 `gateway: add websocket heartbeat`)
- scope 取值: `gateway`, `agent`, `config`, `cli`, `channels`, `tools`, `sessions`, `plugins`, `skills`, `memory`, `media`, `security`, `evomap`, `docs`, `tests`, `deps`, `ci`
- 一个 PR 只解决一个问题/功能
- 不提交含有真实密钥/手机号/个人信息的代码

## 安全原则

- 不提交或发布真实凭证 (API key, token, 手机号)
- 测试/文档中使用明显的假数据占位
- 默认 DM 策略: `pairing` (未知发送者需配对码)
- 工具执行: 主会话信任执行，非主会话走沙箱
- 飞书 Webhook: 必须验证签名
- WebChat: 必须启用认证 (Token 或密码)

## 阶段开发指南

本项目分 8 个阶段推进，每个阶段有独立的 SPEC 文件:

1. **Phase 1** `docs/specs/phase1-foundation.md` — 基座 (配置 + Gateway + 基础 Agent)
2. **Phase 2** `docs/specs/phase2-sessions-tools.md` — 会话 + 多模型 + 工具
3. **Phase 3** `docs/specs/phase3-webchat.md` — WebChat 渠道
4. **Phase 4** `docs/specs/phase4-feishu.md` — 飞书渠道
5. **Phase 5** `docs/specs/phase5-plugins-skills.md` — 插件 + 技能系统
6. **Phase 6** `docs/specs/phase6-security-routing.md` — 安全 + 多 Agent 路由
7. **Phase 7** `docs/specs/phase7-memory-advanced.md` — 记忆 + 高级功能
8. **Phase 8** `docs/specs/phase8-apps-ops.md` — 桌面/移动端 + 运维

开发每个阶段前，先阅读对应 SPEC 文件，按 SPEC 中的验收标准逐项完成。每个阶段完成后运行完整测试套件确认无回归。

## 运行项目

```bash
# 使用项目内嵌 Python
./python/bin/python3.12 -m pip install -e ".[dev]"
./python/bin/python3.12 -m whaleclaw gateway run --port 18666 --verbose

# 或通过 CLI 入口
./python/bin/python3.12 -m whaleclaw --help
```

## Agent 专用备注

- 使用项目内嵌 Python: `./python/bin/python3.12`，不要使用系统 Python
- pip 安装: `./python/bin/pip3.12 install ...`
- 终端代理: 如需访问外网，设置 `export https_proxy=http://127.0.0.1:7897`
- 不要编辑 `python/` 目录下的任何文件
- 不要编辑 `node_modules/` (如果存在)
- 当回答问题时，先在代码中验证，不要猜测

---
> Source: [flywhale-666/whaleclaw](https://github.com/flywhale-666/whaleclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
