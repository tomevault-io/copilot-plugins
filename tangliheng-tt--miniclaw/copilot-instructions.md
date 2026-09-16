## miniclaw

> - Miniclaw 是自托管多渠道开发协作 Agent 系统，基于 Python、FastAPI、DeepAgents 与 LangGraph。

# Miniclaw · 开发协作规范

## 项目
- Miniclaw 是自托管多渠道开发协作 Agent 系统，基于 Python、FastAPI、DeepAgents 与 LangGraph。
- `agent/` 负责 Agent 构建，`channels/` 负责渠道接入，`api/` 提供 HTTP 接口。
- `sandbox/` 负责隔离执行，`tasks/` 负责任务编排，`app/` 包含订单报价业务，`tests/` 存放测试。

## 实施要求
- 修改前阅读相关代码、接口与配置，保持现有部署兼容。
- 金额使用 Decimal，禁止用 float 做货币运算。
- 所有公开函数带类型注解和 docstring。
- 任何代码改动都必须附带对应的 pytest 测试。
- 不编造检索结果、执行结果或测试通过记录。
- 密钥通过环境变量配置，不写入源码、日志或沙箱。

## 风格
- 内部使用 snake_case；现有外部 API 字段保持兼容。
- 提交信息用祈使句，一行概述，可附必要正文。

---
> Source: [Tangliheng-tt/Miniclaw](https://github.com/Tangliheng-tt/Miniclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
