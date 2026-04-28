## wenexus

> **Monorepo 结构**：全栈应用，包含 Java 后端（Spring Boot 微服务）、Python 后端（FastAPI + AI/ML）和 Next.js/React Native 前端。

# CLAUDE.md

项目开发指南和 AI 辅助编程最佳实践。

## 应用架构

**Monorepo 结构**：全栈应用，包含 Java 后端（Spring Boot 微服务）、Python 后端（FastAPI + AI/ML）和 Next.js/React Native 前端。

### 核心概念

- **技术方案文档**：`docs/technical` - 架构决策和实现方案
- **前端共享包**：`frontend/packages` - UI 组件、类型定义、工具函数
- **后端服务**：`backend/java` + `backend/python` - 微服务架构

### 目录结构

```
wenexus/
├── frontend/                     # 前端 Monorepo (pnpm + Turborepo)
│   ├── apps/
│   │   ├── web/                 # 主 Web 应用 (Next.js)
│   │   │   └── src/components/agent/  # Agent 聊天 UI 组件
│   │   ├── admin/               # 管理后台 (Next.js)
│   │   └── mobile/              # 移动端 (React Native)
│   └── packages/
│       ├── ui/                  # 共享 UI 组件
│       ├── types/               # TypeScript 类型定义
│       ├── utils/               # 工具函数
│       └── shared/              # 通用 hooks 和配置
│
├── backend/                      # 后端服务
│   ├── java/                    # Java 微服务 (Spring Boot + Maven)
│   │   ├── core-service/        # 核心业务逻辑
│   │   ├── user-service/        # 用户管理
│   │   ├── content-service/     # 内容管理
│   │   └── consensus-service/   # 共识算法
│   └── python/                  # Python 服务 (FastAPI + AI/ML, Python 3.14)
│       └── src/
│           ├── facade/          # API 网关层
│           │   └── langgraph_api/  # LangGraph 兼容 API (assistants/threads/runs)
│           ├── app/             # 应用层
│           │   ├── agent_config.py    # Agent 配置模型
│           │   ├── agent_pool.py      # Agent 实例池（生命周期管理）
│           │   ├── agent_registry.py  # Agent 注册表（静态配置）
│           │   └── agent_initializer.py  # Agent 启动编排
│           ├── service/         # 领域服务层
│           │   ├── agent/       # Agent 实现（fact_checker/facilitator）
│           │   ├── langgraph_api/  # LangGraph 运行时服务
│           │   └── research_agent/  # 研究 Agent
│           ├── repository/      # 数据持久层
│           ├── model/           # 数据模型
│           └── util/            # 工具
│               ├── logging_config.py  # 统一结构化日志（SLS 兼容）
│               └── llm_callback.py    # LLM 调用日志回调
│
└── docs/                         # 关于项目的各种文档，有不清楚问题时优先查阅文档
    └── technical/               # 技术方案和架构决策
```

### 架构原则

- **分层清晰**：
  - `facade` → `app` → `service` → `repository`
  - 依赖方向单向，不反向依赖
- **配置与实例分离**：配置管理 what to create，实例池管理 when to create
- **Agent 生命周期管理**：类似 Spring 的 Bean 生命周期
  - `AgentRegistry`: 静态配置注册表（BeanDefinitionRegistry）
  - `AgentPool`: 实例池管理 singleton/prototype 作用域
  - `AgentInitializer`: 应用启动编排
- **契约先行**：API 实现必须严格遵循类型定义
- **LangGraph 兼容 API**：自托管 LangGraph API，支持 assistants/threads/runs

## 常用开发命令

### 本地开发启动

```bash
# 一键启动（推荐）
./scripts/dev.sh           # 全栈：数据库 + 后端 + Tunnel + 前端 Workers 预览
./scripts/dev.sh --reload  # 全栈（开启 Python 热重载）
./scripts/dev.sh frontend  # 仅前端 Workers 预览（http://localhost:8787）
./scripts/dev.sh stop      # 停止所有
```

前端使用 Cloudflare Workers 预览模式（`pnpm cf:preview`），模拟线上运行时，Secrets 读取自 `frontend/apps/web/.dev.vars`。

> 后端已设置 LaunchAgent 开机自启（无 `--reload`）。开发时用 `dev.sh` 或手动启动带 `--reload` 的版本。

### 日志配置

**日志位置**：

- 应用日志：`logs/app.jsonl`（JSON Lines 格式）
- 错误日志：`logs/error.jsonl`
- dev.sh 脚本日志：`scripts/logs/frontend.log`, `scripts/logs/python.log`

```bash
# 环境变量
LOG_JSON=true|false              # 控制台输出格式（生产环境设为 true）
LOG_LEVEL=DEBUG|INFO|WARNING|ERROR
LOG_FILE_PATH=logs/app.jsonl     # JSON 日志文件路径
SERVICE_NAME=wenexus             # 服务名称（用于日志标识）
```

### 代码质量

```bash
pre-commit run --all-files       # 运行所有 pre-commit hooks
```

## 开发原则

### 核心理念

- **程序 = 算法 + 数据结构**：编程的核心是控制数据的流动
- **高内聚、低耦合**：短期代码必须能跑，长期人能看懂
- **单一数据源原则（SSOT）**：避免在多处维护相同信息

### 代码质量标准

- **拒绝硬编码**：使用常量、枚举或配置文件
- **拒绝过度设计**：只解决当前问题，不预设未来需求
- **拒绝 mock 测试**：优先集成测试和端到端测试
- **函数长度限制**：单个函数不超过 30 行
- **命名清晰**：函数名、变量名要自解释，减少注释需求
- **使用 StrEnum**：Python 3.11+ 使用 `StrEnum` 替代 `(str, Enum)`

