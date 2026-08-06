## buy-me-a-coffee

> > 本文档旨在帮助大语言模型快速理解本项目的架构、代码组织和核心逻辑。

# AGENTS.md - AI 助手项目理解指南

> 本文档旨在帮助大语言模型快速理解本项目的架构、代码组织和核心逻辑。

## 项目概述

**Buy A Coffee** 是一个基于 Google ADK (Agent Development Kit) 的多 Agent 系统，实现智能咖啡订购和配送服务。项目展示了 A2A (Agent-to-Agent) 协作、工具调用 (Tool Calling)、多系统协同等 AI Agent 核心能力。

### 技术栈

| 层级 | 技术 | 用途 |
|------|------|------|
| AI 框架 | Google ADK | Agent 定义、工具绑定、多 Agent 协作 |
| A2A 协议 | A2A SDK | Agent 间通信协议 |
| 后端框架 | FastAPI | REST API、SSE 流式响应 |
| 数据库 | SQLite + aiosqlite | 异步数据持久化 |
| 前端框架 | React + TypeScript | 用户界面 |
| UI 组件 | CopilotKit | AI 聊天组件 |
| 样式 | TailwindCSS | 响应式设计 |

## 目录结构

```
buy-a-coffee/
├── backend/
│   ├── main.py              # 主入口（统一部署）
│   ├── run_all.py           # 分布式部署启动脚本
│   ├── config.py            # 全局配置
│   │
│   ├── shared/              # 共享模块
│   │   ├── __init__.py
│   │   ├── database.py      # 共享数据库基础设施
│   │   ├── http_client.py   # HTTP 客户端（工具调用后端 API）
│   │   └── utils.py         # 共享工具
│   │
│   ├── assistant/           # 助手服务
│   │   ├── __init__.py
│   │   ├── agent.py         # Assistant Agent（天气、时间、提醒）
│   │   └── tools.py         # 助手工具
│   │
│   ├── coffee/              # 希希咖啡服务 ☕
│   │   ├── __init__.py
│   │   ├── main.py          # 独立后端 API 入口 (端口 8001)
│   │   ├── a2a.py           # 独立 A2A Agent 入口 (端口 8003)
│   │   ├── agent.py         # Coffee Agent（含 order/query 子 Agent）
│   │   ├── api.py           # REST API 路由
│   │   ├── database.py      # 咖啡店数据库
│   │   └── tools.py         # 咖啡相关工具（通过 HTTP 调用后端）
│   │
│   ├── delivery/            # 送了么配送服务 🛵
│   │   ├── __init__.py
│   │   ├── main.py          # 独立后端 API 入口 (端口 8002)
│   │   ├── a2a.py           # 独立 A2A Agent 入口 (端口 8004)
│   │   ├── agent.py         # Delivery Agent
│   │   ├── api.py           # REST API 路由
│   │   ├── database.py      # 配送数据库
│   │   └── tools.py         # 配送相关工具（通过 HTTP 调用后端）
│   │
│   └── gateway/             # 网关服务 🌐
│       ├── __init__.py
│       ├── main.py          # 主网关入口 (端口 8000)
│       └── agent.py         # Root Agent（通过 A2A 调用子 Agent）
│
├── frontend/
│   └── src/
│       ├── components/      # React 组件
│       ├── App.tsx          # 主应用组件
│       └── main.tsx         # 入口文件
│
└── data/                    # SQLite 数据库文件（运行时生成）
```

## 部署模式

### 1. 统一部署（默认）

所有服务在同一个进程中运行，适合开发和演示：

```bash
cd backend
python main.py
```

服务启动后：
- API 文档: http://localhost:8000/docs
- 聊天接口: http://localhost:8000/api/chat/stream
- 咖啡 API: http://localhost:8000/api/coffee
- 配送 API: http://localhost:8000/api/delivery
- 咖啡 Agent Card: http://localhost:8000/agents/coffee/.well-known/agent.json
- 配送 Agent Card: http://localhost:8000/agents/delivery/.well-known/agent.json

### 2. 分布式部署

各服务独立运行，通过 A2A 协议通信，适合生产环境：

```bash
cd backend
python run_all.py
```

或手动启动各服务：

```bash
# 1. 启动咖啡店后端 API (端口 8001)
python -m coffee.main

# 2. 启动配送后端 API (端口 8002)
python -m delivery.main

# 3. 启动咖啡店 Agent A2A (端口 8003)
python -m coffee.a2a

# 4. 启动配送 Agent A2A (端口 8004)
python -m delivery.a2a

# 5. 启动主网关 (端口 8000)
DEPLOY_MODE=distributed python -m gateway.main
```

