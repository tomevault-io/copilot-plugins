## ai-stock-picker

> aipick cn validate-evidence

# AGENTS.md

## 项目目标

本仓库提供严格、可审计的候选股重排工具。

当前公开命令包括：

```text
aipick cn pick
aipick us pick
aipick cn pick-plan
aipick cn validate
aipick cn validate-evidence
aipick cn stability-plan
aipick cn trial
```

候选池由外部系统生成。本仓库负责校验输入、构建 prompt、调用对应模型、校验模型输出并发布结果文件。

## 核心约束

修改代码时必须保留以下约束：

1. 模型只能选择候选池中的股票。
2. 股票名称和主题只能从候选池回填。
3. A 股正式选择只能使用 `DEEPSEEK_API_KEY`；冻结的 bounded v2 研究 shadow 可以隔离使用
   `OPENAI_API_KEY`，但不能生成正式发布选择。
4. 美股只能使用 `GEMINI_API_KEY`。
5. provider 凭据不能进入 URL。
6. provider 凭据不能随重定向转发。
7. 模型输出必须通过 strict schema 校验。
8. 输出数量必须与 `top_n` 完全一致。
9. 输出股票代码必须唯一。
10. 已存在的结果文件不能被覆盖。
11. A 股的 `reasoning` 和 `risk_note` 必须包含中文。
12. 输出不能声称具备严格时点证明或样本外证据资格。
13. `reasoning` 和 `risk_note` 的每个句子必须以字段名或获批自然标签引用实际候选字段，
    并拒绝 confusable、URL/IP/邮件、系统元数据、secret、交易指令、语义颠倒及外部事实。
14. prompt 中的 `risk_score` 只能投影为 `intraday_stability_score`，语义为值越高越稳定。
15. 显式凭据文件必须以安全文件描述符读取，要求当前用户、普通文件、`0600`、不超过
    128 KiB，读取前后元数据一致，且只能返回当前 provider 的专属 key。
16. 正式调用必须写入 append-only 证据目录，保存精确 prompt、脱敏 HTTP 信息和原始
    响应。已有目录、缺失清单或哈希不一致都必须失败。
17. 稳定性试验固定生成 `canonical`、`shuffle_101`、`shuffle_202`、
    `shuffle_303` 和 `opaque_404` 五个实验臂。三个 shuffle 必须真正改变最终 Prompt
    顺序。
18. 匿名臂同时替换股票代码和名称，保留标准顺序和全部数值字段，并保存基于
    `SHA256(campaign_id, date, symbol, 404)` 排序得到的可逆映射。
19. production v4 和 legacy v3 使用独立 Prompt 配置。`trial` 可以执行冻结的
    production `plan.json`，也可以读取旧版稳定性 `trial.json`。运行时不能覆盖冻结参数。
20. HTTP 成功但响应正文无法提取时，必须保存脱敏后的拒绝证据和原始响应字节，不得生成
    结果文件。证据清单同时记录请求模型别名和响应实际模型标识。
21. DeepSeek 计划必须显式保存 `thinking` 和 `max_tokens`。关闭推理时拒绝
    `reasoning_effort`，开启推理时保存该字段，且只能使用 `high` 或 `max`。输出预算
    上限为 65,536。
22. DeepSeek 开启推理时不能发送 `temperature`。关闭推理时请求必须包含显式的
    `thinking=disabled`、`max_tokens`、`temperature` 和 JSON 输出格式。
23. 证据 v2 分别记录传输、排序和发布合同。排序通过但发布失败时，只能写入受哈希保护的
    `ranking_diagnostic.json`，不能生成正式选择结果。
24. production v4 匿名计划必须同时冻结完整的代码映射和名称映射，保存规范化文件、映射
    哈希和可逆关系。完整 Prompt 中不能残留任何真实候选代码或名称。

## 主要目录

