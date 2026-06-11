## bottleneckhunter

> BottleneckHunter 是一个 AI 驱动的产业链瓶颈选股系统。核心方法论来自 Serenity 的"三步法"：

# BottleneckHunter - CLAUDE.md

## 项目概述

BottleneckHunter 是一个 AI 驱动的产业链瓶颈选股系统。核心方法论来自 Serenity 的"三步法"：
1. **产业链拆解** — 从终端产品逐层拆解到原材料，3+ 层深度
2. **供应商检索** — 沿瓶颈环节找到被忽视的优质供应商
3. **交叉验证** — 多个 LLM 从反面角度拷问投资逻辑

## 技术栈

- Python 3.10+
- LangGraph / LangChain — 工作流编排
- Pydantic v2 — 数据模型（所有模型定义在 `chain/models.py`）
- Rich / Questionary / Typer — CLI
- yfinance / akshare — 市场数据

## 项目结构

```
bottleneck_hunter/
├── cli.py              # Typer CLI 入口
├── chain/
│   ├── models.py       # 所有 Pydantic 数据模型
│   ├── decomposer.py   # LLM 产业链拆解引擎
│   ├── bottleneck.py   # 瓶颈评分算法
│   ├── graph.py        # LangGraph 筛选工作流
│   ├── report.py       # Markdown 报告生成
│   ├── prompts/        # LLM 提示词模板 (.md)
│   └── data/           # 预设产业链 JSON
└── llm_clients/
    └── factory.py      # LLM 客户端工厂
```

## 开发规范

- 中文注释和提示词（面向中文用户）
- 代码风格：ruff, line-length=120
- 异步优先：所有 LLM 调用使用 `async/await`
- Pydantic 模型做数据验证，不使用裸 dict
- LLM 返回 JSON 时，做好 markdown code fence 剥离和容错解析

## 环境配置

复制 `.env.example` 为 `.env`，填入 API key。支持 provider: openai, anthropic, deepseek, google, qwen, glm, ollama, openrouter

## 运行

```bash
pip install -e .
bottleneck-hunter screen
```

## 开发计划

详见 `PLAN.md`。Phase 1 骨架已完成，下一步是 Phase 2（供应商检索与评估）。

---
> Source: [Acclerate/BottleneckHunter](https://github.com/Acclerate/BottleneckHunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
