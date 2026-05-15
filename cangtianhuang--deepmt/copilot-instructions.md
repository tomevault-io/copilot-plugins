## deepmt

> 本文件为 Claude Code 提供项目上下文指引。

# CLAUDE.md

本文件为 Claude Code 提供项目上下文指引。

## 项目简介

DeepMT（Deep Metamorphic Testing）是面向深度学习框架（PyTorch、TensorFlow、PaddlePaddle）的**蜕变关系（MR）自动生成与分层测试系统**。

## 当前阶段与下一步

> **在开始任何”阶段开发”任务前，必须先阅读 `docs/dev/` 中的规划文档。”代码修复”任务不必查阅。**  
> 入口：`docs/dev/status.md` → 对应阶段文档。  
> 执行规范：`docs/dev/agent_rules.md`

| 阶段                                  | 状态     | 文档                                                           |
| ------------------------------------- | -------- | -------------------------------------------------------------- |
| Phase A：算子数据层完善               | ✅ 完成   | `docs/dev/achived/01_Phase_A_算子数据层完善.md`                |
| Phase B：算子层 MR 生成与知识库       | ✅ 完成   | `docs/dev/achived/02_Phase_B_算子层MR生成与知识库.md`          |
| Phase C：测试执行与跨框架适配         | ✅ 完成   | `docs/dev/achived/03_Phase_C_测试执行与跨框架适配.md`          |
| Phase D：缺陷分析与实验闭环           | ✅ 完成   | `docs/dev/achived/04_Phase_D_缺陷分析、实验闭环与研究结论.md`  |
| Phase E：演示交付与生产化加固         | ✅ 完成   | `docs/dev/achived/05_Phase_E_演示交付与生产化加固.md`          |
| Phase F：软件工程规范化               | ✅ 完成   | `docs/dev/achived/06_Phase_F_软件工程规范化与包发布准备.md`    |
| Phase G：统一IR与三层对象建模         | ✅ 完成   | `docs/dev/archived/07_Phase_G_统一IR与三层对象建模.md`         |
| Phase H：第二框架落地与真实跨框架适配 | ✅ 完成   | `docs/dev/archived/08_Phase_H_第二框架落地与真实跨框架适配.md` |
| Phase I：模型层MR自动生成引擎         | ✅ 完成   | `docs/dev/archived/09_Phase_I_模型层MR自动生成引擎.md`         |
| Phase J：应用层语义MR生成与验证       | ✅ 完成   | `docs/dev/archived/10_Phase_J_应用层语义MR生成与验证.md`       |
| Phase K：全层MR质量保障与知识库治理   | ✅ 完成   | `docs/dev/archived/11_Phase_K_全层MR质量保障与统一知识库治理.md`        |
| Phase L：论文实验基准与自动化数据生产 | ✅ 完成   | `docs/dev/archived/12_Phase_L_论文实验基准与自动化数据生产线.md`        |
| Phase M：真实缺陷挖掘与案例沉淀       | 🔄 进行中 | `docs/dev/13_Phase_M_真实缺陷挖掘与案例沉淀.md`                         |
| Phase M 系统能力缺口修复（T1~T9）     | ✅ 完成   | `docs/dev/archived/15_Phase_M_system_capability_gaps.md`                |
| Phase N：论文交付收口与复现资产封装   | ⬜ 未开始 | `docs/dev/14_Phase_N_论文交付收口与复现资产封装.md`                      |
| Phase O：核心框架插件闭环与健康管理   | ✅ 完成   | `docs/dev/archived/16_Phase_O_framework_plugin_closure_and_health.md`   |
| Phase P：仪表盘三层重设计             | ✅ 完成   | `docs/dev/archived/17_Phase_P_仪表盘三层重设计.md`                      |

**当前主链：** A~O + Phase P 均完成 → **当前进行：Phase M 真实缺陷挖掘主干**（用户手动执行）→ Phase N 论文交付收口未开始

## 环境与运行

项目使用 `uv` 管理虚拟环境：

```bash
source .venv/bin/activate && PYTHONPATH=$(pwd) python -m pytest tests/
```

关键环境变量：`OPENAI_API_KEY`、`DEEPMT_LOG_LEVEL`、`DEEPMT_LOG_DIR`。详见 `docs/environment_variables.md`。

## 文档

| 文件                            | 内容               |
| ------------------------------- | ------------------ |
| `docs/dev/status.md`            | 已完成模块清单     |
| `docs/dev/agent_rules.md`       | 编码智能体执行规范 |
| `docs/dev/`                     | 各阶段规划文档     |
| `docs/cli_reference.md`         | CLI 命令参考       |
| `docs/environment_variables.md` | 环境变量说明       |
| `docs/tech/operator_catalog.md` | 算子目录设计       |
| `docs/tech/operator_mr.md`      | 算子层 MR 技术细节 |
| `docs/quick_start.md`           | 快速上手           |

## 架构概览

采用**微内核 + 插件化**架构。MR 生成与测试执行分离：先生成 MR 存入知识库（SQLite），再复用于多次测试。

