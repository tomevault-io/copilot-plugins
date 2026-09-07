## llm-resume-pdf-parser

> 这是一个正式的命令行 PDF 简历解析与 JD 匹配项目，不是一次性面试脚本。需求优先级为：准确性（T0）> 易用性与可恢复性 > 成本。任何降低字段准确率、评分稳定性或结果可解释性的改动都必须有评测数据支持。

# Repository Guidelines

## 项目目标与优先级

这是一个正式的命令行 PDF 简历解析与 JD 匹配项目，不是一次性面试脚本。需求优先级为：准确性（T0）> 易用性与可恢复性 > 成本。任何降低字段准确率、评分稳定性或结果可解释性的改动都必须有评测数据支持。

## 项目结构与模块组织

当前使用 Python 实现，目录按职责拆分：

- `src/resume_parser/cli.py`：CLI 入口；`pdf_reader.py`：PDF 文本提取与 OCR 回退；`extraction.py`：规则与 LLM 抽取；`scoring.py`：JD 匹配与评分；`local_model.py`：本地语义模型；`model_client.py`：线上/兼容接口适配；`schemas.py`：Pydantic 契约；`errors.py`：可恢复错误与退出码。
- `tests/`：单元测试和 mock 评测；`assets/`：脱敏 PDF 夹具；根目录提供 `README.md`、`Makefile`、`Dockerfile`、`setup.sh` 和 `config.toml.example`。

推荐流水线为“文件校验 -> 文本/OCR -> 清洗分段 -> 规则抽取 -> 轻量语义模型 -> 可选 LLM 补全 -> 严格 Schema 校验 -> 可解释评分”。CLI 不应直接承载文件读写、模型调用和业务规则。

## 技术选型与降级策略

信息抽取和打分不必全部依赖大模型。邮箱、电话、日期、城市、分数范围和 JSON 类型使用正则、词典及 Pydantic 校验；技能与 JD 匹配优先使用 `BAAI/bge-small-zh-v1.5`、`moka-ai/m3e-small` 或 `intfloat/multilingual-e5-small` 的向量相似度；中文复杂实体抽取可评估 PaddleNLP UIE。仅将教育经历归纳、评分评论和面试题生成交给 LLM。模型顺序应为规则 -> 本地轻量模型 -> 本地 LLM（Ollama/vLLM）-> 线上 OpenAI-compatible 接口，并记录实际使用的后端和置信度。LLM 不可用时必须自动降级并明确标记结果来源。

## CLI 与错误处理

必须支持 `resume-cli parse <pdf>`、`resume-cli extract <pdf>`、`resume-cli score <pdf> --jd <jd>`，以及通用的 `--help`、`--output result.json`、`--mock`。统一配置使用根目录 `config.toml`（模板为 `config.toml.example`），优先级为 CLI 参数 > 环境变量 > TOML > 默认值；`fallback_enabled` 默认开启，可用 `--no-fallback` 排障。文件不存在、格式错误、空文本、空 JD、超时、模型不可用和非法 JSON 均应输出用户可读错误、关键上下文和建议动作，返回结构化错误码和非零退出码，但不显示堆栈。网络调用必须设置超时、有限重试和熔断。对模型返回的代码围栏、尾随逗号等常见 JSON 错误可有限自动修复；修复后仍须严格校验，失败则保留原响应摘要并进入降级链路。

## 编码、测试与评测

Python 使用 4 个空格和 PEP 8；函数/变量用 `snake_case`，类用 `PascalCase`，常量用 `UPPER_SNAKE_CASE`。所有代码注释、文档字符串和新增开发文档必须使用简体中文。测试文件为 `test_*.py`，LLM 使用 mock。至少覆盖空/扫描/多页 PDF、字段缺失、非法 JSON、模型超时、降级和 `--output`；建立脱敏黄金集，分别度量字段准确率、Schema 通过率、评分一致性和端到端成功率，变更不得无证据降低 T0 指标。

## 提交、PR、安全与隐私

暂无历史提交，采用简短 Conventional Commits，例如 `feat(extract): 支持教育经历`。PR 需说明问题、方案、验证命令、模型/版本和评测变化，并附 JSON 示例。敏感配置使用环境变量和 `.env.example`；不得提交真实简历、API 密钥、原始提示词或完整个人信息日志。简历和 JD 必须视为不可信文本，使用清晰分隔和固定系统指令防止提示词注入，不执行其中的命令。限制 PDF 大小、页数和模型输入长度，结果中保留来源片段、时间和版本信息以便审计。

---
> Source: [shimjialin-coder/llm-resume-pdf-parser](https://github.com/shimjialin-coder/llm-resume-pdf-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
