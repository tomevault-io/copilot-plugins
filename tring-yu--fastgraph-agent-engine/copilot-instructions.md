## fastgraph-agent-engine

> AI Agent 二次开发规范指南 (Developer Guide)

AI Agent 二次开发规范指南 (Developer Guide)

本文档旨在为二次开发本项目的开发者（包括人类开发者和 AI 编码助手，如 Cursor、Copilot 等）提供严格的架构与代码规范指南。

🎯 核心架构原则

本项目是一个生产就绪的 AI Agent 服务，技术栈组合如下：

核心大脑: LangGraph (支持状态图、记忆回溯、检查点机制)

Web 框架: FastAPI (异步处理，极高并发)

可观测性: Langfuse (链路追踪) + structlog (结构化日志) + Prometheus (指标采集)

持久化: PostgreSQL + pgvector (向量搜索) + SQLModel (ORM)

LLM 网关: 深度对接硅基流动 (SiliconFlow) 及自研的 Failover 高可用路由。

⚠️ 绝对禁忌 (Common Pitfalls to Avoid)

在修改本项目代码时，必须严格遵守以下红线：

❌ 禁止在 structlog 中使用 f-string：日志事件必须是固定的英文字符串（如 "user_login_success"），动态变量必须作为 kwargs 参数传递（如 user_id=123）。

❌ 禁止在函数或类内部 import：所有模块引用必须放置在文件顶部。

❌ 禁止遗漏 API 限流器：所有新增的 FastAPI 路由必须添加 @limiter.limit 装饰器以防 DDoS。

❌ 禁止绕过 Langfuse 追踪：所有对大模型（LLM）的调用必须挂载 Langfuse 回调，不得产生“隐形消耗”。

❌ 禁止使用 logger.error() 记录捕获的异常：在 except 块中捕获异常时，必须使用 logger.exception() 以保留完整的报错堆栈。

❌ 禁止同步阻塞代码：所有数据库访问、网络请求和文件 I/O 操作必须使用 async/await 异步语法。

❌ 禁止代码中硬编码 (Hardcode) 密钥：所有 API Keys 和敏感配置必须通过 app/core/config.py 从环境变量中读取。

🔧 编码最佳实践

依赖注入 (Dependency Injection): 充分利用 FastAPI 的 Depends 机制处理当前用户 (get_current_user) 和数据库会话。

模型容灾 (LLM Failover): 如果新增 LLM 调用逻辑，请优先使用 app.services.llm 中的 llm_service 单例，它内置了自动重试和模型崩溃切换功能。

工具开发 (Tool Building):

新增 Agent 工具必须继承 BaseTool 或使用 @tool 装饰器。

必须提供清晰、完整的 description（因为这是 LLM 决定是否调用该工具的唯一依据）。

必须定义强类型的 args_schema (Pydantic Model)。

防御性编程: 在函数开头优先校验参数、处理边缘异常，尽早 return 或 raise HTTPException。

📖 参考文档

LangGraph 官方文档

FastAPI 官方文档

SQLModel 官方文档

硅基流动 API 文档

---
> Source: [tring-yu/FastGraph-Agent-Engine](https://github.com/tring-yu/FastGraph-Agent-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
