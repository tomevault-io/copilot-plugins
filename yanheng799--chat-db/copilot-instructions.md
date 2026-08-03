## chat-db

> Chat-DB 项目的 agent 入口文件。适用于 Claude Code、Copilot、Cursor 等任何 AI 编码 agent。

# AGENTS.md

Chat-DB 项目的 agent 入口文件。适用于 Claude Code、Copilot、Cursor 等任何 AI 编码 agent。

## 项目一句话

中文自然语言 → 多 Agent 协作 → 只读 SQL → 返回结果。Python + LangGraph。

## 开始工作前必读

1. **[CLAUDE.md](CLAUDE.md)** — 项目总入口（技术栈、命令、源码布局）
2. **[docs/agent-harness/index.md](docs/agent-harness/index.md)** — harness 文档索引（根据任务类型选择必读文档）

## 命令速查

```bash
# 安装依赖
python -m uv sync --extra dev --index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 测试
.venv/Scripts/python.exe -m pytest

# Lint + 格式化
.venv/Scripts/ruff.exe check --fix src/ && .venv/Scripts/ruff.exe format src/
```

## 关键约束

- **禁止占位符**：代码中不允许 `TODO`、`pass`、`NotImplementedError`、`...`
- **TDD**：先写测试（Red）→ 最小实现（Green）→ 重构
- **设计原则**：OCP（扩展不改旧代码）、DIP（依赖抽象）、SRP（单一职责）
- **配置**：所有配置通过 `.env`，禁止 YAML
- **新增包**：必须在 `pyproject.toml` 的 `packages` 列表中注册

## 遇到问题时

1. 查阅 [known-failures.md](docs/agent-harness/known-failures.md)
2. 查阅 [commands.md](docs/agent-harness/commands.md) 中的常见失败原因
3. 新问题记录到 [known-failures.md](docs/agent-harness/known-failures.md)
4. 环境或工具缺口记录到 [harness-debt.md](docs/agent-harness/harness-debt.md)

---
> Source: [yanheng799/chat-db](https://github.com/yanheng799/chat-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
