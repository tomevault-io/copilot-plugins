## ai-tool-functioncall

> > 最后校准：2026-07-29。本文只保留当前代码库的协作约束，不记录历史进度。历史设计、审计与修复过程统一位于 [`docs/archive/`](docs/archive/README.md)。

# Gateway 项目协作说明

> 最后校准：2026-07-29。本文只保留当前代码库的协作约束，不记录历史进度。历史设计、审计与修复过程统一位于 [`docs/archive/`](docs/archive/README.md)。

## 项目定位

本仓库实现一个位于三方 LLM API 与 coding-agent / SDK 客户端之间的中游 Gateway，主要负责：

- OpenAI Chat、OpenAI Responses、Anthropic Messages 协议转换；
- 弱上游工具调用适配、工具归属分流与多轮编排；
- Gateway-owned 工具、MCP、HTTP Actions 与内置工具 runtime；
- 长上下文压缩、SQLite 记忆、fan-out、缓存和持久化；
- Admin UI、9-Tab Config Center、运行状态与部署入口；
- Assistants / Threads、Web2API、多上游和请求前 Intelligence。

运行时能力以 `GET /capabilities`、真实请求行为和当前测试为准。文件名、注释、历史文档或 helper 存在，不等于能力已接入生产请求路径。

## 所有权边界

```text
上游 Provider
  负责模型推理，以及其明确声明并实际支持的原生 tool/function-call 能力

Gateway（本仓库）
  负责协议转换、认证、路由、adapter/orchestration、Gateway-owned 工具、
  MCP/HTTP Action、缓存、持久化、统计、Admin/Config 和安全边界

下游 Claude Code / Codex / SDK
  负责用户机器上的 Read/Write/Edit/Bash/Skill/GUI/local-agent 等工具执行
```

原则：可以 adapter，不能伪造成功。用户侧工具默认下发给客户端；只有显式开启 `gateway.execute_user_side_tools_in_gateway=true` 并满足对应权限开关时，才允许本地代理式执行。

## 当前权威文档

| 目标 | 文档 |
|---|---|
| 项目定位、快速开始、公开 API | [`README.md`](README.md) |
| 全部文档导航与权威顺序 | [`docs/README.md`](docs/README.md) |
| 当前实现状态与边界 | [`docs/IMPLEMENTATION_STATUS.md`](docs/IMPLEMENTATION_STATUS.md) |
| 系统边界与模块结构 | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) |
| 本地运行、配置、测试和排障 | [`docs/RUNNING_AND_TESTING.md`](docs/RUNNING_AND_TESTING.md) |
| 生产部署 | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) |
| 历史审计、设计和进度快照 | [`docs/archive/README.md`](docs/archive/README.md) |

文档冲突时按以下顺序判断：

1. live runtime / HTTP 行为；
2. 当前网络请求与响应；
3. 当前配置和已部署资源；
4. 当前源码与测试；
5. 历史审计、设计和进度记录。

## 主要代码入口

| 范围 | 主要文件 |
|---|---|
| 进程入口与应用装配 | `src/toolcall_gateway.py`, `src/gateway_app.py` |
| HTTP 路由、认证和安全 | `src/gateway_http_handler.py`, `src/gateway_http_auth.py`, `src/gateway_http_security.py` |
| 配置与管理面 | `src/gateway_config.py`, `src/gateway_admin.py`, `src/gateway_admin_api.py`, `src/gateway_web_config.py` |
| 协议和上游代理 | `src/gateway_protocol.py`, `src/gateway_proxy.py`, `src/gateway_upstream_pool.py` |
| 工具与 Agent Planner | `src/gateway_tool_runtime.py`, `src/gateway_builtin_tools.py`, `src/gateway_agent_planner.py` |
| 流式与上下文 | `src/gateway_streaming.py`, `src/gateway_context.py`, `src/gateway_stream_state.py` |
| 缓存、持久化和统计 | `src/gateway_cache.py`, `src/gateway_persistence.py`, `src/gateway_logging.py`, `src/gateway_stats.py` |
| 扩展能力 | `src/gateway_mcp.py`, `src/gateway_http_actions.py`, `src/gateway_assistants.py`, `src/gateway_web2api.py` |
| 沙箱与进程边界 | `src/gateway_sandbox.py`, `src/gateway_sandbox_worker.py`, `src/gateway_process_ops.py` |

`src/gateway_apply_patch.py` 和 `src/gateway_sandbox_worker.py` 是独立 worker/CLI 入口，即使没有普通模块 import，也不能按“零引用文件”删除。

## 修改规则

1. 优先修改当前生产调用链，不以历史文档推翻已观察到的运行行为。
2. 删除文件前必须确认没有源码、测试、启动脚本、Docker/Compose、CI 或文档入口依赖。
3. 不提交 `.gateway_service.json`、`.gateway_runtime/`、`gateway_log.sqlite3`、`.case.txt`、`.env*`、证书、密钥或 trace。
4. 不清理用户现有运行数据库、请求历史或本地服务配置；测试使用临时目录和独立端口。
5. 公开能力、配置字段、工具数量等数字必须从当前 registry/schema 或门禁输出重新生成。
6. 外部 Provider 只有在配置 `TEST_UPSTREAM_URL` 和有效凭据并执行 live E2E 后，才能写成“实时验证通过”。
7. 历史资料只放 `docs/archive/`；根目录只保留 `README.md` 和本文件两个 Markdown 入口。
8. 代码变化必须带回归测试；修复后先跑相关测试，再跑完整门禁。

## 测试与发布门禁

```bash
# 相关测试
python3 -m pytest -q <affected tests>

# 全量 pytest
unset GATEWAY_AGENT_PLANNER_STRICT_EVERY_TURN
python3 -m pytest -ra tests

# Agent Planner 完整验收
./scripts/agent_planner_acceptance.sh --full

# 发布级完整门禁：静态、安全、依赖、测试、配置、Compose、Docker
./scripts/ci_gate.sh
```

提交前至少执行：

```bash
python3 -m compileall -q src tests
python3 -m ruff check src tests
git diff --check
bash -n scripts/*.sh
```

涉及配置中心时，必须让 Playwright/Chromium 回归真实执行；CI 使用 `GATEWAY_REQUIRE_BROWSER_TESTS=1` 禁止静默 skip。

## 当前结构性契约

以下数字由当前代码 registry/schema 生成，不应手工猜测：

- 公开路径：24；
- 内置工具：67 个唯一 canonical 工具；
- 内置工具 registry：178 个含别名 key；
- Config Center：9 个标签页、89 个字段。

仓库整洁性由 `tests/test_repository_hygiene.py` 约束：Admin renderer 只能有一个权威定义，根目录文档只能保留两个入口，退休文件不能回流，本地 Markdown 链接必须可解析，代码围栏必须闭合。

---
> Source: [hhhaiai/ai_tool_functioncall](https://github.com/hhhaiai/ai_tool_functioncall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