### 测试策略

**代码测试与 AI 测试分层**：

| 层级 | 方式 | 适用场景 |
|------|------|----------|
| CI/CD | Playwright 代码 | 关键路径回归测试，需要快、稳、可复现 |
| 探索性 | 自然语言用例 (`e2e/cases/*.yaml`) | AI 执行，发现边界问题，验证 UX |

- **关键路径**用代码测试保证稳定性（注册、登录、支付等）
- **探索性测试**用自然语言描述，让 AI 理解并执行
- E2E 测试放在独立包 `frontend/packages/e2e/`，不与应用代码混合

### 必须完成端到端验证

代码开发后，必须自己完成完整验证，包括：

1. **启动应用**：使用实际启动命令，不能只写代码
2. **检查启动日志**：确保无错误、无警告
3. **验证接口功能**：
   - 使用 curl 或实际前端测试 API
   - 验证返回数据结构符合预期
   - 测试正常流程和异常流程
4. **确认集成正确**：检查日志中的相关信息
5. **只有全部通过后才交给用户最终验证**

**推荐 TDD 方式**：先写测试，再实现功能，确保端到端验证。

**禁止行为**：

- 写完代码就认为任务完成
- 依赖用户进行基本测试
- 假设代码能运行而不实际验证

### 安全问题处理原则

- **只修复重大安全漏洞**：除非是可被直接利用的重大漏洞（如 SQL 注入、RCE），否则不主动处理安全问题
- **开发阶段优先功能**：当前阶段以功能开发和迭代为主，非重大安全问题不阻塞开发进度
- **不做预防性安全加固**：不主动添加安全相关的限制性配置（如图片域名白名单、CSP 头等），除非有明确的安全事件驱动

### 代码内文档要求

文档离代码越近越好，docstring 优于独立 md 文件。

**检查清单**（每次提交前）：

- [ ] commit 对应的技术文档已创建或更新
- [ ] 新模块有顶部注释说明用途和依赖关系
- [ ] 变更的模块注释仍然准确
- [ ] `package.json` 的 `description` 与实际功能一致
- [ ] 没有创建任何非必要的文档文件

#### 禁止行为

- 禁止创建总结性文档（如 xxx-summary.md、xxx-overview.md、xxx-guide.md）
- 禁止在 commit 中附带"顺便更新了文档"的无关改动
- 禁止创建与已有文档内容重复的新文件

#### 允许行为

- 修改代码时，同步更新同文件内的注释/docstring
- 创建 commit 对应的技术文档（按上述格式）
- 用户明确要求时，更新指定的文档文件

## 部署架构与流程

### 当前架构

```
用户 → Cloudflare Workers（前端 Next.js via OpenNext）
           │  PYTHON_BACKEND_URL
           ▼
     Cloudflare Tunnel → 本地机器 :8000（FastAPI）
                              ├── Supabase PostgreSQL（云数据库）
                              └── Upstash Redis（云缓存）
```

| 层 | 服务 | 说明 |
|----|------|------|
| 前端 | Cloudflare Workers | Next.js via OpenNext，已开通 Paid 计划 |
| 后端 | 本地机器 + Cloudflare Tunnel | FastAPI (Python 3.14)，固定域名 `https://api.aispeeds.me` |
| 数据库 | Supabase (Transaction Pooler) | 生产用；本地开发连 `localhost:5432` |
| CI/CD | GitHub Actions | `develop` → staging；`main` → production |

### Agent 系统

**后端 Agent 架构**：

- **Fact Checker Agent**: 事实核查深度 Agent，使用 LangGraph 运行时
- **Research Agent**: 研究 Agent，支持 critique 和 research 子 Agent
- **LangGraph API**: 自托管 LangGraph 兼容 API，支持 assistants/threads/runs

**前端 Agent 界面**：

- 访问 `/agent` 路径使用 Agent 聊天界面
- 使用 LangGraph SDK 与后端通信
- 支持流式响应和工具调用可视化

### 推送到线上

```
# 前端（自动）
git push origin develop   →  CI/CD 自动部署 → staging
develop → main PR 合并    →  CI/CD 自动部署 → production

# 后端（手动，运行在本机）
# 改完代码重启服务即生效，git push 不影响后端
launchctl stop com.wenexus.backend && launchctl start com.wenexus.backend
```

### 环境变量

| 文件 | 用途 |
|------|------|
| `frontend/apps/web/.dev.vars` | 本地 Workers 预览的 Secrets（`DATABASE_URL`、`AUTH_SECRET`、`PYTHON_BACKEND_URL`） |
| `backend/python/.env.development` | Python 后端本地配置（含 `LOG_JSON`, `LOG_LEVEL`, `SERVICE_NAME`） |
| `backend/python/.env.production.example` | 生产环境配置模板 |
| Cloudflare Workers Secrets | 生产/staging 的敏感变量，通过 CI/CD 注入 |
| GitHub Environment Secrets | CI/CD 用（`CLOUDFLARE_API_TOKEN` 等） |

> 详细部署文档 → `docs/technical/deployment/`
> 日志配置文档 → `docs/technical/logs/logging-sls-setup.md`

---

## 最重要的原则

**有任何不清楚的，随时提问。**

不确定时宁愿多问一句，也不要基于假设实现错误的方案。

---
> Source: [ournexus/wenexus](https://github.com/ournexus/wenexus) — distributed by [TomeVault](https://tomevault.io/claim/ournexus).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
