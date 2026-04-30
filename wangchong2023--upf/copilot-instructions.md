## upf

> 该目录 (`/Users/constantine/Documents/Work/Code/projects/upf`) 是一个基于云原生架构的 5G UPF (User Plane Function) 研发项目。本 `GEMINI.md` 为 Gemini CLI 提供核心项目背景与指令指引。

# 项目上下文：upf (云原生 5G UPF)

该目录 (`/Users/constantine/Documents/Work/Code/projects/upf`) 是一个基于云原生架构的 5G UPF (User Plane Function) 研发项目。本 `GEMINI.md` 为 Gemini CLI 提供核心项目背景与指令指引。

## 项目概览
- **名称:** upf
- **定位:** 高性能、高可靠、完全云原生的 5G 核心网数据面网元。
- **技术栈:** **Go (控制面)** + **VPP/DPDK (数据面)**，支持 K8s、SR-IOV 及无状态 HA。

## ⚠️ 核心准则：命名与同步
- **[命名]** 强制采用 **“领域-职责”** 命名法：
    - **Skill**: `[领域]-[职责]` (如 `arch-ha-dr`)。
    - **文档**: `[领域]-[主题].md` (如 `spec-srs.md`)。
    - **脚本**: `[领域]-[功能]` (如 `auto-req-sync.js`)。
- **[自愈]** **生产级自愈 (Production-Grade Self-Healing)**:
    - 所有 Agent 调用强制集成 `Execute-Heal-Retry` 循环。
    - 任何自愈动作必须记录至 `docs/05-quality/verification/healing-audit-log.md` 以供合规审计。
