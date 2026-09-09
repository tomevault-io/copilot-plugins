## sirchmunk

> 本文件定义 Sirchmunk 项目中 AI agent、自动化助手和协作者在修改代码、文档、实验配置与 Web 界面时必须遵守的协作规约。其目标是保护项目结构、核心检索算法链路、对外接口、实验可复现性和 Web 用户体验，避免未经确认的破坏性改动。

# AGENTS.md

本文件定义 Sirchmunk 项目中 AI agent、自动化助手和协作者在修改代码、文档、实验配置与 Web 界面时必须遵守的协作规约。其目标是保护项目结构、核心检索算法链路、对外接口、实验可复现性和 Web 用户体验，避免未经确认的破坏性改动。

## 1. 总体原则

- 默认优先保持现有架构、目录边界、公共接口和实验协议稳定。
- 任何涉及项目边界、核心行为、公共 API、Web 布局或论文级实验协议的改动，都必须先说明影响范围、风险和回滚方式。
- 严禁硬编码配置项、路径、阈值、模型名、数据集路径、端口、密钥或运行环境假设；必须通过参数、配置文件、环境变量或集中常量管理。
- 不得提交真实密钥、token、私有数据路径或未脱敏的环境快照。
- 不能为了让测试通过而降低质量门控、跳过验证、删除关键检查或静默吞掉错误。

## 2. 必须项目 Owner 确认的改动

以下改动在实施前必须获得项目 owner 明确确认。若当前任务中没有明确授权，agent 必须先停下来提出方案，不得直接修改。

### 2.1 项目结构与模块边界

- 新增、删除、移动、重命名顶层目录或核心模块目录。
- 改变 `src/`、`benchmarks/`、`web/`、`requirements/`、`config/`、`scripts/`、`docker/` 等目录的职责边界。
- 修改 Python package 名称、import 根路径、entry points、构建配置或发布配置。
- 大规模重构文件布局、抽取公共模块、合并模块或拆分模块。

### 2.2 核心算法链路

以下属于高风险核心链路，必须先取得 owner 确认，并在动手前提供技术方案：

- `src/sirchmunk/search.py` 中 FAST / DEEP / FILENAME_ONLY 主检索链路。
- DEEP 模式的并行检索、证据采样、ReAct refinement、自修正、树导航、知识复用、编译产物复用等流程。
- 影响答案生成、证据选择、文件选择、telemetry、token budget、fallback 策略的逻辑。
- 影响 source fidelity、raw-corpus 检索、无扩展名纯文本语料、特定数据集原始分片读取的逻辑。
- 任何可能改变 benchmark 主结果、论文指标或用户可见搜索行为的算法策略。

### 2.3 对外关键接口和函数入口

以下改动必须 owner 确认：

- Public Python API、SDK 入口、类名、函数签名、返回结构或异常语义。
- CLI 参数、默认值、输出格式、退出码或脚本职责变化。
- MCP server、API route、HTTP endpoint、schema、payload 格式或鉴权行为。
- `pyproject.toml`、`setup.py`、entry points、package data、依赖范围等发布相关改动。
- 被 README、论文实验、benchmark 流程或外部用户依赖的函数入口。

### 2.4 Web 端功能模块与布局

以下 Web 改动必须 owner 确认：

- `web/app/` 路由结构、页面层级、layout、导航、全局样式或响应式布局。
- `web/components/` 中核心交互组件、图表、监控面板、上传/检索入口和状态展示组件。
- `web/context/`、`web/hooks/`、`web/lib/` 中影响全局状态、请求封装、数据流或错误处理的逻辑。
- 用户可见的交互流程、加载状态、错误提示、结果展示方式、监控指标含义。
- 任何可能破坏现有截图、演示视频、README 展示或产品叙事的一致性改动。

### 2.5 Benchmark 与 ResearchOps 实验治理

以下实验治理改动必须 owner 确认：