### MR 生成四阶段流水线（`deepmt/mr_generator/operator/operator_mr_generator.py`）

1. **信息准备** — 提取算子代码与文档（网络搜索）
2. **候选生成** — LLM 猜想 + 模板池匹配
3. **快速筛选（Pre-check）** — 随机输入数值验证
4. **形式化验证** — SymPy 符号证明

### 核心数据结构（`deepmt/ir/schema.py`）

- `OperatorIR`：算子描述（名称、输入、输出、属性）
- `ModelIR`：模型描述（model_type、task_type、input_shape、model_instance 等）
- `ApplicationIR`：应用描述（task_type、domain、sample_inputs、sample_labels 等）
- `MetamorphicRelation`：MR 对象，包含 `transform_code`（输入变换 lambda）、`oracle_expr`（输出关系表达式）、`layer`（算子/模型/应用）、`lifecycle_state`、`source`

### 三层 MR 生成概览

| 层次   | 生成器                                     | 特点                                            |
| ------ | ------------------------------------------ | ----------------------------------------------- |
| 算子层 | `OperatorMRGenerator`（四阶段流水线）      | LLM 猜想 + 模板池 + 数值预检 + SymPy 证明       |
| 模型层 | `ModelMRGenerator`（模板驱动）             | 结构分析（GraphAnalyzer）→ 策略库选择 → 生成 MR |
| 应用层 | `ApplicationMRGenerator`（LLM + 模板回退） | 场景注册 → 上下文构建 → LLM/模板生成 → 语义验证 |

## 项目地图

```
.
├── deepmt/                 # 主包（CLI 入口 + 所有核心模块）
│   ├── __init__.py         #   公共 API（导出 DeepMT, TestResult）
│   ├── __main__.py         #   python -m deepmt 入口
│   ├── cli.py              #   CLI 命令组
│   ├── client.py           #   DeepMT / TestResult 高层 API
│   ├── commands/           #   CLI 子命令实现（mr / test/ / repo / catalog / data / health）
│   │   └── test/           #     test 命令包（execution / analysis / evidence / history）
│   ├── core/               #   微内核框架
│   │   ├── config_manager.py   #   配置加载与管理
│   │   ├── logger.py           #   日志（get_logger / log_structured）
│   │   ├── plugins_manager.py  #   插件加载
│   │   └── results_manager.py  #   结果管理
│   ├── engine/             #   测试执行引擎
│   │   ├── batch_test_runner.py  #   算子层批量测试执行器
│   │   └── model_test_runner.py  #   模型层测试执行器
│   ├── ir/                 #   统一中间表示
│   │   └── schema.py           #   三层 IR（OperatorIR/ModelIR/ApplicationIR）+ MR 数据结构
│   ├── mr_generator/       #   MR 生成引擎
│   │   ├── operator/           #   算子层（四阶段流水线）
│   │   │   ├── operator_mr_generator.py  #   主生成器
│   │   │   ├── operator_llm_mr_generator.py  # LLM 生成
│   │   │   ├── sympy_prover.py               # 符号证明
│   │   │   ├── sympy_translator.py           # 代码→SymPy
│   │   │   └── ast_parser.py                 # AST 解析
│   │   ├── model/              #   模型层（Phase I）
│   │   │   ├── model_mr_generator.py         # 模板驱动生成器
│   │   │   └── transform_strategy.py         # 策略库
│   │   ├── application/        #   应用层（Phase J）
│   │   │   ├── app_mr.py                     # ApplicationMRGenerator
│   │   │   ├── app_context_builder.py        # 知识上下文构建
│   │   │   └── app_llm_mr_generator.py       # LLM 候选生成
│   │   ├── base/               #   知识库、模板池、MR 仓库
│   │   └── config/             #   模板/知识库 YAML + 算子目录
│   ├── model/              #   模型结构分析（GraphAnalyzer）
│   ├── application/        #   应用层场景描述（ApplicationScenario）
│   ├── benchmarks/         #   基准对象注册表
│   │   ├── models/             #   ModelBenchmarkRegistry + 预置 PyTorch 模型
│   │   ├── applications/       #   ApplicationBenchmarkRegistry（图像分类/文本情感）
│   │   └── suite.py            #   论文实验 Benchmark 固化清单（BenchmarkSuite）
│   ├── plugins/            #   框架适配器（PyTorch 完整，NumPy/Paddle 部分）
│   ├── tools/              #   通用工具
│   │   ├── llm/            #     LLM 客户端 / OCR
│   │   └── web_search/     #     搜索、Sphinx 解析、算子文档获取
│   ├── analysis/           #   验证与测试工具（三层分包）
│   │   ├── verification/       #   验证核心与输入生成
│   │   │   ├── mr_prechecker.py        # 算子层数值预检
│   │   │   ├── mr_verifier.py          # 算子层 oracle 验证
│   │   │   ├── model_verifier.py       # 模型层 oracle 验证
│   │   │   ├── semantic_mr_validator.py # 应用层语义验证（Phase J）
│   │   │   └── random_generator.py     # 随机输入生成器
│   │   ├── reporting/          #   测试报告、证据收集与变异测试
│   │   │   ├── report_generator.py     # 算子层报告
│   │   │   ├── application_reporter.py # 应用层报告生成（Phase J）
│   │   │   ├── evidence_collector.py   # 证据包采集
│   │   │   └── mutation_tester.py      # 变异测试器
│   │   └── qa/                 #   跨框架一致性、缺陷去重与知识库审计
│   │       ├── cross_framework_tester.py # 跨框架一致性
│   │       ├── defect_deduplicator.py  # 缺陷线索去重
│   │       └── repo_audit.py           # 知识库审计
│   ├── experiments/        #   论文实验管理（Phase L）
│   │   ├── organizer.py        #   实验数据组织器（RQ1-RQ4 汇总）
│   │   ├── rq_config.py        #   RQ 口径与指标定义
│   │   ├── case_study.py       #   Case Study 数据模板
│   │   ├── version_matrix.py   #   框架版本矩阵
│   │   ├── stats/              #   论文统计聚合与导出
│   │   │   ├── aggregator.py       # ThesisStats 聚合
│   │   │   └── exporter.py         # CSV/JSON/Markdown 导出
│   │   └── runs/               #   运行清单与环境记录
│   └── monitoring/         #   健康检查与进度追踪
├── tests/                  # 测试用例（unit/ + integration/）
├── demo/                   # 快速演示
├── docs/                   # 开发文档
├── data/                   # 数据（日志、SQLite、MR YAML）
├── config.yaml             # 运行配置
└── pyproject.toml          # 项目元数据
```