服务架构：

```
┌─────────────────────────────────────────────────────────────┐
│                     前端 (localhost:5173)                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 主网关 (localhost:8000)                      │
│                     Root Agent                               │
│                         │                                    │
│    ┌────────────────────┼────────────────────┐              │
│    │                    │                    │              │
│    ▼                    ▼                    ▼              │
│ Assistant           A2A 调用            A2A 调用           │
│  Agent                  │                    │              │
└─────────────────────────┼────────────────────┼──────────────┘
                          │                    │
          ┌───────────────┘                    └───────────────┐
          ▼                                                    ▼
┌─────────────────────┐                          ┌─────────────────────┐
│ 咖啡 Agent A2A      │                          │ 配送 Agent A2A      │
│ (localhost:8003)    │                          │ (localhost:8004)    │
│                     │                          │                     │
│ coffee_agent        │                          │ delivery_agent      │
│ ├── order_agent     │                          │                     │
│ └── query_agent     │                          │                     │
└─────────┬───────────┘                          └─────────┬───────────┘
          │ HTTP                                           │ HTTP
          ▼                                                ▼
┌─────────────────────┐                          ┌─────────────────────┐
│ 咖啡后端 API        │                          │ 配送后端 API        │
│ (localhost:8001)    │                          │ (localhost:8002)    │
│                     │                          │                     │
│ /api/coffee/*       │                          │ /api/delivery/*     │
└─────────────────────┘                          └─────────────────────┘
```

## A2A 架构

### 核心设计

本项目的核心特性是使用 **A2A (Agent-to-Agent) 协议** 实现 Agent 间通信：

1. **咖啡 Agent** 和 **配送 Agent** 作为 A2A 服务暴露
2. **Root Agent** 通过 A2A 协议远程调用这些服务
3. 每个 A2A 服务都有标准的 Agent Card

### Agent 层级关系

```
root_agent (根 Agent) [gateway/agent.py]
├── assistant_agent (助手 Agent) [assistant/agent.py] - 本地调用
│   └── 工具: 天气查询、时间管理、提醒、日程
│
├── coffee_agent (咖啡服务 Agent) [coffee/agent.py] - A2A 远程调用
│   ├── order_agent (下单 Agent)
│   │   └── 工具: 获取菜单、搜索商品、创建订单
│   └── query_agent (查询 Agent)
│       └── 工具: 查询订单、获取订单列表、更新状态
│
└── delivery_agent (配送 Agent) [delivery/agent.py] - A2A 远程调用
    └── 工具: 创建配送、查询配送、更新配送状态
```

### 工具调用流程

Agent 的工具通过 HTTP 调用后端 API，而不是直接访问数据库：

```
Agent 工具 (coffee/tools.py)
    │
    │ HTTP 请求
    ▼
后端 API (coffee/api.py)
    │
    │ 数据库操作
    ▼
数据库 (coffee/database.py)
```

这种设计的优点：
1. Agent 和后端解耦，可以独立部署
2. 后端 API 可以被其他系统调用
3. 便于测试和调试

## 关键实现细节

### 1. A2A 服务注册

在 `gateway/main.py` 中，咖啡和配送 Agent 被注册为 A2A 服务：

```python
from a2a.server.apps import A2AStarletteApplication
from google.adk.a2a.utils.agent_card_builder import AgentCardBuilder
from google.adk.a2a.executor.a2a_agent_executor import A2aAgentExecutor

async def build_a2a_starlette_app(agent, base_path: str):
    # 创建 Agent Card
    card_builder = AgentCardBuilder(agent=agent, rpc_url=rpc_url)
    agent_card = await card_builder.build()
    
    # 创建 A2A 应用
    a2a_app = A2AStarletteApplication(
        agent_card=agent_card,
        http_handler=request_handler,
    )
    
    return a2a_app
```

### 2. Root Agent 通过 A2A 调用子 Agent

在 `gateway/agent.py` 中，使用 `RemoteA2aAgent` 调用远程服务：

```python
from google.adk.agents.remote_a2a_agent import RemoteA2aAgent

remote_coffee = RemoteA2aAgent(
    name="coffee_agent",
    agent_card="http://localhost:8003/.well-known/agent.json",
)

remote_delivery = RemoteA2aAgent(
    name="delivery_agent", 
    agent_card="http://localhost:8004/.well-known/agent.json",
)

root_agent = Agent(
    name="root_agent",
    sub_agents=[assistant_agent, remote_coffee, remote_delivery],
)
```

