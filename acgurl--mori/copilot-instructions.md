## mori

> - 每个模型类型必须配对特定的 Formatter（见 [`mori.py:_create_model()`](../../mori/mori.py:127)）

# Architect Mode - 项目架构规则（仅非显而易见内容）

## 架构约束

### 模型与 Formatter 强耦合
- 每个模型类型必须配对特定的 Formatter（见 [`mori.py:_create_model()`](../../mori/mori.py:127)）
- 映射关系：`openai` → `OpenAIChatFormatter`, `dashscope` → `DashScopeChatFormatter` 等
- 不能混用模型和 Formatter

### 长期记忆架构限制
- Mem0 不支持流式响应，必须使用 [`NonStreamingModelWrapper`](../../mori/utils/model_wrapper.py:9) 包装主模型
- 向量维度必须在多处保持一致：嵌入模型的 `dimensions` → `VectorStoreConfig` 的 `embedding_model_dims`
- 长期记忆在 `agent_control` 模式下会自动注册工具，影响 Agent 的工具集

### 配置系统设计
- 配置文件分离设计：模型配置、Agent 配置、全局配置各自独立
- Agent 通过名称引用模型，而非直接嵌入模型配置（间接引用模式）
- 环境变量解析在配置加载时完成，不在运行时解析

### 模板系统设计
- 双层查找机制：自定义模板优先于内置模板
- 运行时信息注入在代码层面，不在模板层面（关注点分离）

---
> Source: [acgurl/mori](https://github.com/acgurl/mori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
