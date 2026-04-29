## mori

> This file provides guidance to agents when working with code in this repository.

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## 项目特定约定

### 模板系统优先级
- 模板查找顺序：`config/template/` (自定义) > `mori/template/internal_template/` (内置)
- 简短名称（如 `mori`）自动添加 `.jinja2` 扩展名并按优先级查找
- 运行时信息（`current_time`, `current_date`）在 [`mori.py:_load_system_prompt()`](mori/mori.py:117) 中注入，不在模板中硬编码

### 配置加载机制
- 环境变量格式：`${ENV_VAR_NAME}` 在 [`config.py:resolve_env_var()`](mori/config.py:27) 中解析
- 配置文件必须分离：`models.yaml` (模型), `agents.yaml` (agent), `config.yaml` (全局)
- Agent 通过 `model` 字段引用 `models.yaml` 中的模型名称，不是直接配置

### Model 和 Formatter 配对
- 每个模型类型必须配对对应的 Formatter（见 [`mori.py:_create_model()`](mori/mori.py:127)）
- OpenAI 兼容接口使用 `model_type: openai` + `base_url` 参数
- `client_args` 用于传递额外的客户端参数（如 `base_url`）

### 工具注册模式
- 工具函数必须返回 [`ToolResponse`](mori/tool/internal_tools/example_tools.py:20) 对象，不是普通字符串
- 使用 [`toolkit.register_tool_function()`](mori/tool/internal_tools/example_tools.py:92) 注册，AgentScope 自动解析函数签名为 JSON Schema
- 工具函数支持 async/await

### 响应内容提取
- Agent 响应可能是字符串或包含 `TextBlock` 的列表
- 使用 [`_extract_text_from_response()`](mori/mori.py:383) 统一处理不同格式

### 长期记忆配置
- 嵌入模型配置在 `models.yaml` 的 `embedding_models` 部分
- Agent 通过 `long_term_memory.embedding_model` 字段引用嵌入模型名称
- 支持的嵌入模型类型：`dashscope`, `openai`, `gemini`, `ollama`
- 长期记忆模式：
  - `agent_control`: Agent 自主管理记忆（通过工具调用）
  - `static_control`: 开发者通过代码显式控制
  - `both`: 同时支持两种模式
- 存储配置：
  - `on_disk: true`: 持久化存储到 `storage_path` 指定的目录
  - `on_disk: false`: 内存存储（重启后丢失）
- 用户隔离：通过 `user_name` 区分不同用户的记忆数据

### 嵌入模型创建
- 在 [`mori/model/factory.py:create_embedding_model()`](mori/model/factory.py:91) 中实现
- 根据 `model_type` 创建对应的嵌入模型实例
- 必须显式设置 `dimensions` 参数，Mem0 从此推断向量维度
- `base_url` 传递方式：
  - OpenAI 嵌入模型：直接传递 `base_url` 参数（内部直接传给 `openai.AsyncClient`）
  - Ollama 嵌入模型：通过 `client_args` 传递 `base_url`

### 长期记忆实例化
- 在 [`mori.py:_create_long_term_memory()`](mori/mori.py:260) 中实现
- 使用 `Mem0LongTermMemory` 类
- 必须使用 [`NonStreamingModelWrapper`](mori/utils/model_wrapper.py:9) 包装主模型（Mem0 不支持流式响应）
- 必须通过 `VectorStoreConfig` 显式传递 `embedding_model_dims` 参数
- 在 `agent_control` 模式下自动注册 `record_to_memory` 和 `retrieve_from_memory` 工具

## 依赖管理

- 使用 `uv` 作为包管理器（不是 pip）
- 安装依赖：`uv pip install -e .` 或 `uv pip install -e ".[dev]"`
- 更新依赖：由 Dependabot 自动管理（每周一上午 9:00 北京时间）

## 测试命令

```bash
# 运行所有测试
pytest tests/

# 运行特定测试文件
pytest tests/test_config.py

# 带详细输出
pytest tests/ -v
```

## 代码风格

- Line length: 100 (black/ruff 配置)
- Python 3.10+ 语法
- 使用 pre-commit hooks: `pre-commit run --all-files`

---
> Source: [acgurl/mori](https://github.com/acgurl/mori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