### 3. HTTP 工具调用

工具通过 HTTP 调用后端 API，使用线程池避免阻塞：

```python
from shared.http_client import get_coffee_api_client

def get_menu(category: str = None) -> dict:
    """获取菜单"""
    client = get_coffee_api_client()
    response = client.get("/api/coffee/products", params={"category": category})
    return response
```

### 4. Agent 定义模式

每个 Agent 遵循以下定义模式：

```python
from google.adk import Agent

agent = Agent(
    name="agent_name",           # Agent 标识符
    model="gemini-2.0-flash",    # 使用的 LLM 模型
    description="...",           # Agent 功能描述（用于路由）
    instruction="...",           # 系统提示词（定义行为）
    tools=[...],                 # 可用工具列表
    sub_agents=[...],            # 子 Agent 列表（可选）
)
```

## API 端点

### 网关 API (端口 8000)

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/chat` | POST | 非流式聊天 |
| `/api/chat/stream` | POST | SSE 流式聊天 |
| `/api/copilotkit` | POST | CopilotKit 兼容接口 |

### 咖啡店 API (端口 8001 或 8000)

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/coffee/products` | GET | 获取商品列表 |
| `/api/coffee/products/{id}` | GET | 获取商品详情 |
| `/api/coffee/orders` | POST | 创建订单 |
| `/api/coffee/orders` | GET | 获取订单列表 |
| `/api/coffee/orders/{id}` | GET | 获取订单详情 |
| `/api/coffee/orders/{id}/status` | PUT | 更新订单状态 |

### 配送 API (端口 8002 或 8000)

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/delivery/deliveries` | POST | 创建配送 |
| `/api/delivery/deliveries` | GET | 获取配送列表 |
| `/api/delivery/deliveries/{id}` | GET | 获取配送详情 |
| `/api/delivery/deliveries/order/{order_id}` | GET | 按订单查配送 |
| `/api/delivery/deliveries/{id}/status` | PUT | 更新配送状态 |

## 环境变量

| 变量 | 必需 | 默认值 | 描述 |
|------|------|--------|------|
| `DEPLOY_MODE` | ❌ | unified | 部署模式：unified/distributed |
| `GATEWAY_PORT` | ❌ | 8000 | 主网关端口 |
| `COFFEE_API_PORT` | ❌ | 8001 | 咖啡店后端端口 |
| `DELIVERY_API_PORT` | ❌ | 8002 | 配送后端端口 |
| `COFFEE_A2A_PORT` | ❌ | 8003 | 咖啡店 A2A 端口 |
| `DELIVERY_A2A_PORT` | ❌ | 8004 | 配送 A2A 端口 |
| `COFFEE_API_URL` | ❌ | http://localhost:8001 | 咖啡店后端 URL |
| `DELIVERY_API_URL` | ❌ | http://localhost:8002 | 配送后端 URL |
| `COFFEE_A2A_URL` | ❌ | http://localhost:8003 | 咖啡店 A2A URL |
| `DELIVERY_A2A_URL` | ❌ | http://localhost:8004 | 配送 A2A URL |
| `GOOGLE_API_KEY` | ✅* | - | Google AI API 密钥 |
| `GOOGLE_MODEL` | ❌ | gemini-2.0-flash | 模型名称 |
| `API_HOST` | ❌ | 0.0.0.0 | 监听地址 |
| `FRONTEND_URL` | ❌ | http://localhost:5173 | 前端 URL |

*如果使用 AgentRun 集成则不需要

## 调试技巧

1. **查看 Agent Card**: 访问 `/.well-known/agent.json` 查看 A2A 服务描述
2. **查看 Agent 路由**: 观察 SSE 响应中的 `tool_call` 和 `agent_transfer` 事件
3. **数据库调试**: 使用 SQLite 客户端查看 `data/*.db`
4. **API 测试**: 访问 `/docs` 使用 Swagger UI
5. **日志查看**: 查看服务器日志中的 🔧 (工具调用) 和 🌐 (HTTP 请求) 标记

---

*本文档由 AI 生成，旨在帮助大语言模型理解项目结构和实现逻辑。*

---
> Source: [agentrun-ai/buy-me-a-coffee](https://github.com/agentrun-ai/buy-me-a-coffee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
