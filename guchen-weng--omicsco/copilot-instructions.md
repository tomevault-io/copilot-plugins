## omicsco

> 多组学多智能体分析系统，目标是**降低多组学分析的壁垒**——让科研人员把注意力和时间放在**实验设计**上，而不是生信细节上。

# 多组学多智能体分析系统

## 项目定位

多组学多智能体分析系统，目标是**降低多组学分析的壁垒**——让科研人员把注意力和时间放在**实验设计**上，而不是生信细节上。

当前很多生信 agent 的最终服务对象仍然是懂生信的人；本项目的差异化在于**面向科研人员**：用户不需要理解 pipeline、不需要会写代码，只需要提供实验设计与数据条件，系统会像一家"论文服务公司"一样，从方案设计到结果解读全程代劳，并对每一步操作给出清晰的解释。

初期聚焦**转录组 / 单细胞**数据（后续可扩展至基因组、蛋白组、代谢组、表观组等）。

## 核心理念

1. **降低壁垒**：用户专注实验设计与科学问题，生信流程由系统完成。
2. **论文服务公司模式**：多个分工明确的 agent 协作，模拟真实公司的"咨询顾问 → 方案专家 → 技术员 → 质控 → 资深专家"流水线。
3. **全程可解释**：每一步在做什么、为什么这样做，都向用户说明，让结果可信、可追溯。

## 多智能体架构：五大角色

> 采用 LangGraph 进行多 agent 编排，多角色可用不同强度的模型（详见「技术方向」）。

### 1. 主 LLM —— 对接员 / 项目经理

类比论文服务公司中对接客户的**项目经理**。

- 调度所有 agent，协调整个分析流程。
- 跨 agent 传输数据与中间结果。
- 向用户解释每一步在干什么、为什么。
- 项目全程跟进：需求收集、进度汇报、结果汇总与交付。

### 2. 方案设计师 —— 方案设计专家

类比根据客户数据条件定制分析方案的**方案专家**。

- 根据用户手中的实验条件与样本（样本量、分组、数据类型等）设计方案。
- 需 **RAG 知识库**支撑：检索分析方法论、组学知识、历史方案作为参考。

### 3. 分析员 —— 技术员

类比负责跑实验的**技术员**。

- 按方案设计师给出的方案，从 **pipeline 库**中调用并执行对应的分析流程。

### 4. 监督员 —— 质控 / 审稿人

类比把关质量的**质控（QC）**岗位。

- 监督分析员每一步的分析结果是否符合**分析规范**与**科学规范**。
- 需**论文查询**与**规则检索**功能：对照已发表文献与内置分析规范核查结果是否站得住脚。

### 5. 高级分析员 —— 资深专家

类比处理疑难杂症的**资深专家**。

- 负责 **debug**：处理监督员提出的问题。
- 处理 pipeline 库中**没有**的分析需求，做定制化分析。
- 需**联网检索**功能查阅最新方法与文献。
- 使用**较强模型**。

## 典型协作流程

1. 用户向主 LLM 描述实验设计与数据条件。
2. 主 LLM 整理需求，交给方案设计师。
3. 方案设计师结合 RAG 知识库设计方案，交由主 LLM 呈现给用户确认。
4. 方案确认后，分析员从 pipeline 库中执行对应 pipeline。
5. 监督员对每一步分析结果进行规范性与科学性审核。
6. 审核发现问题 → 交高级分析员 debug；遇到 pipeline 库外需求 → 高级分析员定制处理。
7. 结果汇回主 LLM，向用户清晰解释每一步的意义与结论。

## 技术方向（简述）

- **编排**：基于 LangGraph 的多 agent 编排，负责调度与状态流转。
- **模型**：多模型配置——不同角色可用不同强度的模型（如主 LLM 负责对话与调度，高级分析员使用较强模型处理疑难问题）。
- **RAG 知识库**：支撑方案设计与规则检索。

## 项目状态