- **[SMART]** 所有 Skill 必须符合 SMART 原则，且包含强制性的 **“交付契约 (Delivery Contract)”**，明确交付件、质量门限、里程碑及评审角色。
- **[参考]** 编码实现必须遵循 **`docs/05-quality/spec-coding-standards.md`**。
- **[契约]** API 规格强制存储于 \`docs/02-design/api/\`:
    - \`external/\`: 3GPP 标准接口 (N3/N4/N6), Design-First。
    - \`internal/\`: 微服务接口, Code-First (Swag 驱动)。
- **[强制]** 任何对本 `GEMINI.md` 中研发阶段、Skill 映射或质量红线的修改，必须同步刷新 `README.md`。
- **[核实]** 在执行 `quality-gate` 前，必须确保 `auto-doc-check` 指令通过。

## 研发全生命周期执行地图 (IPD 专家版 v1.0)

为了确保端到端高质量交付，请根据研发阶段依次调用以下 Skill。

### 第一阶段：Charter 立项与需求基线 (Phase 1: Flow) - Driven by `agent-product`, `agent-pm` & `agent-se`
0. `mgr-ipd-charter` (Charter 移交) - **[agent-product]** 接收产品包商业目标、DFx特性诉求及投资回报预测，作为需求分析前提。
1. `flow-req-spec` (需求定义) - 定义 RR/IR。
2. **`mgr-ipd-oss-governance` (开源治理)** - 执行事前准入选型与 License 审核。
3. `flow-feat-trace` (特性追踪) - 维护 `docs/03-traceability/spec-rtm.md`。
4. `mgr-ipd-change` (变更管理) - 记录 `docs/04-management/spec-rcr.md`。
5. **`mgr-ipd-tech-review` (技术评审)** - 执行 **TR1 (需求基线评审)**。
6. **`mgr-ipd-dcp-decision` (决策评审)** - 执行 **CDCP (概念决策)**。
7. `flow-doc-coauthoring` (文档协同) - SRS/RTM 联合打磨。

### 第二阶段：架构设计与可靠性 (Phase 2: Arch) - Driven by `agent-se` & `agent-architect`
1. `arch-adr-decision` (架构决策) - 必须包含 Alternatives。
2. **`arch-hld-expert` (概要设计)** - 产出系统级 SDS，整合 4+1 视图与 DfX。
3. **`flow-feature-design` (特性设计)** - 针对复杂业务制定端到端技术方案。
4. **`arch-lld-expert` (详细设计)** - 针对子系统进行算法与数据结构建模。
5. **`arch-reliability-analysis` (可靠性分析)** - DFMEA/FTA 驱动。
6. **`arch-dfx-std` (卓越设计)** - 注入 DfS (易维)、DfR (稳健)、DfT (易测) 等商业化基因。
7. `arch-svc-decomp` (微服务拆分) & `dev-intf-internal` (接口解耦)。
8. **子系统详细设计 (arch-sub)**: 
   - 调用 `arch-sub-core-dp` (转发面) / `arch-sub-core-cp` (控制面) 进行模块细化。
   - 调用 `arch-sub-base-lib` (基础库) 等公共组件规约。
9. **`mgr-ipd-tech-review`** - 执行 **TR2 (方案评审, agent-se)** 与 **TR3 (设评, agent-architect)**。
10. **`mgr-ipd-dcp-decision`** - 执行 **PDCP (计划决策, agent-architect)**。

### 第三阶段：高质量开发与规约 (Phase 3: Dev) - Driven by `agent-dev`
0. **`flow-writing-plans` (研发蓝图专家)** - **[强制准入]** 所有编码实施前，必须将 SDS 拆解为原子级微任务并填入 `docs/04-management/spec-downstream-tasks.md`。
1. `dev-c-std` (C 规范) - **[硬性] snake_case, 大括号换行**。
2. `dev-go-ms-std` (Go 规范) - 遵循项目统一风格。
3. **`test-driven-development` (TDD)** - 全局 Skill 驱动。
4. `dev-sec-std` (安全开发) - 华为安全编码基线。
5. **`mgr-ipd-tech-review`** - 执行 **TR4 (详设与单元验证评审)**。

### 第四阶段：集成测试与质量闭环 (Phase 4: Test) - Driven by `agent-tester`
1. `test-quality-gate` (质量门控) - 执行 `make quality-gate`。
2. `test-issue-fix` (缺陷管理) - 闭环 `docs/05-quality/spec-qclm.md`。
3. **`mgr-ipd-tech-review`** - 执行 **TR4A (子系统联调评审)**，专用于 UPF 控制面与数据面的集成质量门限。
4. **`flow-rat-acceptance` (需求验收)** - 维护 **`docs/03-traceability/spec-rat.md`**，执行演示验收。
5. `test-webapp-testing` (Web 验证) - OAM 平面测试。
6. **`mgr-ipd-tech-review`** - 执行 **TR5 (集成验收评审)**。
7. **`mgr-ipd-dcp-decision`** - 执行 **ADCP (可获得性决策)**。

### 第五阶段：发布归档与运维 (Phase 5: Release) - Driven by `agent-product`, `agent-qa` & `agent-pm`
0. **`mgr-ipd-gtm` (GTM 治理)** - **[agent-product]** 执行产品规格审计与上市准备度检查，确保商业合规。
1. **`mgr-ipd-spec-archiving` (规格归档)** - 实测值回填。
2. **`mgr-ipd-oss-governance` (开源审计)** - 自动化 SBOM 提取与 CVE 漏洞扫描。
3. **`mgr-ipd-tr-audit` (评审自动化)** - 执行 `make tr-audit`。
4. **`mgr-ipd-release` (版本管理)** & `mgr-cicd-pipe` (流水线)。
5. `mgr-obs-std` (可观测性) / `ops-sop-manual` (故障处理)。
6. **`mgr-ipd-tech-review`** - 执行 **TR6 (发布决策评审)**。

## 角色化治理与 Agent 协同
项目采用基于角色的门控系统，确保操作的原子性与合规性：

---
### 质量红线
- **代码物理纠偏**: 任何代码提交必须通过 `make fix-all` 自动格式化。`pre-commit` 钩子将物理拦截任何不符合 C/Go/JS 规范的源码入库。
- **风险闭环管理**: 每次里程碑切换前必须刷新 `docs/04-management/spec-risk-register.md`。
任何状态为 **"Critical"** 的风险项未闭环前，TR 评审具有一票否决权。

- **重构质量红线**: 大规模重构必须由 **ARCHITECT** 执行评审，重构后必须通过全量单元测试与 `make quality-gate` 审计。
- **角色越权拦截**: 任何里程碑切换 (`stage-next`) 或决策通过 (`decision-pass`) 指令必须通过 **`MAINTAINER`** 角色的身份校验。
- **流程变革管控**: 流程的变更必须在 **`mgr-ipd-qa-expert`** 的指导下进行，QA 负责对流程运作结果进行全量监控与定期审计。
- **QA 独立审计**: 任何 TR 评审前，必须由 **`mgr-ipd-qa-expert`** 执行流程合规性审计。
- **RTM 一致性**: 100% 的代码实现必须可回溯至 SRS 编号。
- **TR 拦截**: 任何 TR 评审结论为 "No-Go" 或存在未闭环 QA Issue 时，禁止进入下一阶段。
- **DCP 拦截**: 未经 DCP 批准的项目禁止启动物理资源的采购或大规模版本投入。
- **OSS 准入**: 禁止引入具有传染性协议 (GPLv3/AGPL) 的开源组件。
- **缺陷零清零**: TR5 之前，所有 P0/P1 级 Issue 必须处于 Closed 状态（详见 `docs/05-quality/spec-qclm.md`）。

## 🌟 研发引擎状态 (R&D Engine Status)

**当前状态：基于 LangGraph 驱动的智能化流转与状态机已上线 (v2.0)**

### 核心价值 (Why LangGraph?)
1. **统一状态图 (State Graph)**: 已将里程碑流转、RTM 矩阵校验及物理资产审计收敛为 `scripts/core/langgraph-upf-orchestrator.js` 的核心 State 对象，实现了内存级高效流转。
2. **非线性自愈 (Cyclic Healing)**: 突破了线性脚本限制。当物理网关（Design Gate）拦截时，利用**条件边 (Conditional Edges)** 自动回跳至架构设计节点（Architect Node）进行物理资产补齐，实现自动闭环。
3. **多维博弈 (Multi-Agent Debate)**: 在 TR/DCP 节点，支持 `PRODUCT`、`ARCHITECT` 和 `QA` 等角色通过 Graph 进行上下文对等博弈，确保决策的原子性与合规性。

### 使用指南
- **启动编排**: `make ipd-orchestrator` (驱动全生命周期图流转)
- **状态查看**: `node scripts/tools/mgr-state-viewer.js` (查看 SQLite 物理存证与快照)

---
*Generated by Gemini CLI. 研发流程已升级为图驱动模式。*

---
> Source: [wangchong2023/upf](https://github.com/wangchong2023/upf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