```text
src/stock_analysis/
├── app/cli.py
└── ai_lab/
    ├── candidates.py
    ├── alias_contracts.py
    ├── bundle_paths.py
    ├── commentary_contract.py
    ├── commentary_validation.py
    ├── contracts.py
    ├── credentials.py
    ├── evidence.py
    ├── evidence_consistency.py
    ├── evidence_contracts.py
    ├── frozen_plan.py
    ├── inference_config.py
    ├── prompting.py
    ├── providers.py
    ├── request_validation.py
    ├── shadow_campaign.py
    ├── shadow_exchange_validation.py
    ├── shadow_validation.py
    ├── stability_support.py
    └── selection.py

tests/
examples/
scripts/dev/
docs/
```

主要职责：

- `candidates.py` 读取、校验和归一化候选池
- `alias_contracts.py` 生成匿名映射的规范化字节和组合哈希
- `bundle_paths.py` 拒绝归档目录中的路径穿越和符号链接
- `commentary_contract.py` 定义客户文案自然标签、语义和 grounding 策略
- `commentary_validation.py` 对逐句 grounding、候选值和客户文案安全边界 fail closed
- `contracts.py` 定义模型输出和结果文件
- `credentials.py` 安全读取显式 provider 凭据文件
- `evidence.py` 写入和校验证据目录，并生成无网络稳定性试验计划
- `evidence_consistency.py` 复验归档文件、请求参数和跨文件关系
- `evidence_contracts.py` 分别判定传输、排序和发布合同
- `frozen_plan.py` 写入并重建无网络 production 选择计划
- `inference_config.py` 归一化并校验模型推理参数
- `prompting.py` 隔离 production v4 与 legacy v3 的 Prompt 渲染
- `providers.py` 调用 DeepSeek 与 Gemini，并保留可审计的响应信息
- `request_validation.py` 复验归档请求的完整消息和 provider 参数
- `shadow_campaign.py` 执行 append-only rep3/min2 shadow 与确定性共识
- `shadow_exchange_validation.py` 复验 shadow 请求和原始响应的语义一致性
- `shadow_validation.py` 离线复验 shadow bundle、campaign pin 和逐文件摘要
- `stability_support.py` 生成和校验匿名身份映射
- `selection.py` 构建选择计划、校验结果并按用途写入文件
- `src/stock_analysis/app/cli.py` 处理命令行参数和错误输出

## 开发命令

安装依赖：

```bash
uv sync --locked --group dev
```

运行完整检查：

```bash
uv run python scripts/dev/check.py
```

项目不使用 pre-commit、GitHub Actions 质量工作流或 Makefile。不要为同一套检查重新创建多个独立入口。

## 测试要求

新增或修改行为时，应覆盖：

- 正常输入
- 缺失字段
- 错误类型
- 边界数值
- 重复股票
- 候选池外股票
- 时区和日期边界
- provider 错误
- 凭据隔离
- 文件已存在
- 并发写入

测试不能访问真实 provider。provider 调用应通过 transport、caller 或 monkeypatch 完成。

## Prompt 与 schema

修改以下内容时，需要同步更新测试和文档：

- prompt 结构
- prompt 语言约束
- provider 默认模型
- 模型输出字段
- artifact 字段
- 输入契约
- 时间与证据限制

改变 production Prompt 语义时更新 `PROMPT_VERSION`。冻结的 legacy v3 只能用于预注册
稳定性试验，其字节级 golden test 必须持续通过。

改变持久化 schema 时更新 schema 版本，并说明兼容性影响。

## 文档同步

以下变更必须同步更新 README 或 `docs/`：

- CLI 参数
- 环境变量
- 输入格式
- 默认模型
- style 选项
- 输出字段
- 开发命令
- Python 支持版本

README 只保留新人上手所需内容。字段级契约、架构设计和开发细节放在 `docs/`。

## 安全要求

- 不提交 API 密钥。
- 不在错误信息中输出凭据或底层网络细节。
- 不允许候选字符串改变 prompt 指令。
- 不降低输入、响应和字符串长度限制，除非变更附有明确理由和测试。
- 不允许普通 JSON 或 CSV 通过自报字段升级为受信任契约。

## 修改原则

优先使用清晰、直接的实现。

当前只支持两个固定 provider，无需提前引入插件框架或复杂依赖注入容器。

模块拆分应基于职责，避免为了减少行数创建大量缺少独立意义的小文件。

---
> Source: [runchengxie/ai-stock-picker](https://github.com/runchengxie/ai-stock-picker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
