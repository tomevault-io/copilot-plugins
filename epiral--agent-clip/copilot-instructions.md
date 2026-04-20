## agent-clip

> AI Agent Clip — agentic loop with tool use, memory, and vision.

# AGENTS.md — agent-clip

AI Agent Clip — agentic loop with tool use, memory, and vision.

## 数据位置

所有数据在 `~/.pinix/data/agent/` 下：

| 数据 | 位置 | 用途 |
|---|---|---|
| 数据库 | `agent.db` | topics、messages、runs、summaries、events |
| 配置 | `config.yaml` | LLM provider、system prompt、pinned clips |
| 话题文件 | `{topic-id}/` | 每个话题的附件、截图等 |
| 异步 run 输出 | `runs/{run-id}.jsonl` | 异步运行的 JSONL 输出 |
| 记忆文件 | `schema.sql` | DB schema 定义 |

## 日志

| 日志 | 位置 | 内容 | 格式 |
|---|---|---|---|
| agent 运行日志 | `~/.pinix/logs/agent.log` | LLM 请求/响应、tool call 错误、run_id 关联 | JSON（每行一个 JSON 对象） |
| pinixd 系统日志 | `~/.pinix/logs/pinixd.log` | invoke 路由、clip 启停、trace_id | JSON（slog） |

### 查询日志

```bash
# 查看最近的 agent 日志
tail -20 ~/.pinix/logs/agent.log

# 按 run_id 查完整调用链
grep "run_id_here" ~/.pinix/logs/agent.log

# 只看错误
grep "run.error\|parse_error" ~/.pinix/logs/agent.log

# 查看 LLM 请求/响应
grep "llm\." ~/.pinix/logs/agent.log
```

### 日志事件

| event | 触发时机 | 关键字段 |
|---|---|---|
| `llm.request` | 每次调用 LLM | model, messages, tools |
| `llm.response` | LLM 返回 | content_length, tool_calls |
| `run.error` | tool call 执行失败 | command, stdin_length, error |
| `run.parse_error` | tool call JSON 解析失败 | tool, raw_length, raw_preview |

## 查询数据库

```bash
# 查看所有话题
sqlite3 ~/.pinix/data/agent/agent.db "SELECT id, name FROM topics ORDER BY created_at DESC LIMIT 10"

# 查看某话题的消息
sqlite3 ~/.pinix/data/agent/agent.db "SELECT role, substr(content,1,100) FROM messages WHERE topic_id='xxx' ORDER BY id"

# 查看活跃的 run
sqlite3 ~/.pinix/data/agent/agent.db "SELECT id, topic_id, status FROM runs WHERE status='running'"

# 查看 tool call 错误模式
sqlite3 ~/.pinix/data/agent/agent.db "SELECT substr(content,1,200) FROM messages WHERE role='tool' AND content LIKE '%error%' ORDER BY id DESC LIMIT 10"
```

## 配置

`config.yaml` 关键字段：

```yaml
name: agent-name
llm_provider: openrouter          # provider 名称
llm_model: anthropic/claude-opus-4.6
max_tokens: 65536                  # 可选，不设则用模型默认

providers:
  openrouter:
    base_url: https://openrouter.ai/api/v1
    api_key: sk-or-v1-xxx
    provider:                       # OpenRouter 路由控制
      ignore: [amazon-bedrock]      # 排除不稳定的 provider

pinned: [fs, browser]              # 固定注入 system prompt 的 clips
system_prompt: |
  your prompt here
```

## 开发

- 运行时：Bun
- UI：`ui/` 目录，React + Tailwind + Vite
- 构建 UI：`cd ui && npx vite build`（输出到 `web/`）
- 本地开发：`bun run index.ts --web 9007`（独立 HTTP 模式，需要代理环境变量访问 OpenRouter）
- 类型检查：`cd ui && bun run typecheck`

## 架构

```
index.ts          — Clip 入口，注册所有 command
src/commands.ts   — 命令分发（send、create-topic、delete-topic 等）
src/loop.ts       — agentic loop（callLLM → executeToolCall → 循环）
src/llm.ts        — LLM API 调用（OpenAI 协议 + Anthropic 协议）
src/tools.ts      — 命令注册、run tool 定义、命令链式执行
src/context.ts    — system prompt 组装（clip manifests、recall、environment）
src/config.ts     — config.yaml 读写
src/db.ts         — SQLite 操作
src/log.ts        — 结构化 JSON 日志
src/memory.ts     — 语义搜索、embedding、摘要
src/output.ts     — 输出格式（raw、JSONL、SSE）
```

---
> Source: [epiral/agent-clip](https://github.com/epiral/agent-clip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