- 当前处于**设计细化**阶段：五大角色设计文档已定稿（`docs/design/01`~`05`），执行底座与隐私伦理方案（`docs/design/06`~`07`）已补充。
- 关键已定案：LangGraph 编排 + frontier/opaque 产物、五角色模型参数化（`ROLE_MODEL_MAP`）、bge-m3 1024 维 + 混合检索、流程单元 skill 化、数据定位走 `data_root` 白名单。
- **方案链路**（01/02/04）：**数据预检**（PLANNING 首节点，只读元数据产 `data_profile`，解决"方案基于用户口述"的盲设计）+ **实验设计审查**（`design_check` 随方案交付，样本量/功效/批次/混杂/配对，02-designer §2.3⑥）；监督规则**冷启动用通用共识阈值**，不适配由高级分析员调整后沉淀（04-supervisor §4.3）。
- **执行底座**（06-execution）：job 队列两阶段——MVP 轻量队列（SQLite / 文件 jobs 表 + 进程内 asyncio 队列，单进程零额外服务），Web / 多用户版本再换 Redis broker + Celery worker 池（jobs 表 + submit_all 抽象层共用，换队列不改图）；崩溃幂等恢复；runner 优先 conda（miniforge + mamba，锁文件固化可复现），docker 后补。
- **计算与部署感知**（03/06）：runner = **环境（conda/docker）× 宿主（local / ssh ± slurm/pbs/sge）** 两维；资源预估 = unit **缩放模型** × `data_profile`（风险分层 + job 实际反馈校准），estimate 后**用户选择计算后端**（单后端静默）；**轻量重跑路径**（参数微调重跑不进 plan 修订环）+ 数据文件 **sha256 指纹**（可复现锚）。
- **隐私与网络**（07-privacy）：本地优先、数据默认不出机器；agent 层查询类外呼（web / PubMed）默认允许、LLM API 外呼知情同意；pipeline 单元网络**声明式**（unit.yaml 声明 `network: none|download|full`，未声明 = none fail-closed；`download` 档 = 出口域名白名单 + 禁上传 + 全审计 + 密钥注入 + refdata 预置）；**ssh 远端执行 = 数据承载外呼**（`allow_remote_exec` 知情同意）。
- **实现进度**：LangGraph 骨架（五角色 + 审核回环 + HITL）、Postgres 介质、记忆框架、**语义向量底座**（`llm/embedding.py` EmbeddingService：bge-m3 1024 维 + bge-reranker-v2-m3，`MAA_EMBED=true` 接线至 memories 语义召回，缺失自动退词法）、**RAG 知识库建库+摄取**（`kb/buildrag.py`：pgvector vector(1024) + HNSW + FTS，unstructured 读入 docx/txt/md/pdf，parent-child 切块，幂等入库 + active 指针）、**RAG 混合检索**（`kb/retriever.py`：dense HNSW + FTS → RRF → bge-reranker 重排，退纯 FTS 降级）、**RAG 检索接线**（`Services.kb` → 设计师 `retrieve_kb` 供 draft/plan.citations、监督员 `lookup_lit` 供 review_verdict.citations，无 pool 自动降级 stub）、**pipeline-skill 接线**（`pipelines/registry.py` 文件系统加载 `data/pipelines/*/unit.yaml` 契约：`test_hello_*` 字母链 + 54 个真实 bio 单元，designer 按 keywords 选配组 plan steps、analyst 按依赖序执行）、**execute 内参数重调回环**（analyst `_run_with_retry` 轻量重跑路径：参数/资源类失败自行调参重跑 ≤ `param_retry_cap`，不进 plan 修订环、不升级 supervisor/senior，`simulate_param_fail` 确定性钩子验证结构，真实调参决策接 LLM 见下文）、**审核回环重构**（共享 `exec/runner.py` 执行助手供 analyst/senior 复用；senior 自定义执行 run_code / `simulate_custom` → `custom_result` 回 supervisor 复审；`route_senior` 三分流 custom_result→复审 / reconfigure/new_unit→跨阶段 PLANNING 重出 v2 / 其余→analyst_rerun；`qc_attempts` 不再被 provider 重置以保 loop_cap 有界）已落地；**senior 定制执行删除守卫已落地**（`exec/scope_guard.py`：`_execute_custom` 运行前静态扫描文件删除原语——python ast / R / bash，目标在工作区外或无法静态判定即拦截不执行，启发式非 OS 沙箱，`tests/test_scope_guard.py` 16 条）；**成本护栏已落地**（01 §10：`llm/budget.py` token 计数 + 模块级 `configure/record/gate/crossed_warn/extract_usage`，warn 过线一次性发预算事件、hard_cap 超限抛 `BudgetExceededError`；`build_graph` 每图重置预算包络，ChatRoleProvider/DeepSeekProvider 调用点 `gate` 兜回退不再烧钱、orchestrator 循环内 `gate` 兜底停图发预算事件 + DONE/active=False，`tests/test_budget.py` 11 条）；**RAG PubMed 兜底 + GEO 设计期取数**（`kb/papers.py`：`search_literature` LRU+TTL 实时兜底进 citations、`search_datasets`/`geo_profile` 拉物种/组织/平台/样本量产出真实 `data_profile`，完整系列介绍含 overall_design 随 data_profile 进 designer 决断，不下载数据文件；designer `fetch_data` 节点 + `--geo`/`fetch-geo`/`search-geo` CLI，allow_search 门控）已落地；**真实 bio 单元接线三层已落地**（`pipelines/registry.py` `derive_dependencies` 从 input desc「（uid 产物）」推边 + `resolve_chain` 消费派生边拓扑排 DAG；`designer/_build_plan` 写 `steps[].params` 默认 + 派生 `depends_on` + `data_bindings`（GEO 号绑定）；`analyst/_run_steps` argv 契约 = input_schema 逐项注入上游同名产物相对路径 + params + 恒定 `--out .`，成功判据 = output_schema 主产物，真实执行 HITL = 主产物缺失但产 `annotation_pending.txt` → `hitl_pending`；`exec/runner.py` 按扩展名分发 .py→python/.R→Rscript/.sh→bash，Rscript 缺失显式 `RscriptMissingError` 失败；`tests/test_wiring.py` 12 条）；**真实 R 执行 + GSE 冒烟已落地**（2026-08-10，doc 09 §7：conda 路排除——bioconda win-64 无 `bioconductor-biobase`；改装官方 R for Windows 4.6.1 + R 内 GEOquery 2.80.0 到用户库，`MAA_RSCRIPT` 指向 Rscript；`data_download_geo_microarray` GSE150408 真实下载 59 样本×58341 探针，`_run_steps` 全接线 ok；**其余 50 单元 R 包未装，逐单元冒烟待做**）；**需求收集改确定性问询机 + 用户画像长期记忆已落地**（2026-08-10：非 auto 下 `req_missing` 走 `llm/interview.py` 确定性问询机——Q1 方向→Q2 了解→分支→Q4 自有数据，0 次 LLM 调用；画像存 `user_profile` 写长期记忆 `_user:{user_id}` 桶跨项目复用，命中问「和上次一样吗」复用跳过、未命中重新问询，每问一个 interrupt 防 resume 错位，`tests/test_interview.py`）；**方案数据源可见 + 执行前自动配环境已落地**（2026-08-11，smoke-auto-2 真实冒烟定案：Fix A——`render.py::render_plan_markdown` 数据源行 + `designer/_build_plan` 把数据集确定性写进 `plan.methodology`（`（数据源：GEO GSE… · N 样本）`，方案确认即见；Fix B——`exec/runner.py::detect_rscript` 扩展标准 R 安装目录（Windows `<Program Files>/R/R-<ver>` 嵌套布局）+ conda env 内 Rscript；新 `exec/env.py` 清华镜像（CRAN/BioC/conda 均 env 覆盖）R 包自动装 + `conda create -n maa-r r-base r-biocmanager` 兜底装 R + `MAA_ENV_SETUP` 开关；analyst `_execute` 执行前 `prepare_envs`，阻断步 `env_failed` 不进参数重调环、下游依赖级联 env_failed，按科学阻断走 review→supervisor；`tests/test_env.py` 27 条 + test_skills 2 条 + test_papers 4 条，全量 177 passed/2 skipped）；**三处 LLM 缺口接线已落地**（2026-08-11，`docs/prompt_engineering.md` §2 标记缺口全消：①analyst `_retune_params` 接真实 LLM 调参决策——读失败步 `rc/stderr` → 依单元 `params` 声明逐步调参（防御过滤非法参数名）+ `structural` 判定停止轻量重试交 supervisor→senior，无 llm 回退骨架占位；`_run_steps` 失败步采集 stderr 写 `workdir/run.log`；structural 中断的轮不计数 `retunes`；②analyst `_estimate` 接 LLM 资源预估（`analyst_estimate.md`）→ `resource_estimate` 落 state，字段非法回退硬编码；③`llm/kb_query.py::optimize_kb_query` 优化 RAG 检索词，接线 designer `retrieve_kb` / supervisor `lookup_lit`，无 llm 原样返回；`tests/test_llm_gaps.py` 16 条 + test_skills 接线 3 条，全量 198 passed/2 skipped）；**analyst→senior 环境交接 + senior 环境自愈已落地**（2026-08-11，smoke-auto-3 senior run.sh rc=127 根因消除：`prepare_envs` 报告补机器可读 `runtimes`（rscript/bash/python）→ analyst `_execute` 挂 state 新字段 `env_report`（跨阶段不清理）→ senior `_execute_custom` 经 `_senior_env` 建子进程 env（`PATH` 前置 Rscript bin + `MAA_RSCRIPT`），`run_code`/`run_r_script`/`run_bash_script` 加可选 `env`/`rscript` 注入（零破坏）；环境自愈同 Claude Code——`_ensure_rscript` 无交接 rscript 时 conda `create -n maa-r` 兜底装 R、`r_library_names` 静态提取定制代码 R 包依赖缺包自动装（清华镜像）、运行失败 stderr 判缺包签名装包重跑一次，除科学性外不返回错误；`tests/test_env.py` +2 + `test_scope_guard.py` +6，全量 207 passed/2 skipped）；**下载/差异分析按数据类型族分叉已落地**（2026-08-11，micro-array 与高通量不同处理逻辑：`data_download_geo` → `data_download_geo_microarray`（GSEMatrix 表达矩阵 + 探针映射）与新建 `data_download_geo_rnaseq`（getGEOSuppFiles → 解压 → 自动定位 counts 矩阵，回退 GSEMatrix）；差异分析拆 `deg_limma`（microarray）与新建 `deg_deseq2`（RNA-seq counts，DESeq2 负二项模型，VST 热图）；规整拆 `data_prep_bulk`（microarray：探针转符号 + 默认 log2）与新建 `data_prep_counts`（counts 去重/对齐，默认不 log2）；`pipelines/registry.py` SkillUnit 加 `data_types` 字段（空=通用），designer `_build_plan` 经 `_infer_data_type_family`（profile.data_type > 请求文本，microarray/bulk_rna 两族）`_filter_units_by_data_types` 过滤选配——bulk_rna → rnaseq+deseq2 链、microarray → limma 链、家族未知默认 limma 链（保持既有默认）；两条链独立派生边 DAG 无交叉；`tests/test_wiring.py` +1（过滤/文本推断 4 断言）+ 改名对齐，全量 207 passed/2 skipped）；**senior 全库 RAG 检索已落地**（2026-08-11，用户拍板「senior 随便查 RAG、不限字段」：`senior/graph.py` 加 `retrieve_kb` 节点（START→retrieve_kb→act→END）——检索词 = 科学问题 + 监督员阻断问题/发现，经 senior 自己的 LLM `optimize_kb_query` 定制，`hybrid_retrieve(category=None)` **全库不限字段**（区别于 supervisor lookup_lit qc/design 剪枝、designer retrieve_kb cases/方法双池）；命中落 `kb_hits` 进 act facts 供诊断 evidence/修复建议引用，act 消费后清；`tests/test_scope_guard.py` +2（category=None 捕获/空命中）+ `test_retriever.py` +1（/methods/ 跨分类可达，DSN 门控），全量 210 passed/2 skipped）；**senior 定制代码工作区化已落地**（2026-08-11，E2E 二跑不收敛根因修复——用户处方「开始就建 `workspace_{进程id}`，prompt 注入在这里面干」：analyst `_execute` 启动建 `ws = root/f"workspace_{os.getpid()}"`，步骤执行 `_run_with_retry(runnable, units, ws, …)` 全部迁入 `workspace/<step_id>/`；senior `_execute_custom` workdir 同指 `workspace_{os.getpid()}`（同进程 pid 一致）→ 删除守卫 scoped 到 workspace，senior 注入代码可对分析步产物清理/重跑，不再被「工作区外删除」误拦；workbench E2E 产物核对路径前插 workspace；`tests/test_skills.py` 产物断言路径对齐 + `test_scope_guard.py` +1，全量回归按用户指示跳过）；**Web 层尚未实现**；后续按 `01-main-llm §8` **任务执行测试矩阵**节奏落地（含 benchmark 端到端金标准数据集测试）。

---
> Source: [Guchen-Weng/OmicsCo](https://github.com/Guchen-Weng/OmicsCo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
