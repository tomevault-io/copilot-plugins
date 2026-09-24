## minicode

> MiniCode 是一个轻量级终端 AI 编程 Agent。命令行入口是 `minicoder`

# MiniCode

MiniCode 是一个轻量级终端 AI 编程 Agent。命令行入口是 `minicoder`
和 `minicode`，两者都会启动同一个交互式 REPL。

## 项目结构

- `minicoder/cli.py`：命令行入口和交互模式，处理 `/help`、`/save`、
  `/sessions`、`/compact` 等内置命令。
- `minicoder/agent.py`：核心 Agent 循环，负责调用模型、执行工具并继续
  多轮工具调用。
- `minicoder/llm.py`：OpenAI-compatible 流式接口封装，支持工具调用、
  token 统计和简单重试。
- `minicoder/tools/`：内置工具，包括 shell、读写文件、精确编辑、搜索、
  子 Agent、计划更新和技能读取。
- `minicoder/context.py`：上下文压缩逻辑，避免长对话超过模型上下文。
- `minicoder/session.py`：会话保存和恢复，数据保存在
  `~/.minicoder/sessions`。
- `minicoder/tools/skill.py` 和 `minicoder/hooks/`：发现本地技能，并在用户提问
  时把可用技能摘要注入系统提示。

## 常用命令

```bash
uv run minicoder
uv run minicode
uv run minicoder -m <model>
uv run minicoder -r <session-id>
uv run minicoder --version
```

运行测试：

```bash
uv run pytest tests/ -v
```

不要直接使用全局 `pytest`，它可能缺少当前项目依赖。

## 配置

配置来自环境变量或项目目录上层的 `.env` 文件，命令行参数优先级最高。

- `MINICODE_API_KEY`：API Key，必填。
- `MINICODE_MODEL`：模型名，默认 `deepseek-v4-flash`。
- `MINICODE_BASE_URL`：API 地址，默认 `https://api.deepseek.com`。
- `MINICODE_MAX_TOKENS`：单次生成上限，默认 `8192`。
- `MINICODE_TEMPERATURE`：温度，默认 `1.0`。
- `MINICODE_MAX_CONTEXT`：上下文压缩阈值，默认 `1048576`。

## 开发注意

- 一切设计以MVP为主，不引入复杂结构。
- 小改动优先保持现有简单结构，不要引入过重抽象。
- 修改工具、CLI 行为、Hook、Session 或上下文压缩时，同步更新相关测试。
- 新增工具时继承 `Tool`，实现 `name`、`description`、`parameters` 和
  `execute`，再注册到 `minicoder/tools/__init__.py`。

---
> Source: [xu2023-ICT/MiniCode](https://github.com/xu2023-ICT/MiniCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
