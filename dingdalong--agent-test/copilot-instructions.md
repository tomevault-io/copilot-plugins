## agent-test

> 从零构建的 Python AI Agent 框架，端口-适配器架构。

# AI Agent Framework

从零构建的 Python AI Agent 框架，端口-适配器架构。

## 技术栈

- Python 3.13，uv 管理依赖
- LLM：DeepSeek（OpenAI 兼容接口），通过 LLMProvider Protocol 可替换
- 向量存储：ChromaDB + Ollama embedding（qwen3-embedding:0.6b）
- 异步：asyncio 全链路

## 架构分层

- **Layer 0**（无外部依赖）：`src/config.py`, `src/utils/`, `src/interfaces/`, `src/graph/`, `src/guardrails/`
- **Layer 1**（依赖 Layer 0）：`src/llm/`, `src/tools/`
- **Layer 2**（依赖 Layer 0-1）：`src/memory/`, `src/agents/`, `src/plan/`, `src/mcp/`, `src/skills/`
- **Layer 3**（应用层，组装所有具体实现）：`src/app/`

低层模块不得导入高层模块。

## 关键约定

- 所有可插拔组件使用 Protocol 接口：`LLMProvider`, `MemoryProvider`, `ToolProvider`, `GraphNode`, `UserInterface`
- 具体实现只在 `src/app/bootstrap.py` 中实例化
- 工具通过 `@tool` 装饰器注册，支持中间件管道（错误处理、敏感确认、截断）
- 配置集中在 `config.yaml`（用户配置）+ `.env`（密钥）

## 开发规范

- 每次开始任务时，需要先检查是否有相应的文档
- 如果文档与代码逻辑出现冲突，以代码实现为准
- 代码需要有简介的注释，对于复杂算法可以详细一些
- 每次完成任务后，同步更新相关文档和测试用例
- 类型安全：所有函数签名必须有类型注解，使用 Protocol 而非具体类型，善用 TypedDict / dataclass / Pydantic model 定义数据结构
- 避免使用已弃用方法

## 常用命令

```bash
uv sync                    # 安装依赖
uv run python main.py      # 启动 agent
uv run pytest              # 运行测试
uv run pytest -m slow      # 运行慢速/集成测试
uv run python -m src.tools.classify          # 工具分类
uv run python -m src.tools.classify --force   # 强制重分类
```

## 当前状态

- weather / calendar / email 专家智能体的工具是占位符（`get_weather`, `create_event`, `send_email` 未注册）
- 参见 `TODO` 文件了解未来计划

## 文件导航

- 入口：`main.py` → `src/app/bootstrap.py` → `src/app/app.py`
- 核心接口：`src/llm/base.py`（LLMProvider）、`src/memory/base.py`（MemoryProvider）、`src/tools/router.py`（ToolProvider）、`src/graph/types.py`（GraphNode）、`src/interfaces/base.py`（UserInterface）
- 智能体：`src/agents/agent.py`（Agent 模型）、`src/agents/runner.py`（工具循环）、`src/app/presets.py`（预设定义）
- 图引擎：`src/graph/engine.py`（GraphEngine）、`src/graph/builder.py`（GraphBuilder）
- 规划：`src/plan/flow.py`（PlanFlow 5 阶段）、`src/plan/compiler.py`（Plan → CompiledGraph）
- 记忆：`src/memory/chroma/store.py`（ChromaDB 存储）、`src/memory/extractor.py`（事实提取）、`src/memory/decay.py`（重要性衰减）
- 工具分类：`src/tools/categories.py`（CategoryResolver + 配置加载）、`src/tools/classifier.py`（分类流水线）、`src/tools/classify.py`（CLI 入口）、`src/tools/delegate.py`（DelegateToolProvider）
- 配置：`config.yaml`、`.env`

---
> Source: [dingdalong/agent_test](https://github.com/dingdalong/agent_test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