- 核心 benchmark 的 sampling protocol、GoldenSet、sample IDs、checksum、诊断子集、固定样本复现等相关逻辑。
- Frozen evaluation 门控、validator error/warning 等级、artifact schema、report schema。
- Baseline lifecycle、setup/index/storage cost、failure classification、import coverage、paired statistics。
- `run_quickstart.py`、`run_sampling.py`、`run_evaluation.py`、`run_lifecycle_eval.py`、`run_scaling_study.py` 的职责或关键参数。
- 任何会改变论文主表、sample pairing、frozen stage、cache policy、实验统计口径或可复现性的默认行为。

## 3. 必须二次确认的高风险改动

以下改动即使 owner 已同意总体方向，也必须在实际编辑前进行二次确认。二次确认应列出具体文件、函数、行为变化、验证计划和回滚方式。

- DEEP 模式主链路、证据采样、ReAct、自修正和树导航逻辑。
- 对外 API / CLI / MCP / Web route 的 breaking change。
- Web layout、导航结构、核心页面交互方式。
- 论文级 benchmark protocol、GoldenSet、validator error gate、sample checksum 或主实验统计口径。
- 删除兼容逻辑、迁移数据格式、改变默认配置或默认模型行为。
- 大范围自动格式化、批量重命名、批量移动文件。

## 4. 可以直接执行的低风险改动

在不触碰上述高风险范围的情况下，以下改动通常可以直接执行：

- 明确的错别字、注释、局部文档措辞修复。
- 不改变行为的局部类型标注、lint 修复、格式修复。
- 新增非默认启用的测试、fixture、示例脚本。
- 对已存在配置项补充说明，不改变默认值。
- 用户在当前任务中明确指定的单文件小范围修改。

即使属于低风险改动，也必须保持最小影响面，不得顺手重构无关代码。

## 5. 实施前检查清单

在修改前，agent 应完成以下检查：

- 确认当前分支、工作区状态和是否存在用户未提交改动。
- 明确本次改动是否触发 owner 确认或二次确认规则。
- 阅读相关模块现有实现，不凭猜测修改。
- 对 benchmark / ResearchOps 相关改动，确认是否影响 frozen stage、sample IDs、validator gates、report artifacts 和统计口径。
- 对 Web 改动，确认是否影响 layout、路由、全局状态或用户可见流程。

## 6. 实施后验证要求

修改完成后，应根据改动范围执行最小但充分的验证：

- Python 代码优先运行 `py_compile`、相关 CLI `--help`、局部 smoke test 或对应单元测试。
- Web 改动优先运行类型检查、lint、构建或本地页面 smoke test。
- Benchmark 改动必须验证 sample ID 一致性、manifest/checksum、validator 输出、关键 CLI 参数和统计口径。
- 文档改动需确认路径、标题、术语和最新实现一致。
- 若验证无法运行，必须在最终说明中明确原因和风险。

## 7. 沟通与记录规范

- 对高风险改动，先给出方案、影响面、风险和验证计划，再等待 owner 确认。
- 对二次确认改动，必须再次明确即将修改的文件和函数，不得用笼统描述代替。
- 最终回复应说明修改内容、验证结果、未验证项和剩余风险。
- 不得隐藏失败、跳过失败或把失败降级为成功。

## 8. 特别保护的项目主张

Sirchmunk / LENS 的核心实验主张是：在动态原始数据场景下，于免预处理和源保真约束下保持竞争性质量，并显式报告 setup、indexing、storage、update 和 query 的全生命周期成本。

任何修改若会削弱以下能力，必须 owner 确认：

- raw-corpus / indexless / embedding-free 的核心叙事。
- source fidelity 与 evidence traceability。
- 论文级 raw-corpus protocol。
- frozen stratified subset、paired statistics、sample checksum。
- 外部 index-heavy baseline 的 lifecycle feasibility 对比。
- Web 和文档中对上述主张的表达一致性。

---
> Source: [modelscope/sirchmunk](https://github.com/modelscope/sirchmunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