## 开发规范（强制）

每次完成功能开发后，必须同步更新以下内容：

1. **文档同步**
   - 修改了开发进度、模块状态、架构设计 → 更新 `docs/dev/status.md`（已完成模块列表）
   - 完成某阶段中的关键任务 → 在对应 `docs/dev/0X_Phase_*.md` 中标记完成状态
   - 新增或修改了 CLI 命令（含命令名、选项、行为）→ 更新 `docs/cli_reference.md`

2. **依赖同步**
   - 新增了 `import` 的第三方包 → 同时更新 `pyproject.toml`（`dependencies`）和 `requirements.txt`，两者保持一致

3. **测试覆盖**
   - 任何新功能必须在 `tests/` 下新增最小测试代码（不要求覆盖完全，只验证功能打通）
   - 测试文件放在对应层级：`tests/unit/` 或 `tests/integration/`
   - 单元测试不得依赖 LLM API 或网络（通过 `use_llm=False` / mock 隔离）

4. **框架参数化（拓展性）**
   - 凡涉及框架相关逻辑，框架名称必须以 `FrameworkType` 参数传入，**不得写死**（包括字符串字面量 `"pytorch"`）
   - PyTorch 先行实现，其他框架入口处抛出 `NotImplementedError`，保留接口占位
   - 参考现有模式：`_SUPPORTED_FRAMEWORKS = {"pytorch"}` + 显式的 `not_implemented_error` 提示

5. **无向后兼容性负担**
   - 本项目处于初期开发阶段，**不考虑任何向后兼容性**
   - 重命名函数/类/CLI 命令、修改接口签名、调整数据结构时，必须**彻底清理**：删除旧名称、更新所有调用点、移除兼容性代码
   - 禁止保留废弃别名、`_deprecated_` 包装、兼容性注释（如 `# kept for backward compat`）等过渡代码

6. **禁止自主提交代码**
   - 未经用户明确要求，**禁止执行 `git commit` / `git add`**；用户自行审查并提交

7. **插件职责边界（反复违规，强制记忆）**
   - `FrameworkPlugin` / `FrameworkAdapter` 只提供**基础、框架专属**的原语接口：
     - ✅ 合法：`make_tensor(shape, dtype, value_range)`、`allclose`、`to_numpy`、`get_shape`、`_execute_operator`
     - ❌ 禁止：任何需要解析项目自定义数据格式（`input_specs`、MR 结构、YAML 字段）的逻辑
   - 需要解析 `input_specs` 或执行生成策略的代码，统一放在 `deepmt/analysis/`（如 `InputGenerator`），再调用插件的基础接口

8. **禁止作弊式补丁（反复违规，强制记忆）**
   - 修复功能 bug 时，**必须从逻辑根源修复**，严禁绕过标准流程的投机补丁，例如：
     - ❌ 禁止：直接向 MR 知识库写入预制答案（绕过生成流水线）
     - ❌ 禁止：为特定算子单独注册硬编码 MR（`relu`、`abs` 等），伪造"生成"效果
     - ❌ 禁止：在预检/验证函数里为特定算子名称写特殊分支，假装通过验证
     - ❌ 禁止：跳过 pre-check 或 SymPy 证明以规避失败
   - 标准流程必须完整走通：模板匹配 → 数值预检 → （可选）SymPy 证明 → 知识库存储
   - 若某步骤因环境或算子特性而失败，应修复该步骤的**通用逻辑**，而非为失败案例开后门

---
> Source: [cangtianhuang/DeepMT](https://github.com/cangtianhuang/DeepMT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
