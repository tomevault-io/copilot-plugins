## liverag

> 这个仓库是 LiveRAG：基于 LiveKit 的实时语音个人知识库助手。主链路只使用在线服务：火山 STT、语音 LLM、MiniMax TTS，以及 `liverag/rag/` 封装的 LightRAG Core Service。

# AGENTS.md

## 项目定位

这个仓库是 LiveRAG：基于 LiveKit 的实时语音个人知识库助手。主链路只使用在线服务：火山 STT、语音 LLM、MiniMax TTS，以及 `liverag/rag/` 封装的 LightRAG Core Service。

## 当前架构

- 顶层应用包：`liverag/`
- 统一入口：`liverag/main.py`
- 实时语音模块：`liverag/agent/`
- 上下文、SOUL、history 和知识库概览模块：`liverag/context/`
- 单知识库隔离 RAG 服务模块：`liverag/rag/`
- 前端管理 API：`liverag/api/`
- 全局配置：`liverag/config/`
- 全局日志：`liverag/logging/`
- 运行目录和状态：`liverag/runtime/`

## 代码规范

- Python 包、模块和文件使用小写加下划线命名。
- 所有新增注释、docstring、文档和 `.env.example` 注释使用中文。
- `liverag/agent/` 只放语音 Agent 相关代码，不放全局配置、日志、API 或 RAG server。
- 不做旧入口兼容；不要恢复 `src/`、`agent_Local.py`、`agent_Online.py`、`knowledge_rules.py` 或 local STT/TTS 服务目录。
- 不要恢复 `memory.md`、`MemoryUpdater`、`context_bootstrap.md`、`knowledge_scope.md` 或 `rag_tool_mode=always`。
- LiveKit `AgentSession` 的 STT、LLM、TTS、VAD、打断和 endpointing 参数属于时延调优项，修改时必须说明原因。
- 用户数据不能写入项目目录，统一使用 `~/.LiveRAG/`。
- 前端只对接 `liverag/api/` 暴露的单端口管理 API，不直接依赖内部 `liverag/rag/` 的 `/v1/*` 路径。
- 知识库使用单库物理隔离模型：每个 `knowledge_base` 对应独立 LightRAG workspace；语音通话只允许锁定一个 `kb_id`，不要实现多库 fan-out、后过滤或 `kb_ids` 多选查询。

## 运行命令

```bash
uv run python -m liverag.main dev
```

单独启动 RAG 服务：

```bash
uv run liverag-rag-service
```

启动前端管理 API：

```bash
uv run liverag-api
```

## 验证命令

```bash
uv run ruff check liverag
uv run python -m compileall liverag
```

---
> Source: [YS-BW/LiveRAG](https://github.com/YS-BW/LiveRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
