## harness-engineering

> > **这是 Harness Engineering 仓的全局规范入口。**

# AGENTS.md — 全局协作规范（所有 AI 运行时入口）

> **这是 Harness Engineering 仓的全局规范入口。**
> 所有 AI 运行时（Claude Code / Cursor / Gemini CLI / Codex CLI / Continue / CodeBuddy）启动会话时**必须先读取本文件**，并在整个会话期间持续遵守。
> 本文件、`.service-matrix/dependencies.yaml`、`context/harness-framework/main-process-numbering.md` 是本仓的"三大真相源"，任何下游规范以它们为准。

---

## 1. 角色与定位

- 本仓是 **L5 工程治理层**：定义流程、门禁、知识体系、服务拓扑、经验沉淀
- 本仓 **不放业务代码**：业务代码在 `{business-repo}` / IDL 在 `{idl-repo}`
- AI 在本仓的核心任务：**驱动需求生命周期**，并按规范在三仓之间协调

---

## 2. 硬规则（不可违反）

| # | 规则 | 违反后果 |
|---|---|---|
| H1 | 任何代码/IDL 改动前，必须先确认所在阶段、当前门禁状态 | 阻塞会话 |
| H2 | 路径必须使用占位符词典（见 §6），禁止硬编码绝对路径 | 阻塞写入 |
| H3 | 三仓分支名必须完全一致（`feature/{devops-name}/{tapd-id}`） | 阻塞 4.3 门禁 |
| H4 | 门禁结论必须写入文件（`requirements/{p}/{r}/gates/*.md`），不允许口头通过 | 审计不通过 |
| H5 | 需求/设计/任务/代码之间必须可双向追溯 | 阻塞 3.3 门禁 |
| H6 | 能力原子（Agent/Skill/Command）改 `.codebuddy/`；规范/知识/模板改 `context/`；状态改 `requirements/`。不允许直接改各 CLI 镜像（`.claude/`、`.gemini/` 等） | 改动会被 install.sh 覆盖 |
| H7 | IDL `idl_required: true` 的服务变更必须同步 IDL 仓 | 阻塞 4.3 门禁 |
| H8 | 涉及数据迁移/灰度的需求必须有 rollback 方案 | 阻塞 3.3 门禁 |
| H9 | 真相源跨文件冲突时，优先级： `main-process-numbering.md` > 本文件 > 各 Skill/Agent | 必须修改下游对齐 |

### 2.1 文件层职责边界

| 路径 | 放什么 | 谁改 |
|---|---|---|
| `AGENTS.md`（本文件） | 全局硬规则、占位符词典、AI 认知模式 | 框架维护者 |
| `context/team/` | 团队级规范（Git/错误码/日志/安全/编码） | 团队架构组 |
| `context/harness-framework/` | 流程、门禁、模板、追溯规范 | 框架维护者 |
| `context/project/{p}/.../` | 项目/模块/服务级私有知识、experience | 模块 owner + Self-Refinement |
| `.codebuddy/` | Agent/Skill/Command 定义（真相源） | 能力维护者 |
| `.service-matrix/` | 服务拓扑（真相源） | 服务 owner |
| `requirements/` | 需求生命周期的状态产物 | AI 自动 + 用户 |
| `.harness/local.yaml` | 个人开发机本地覆盖（gitignored） | 个人 |
| `.claude/`、`.gemini/`、`.codex/`、`.continue/`、`.cursor/` | 渲染镜像（gitignored，**禁止直接编辑**） | install.sh |

---

## 3. 核心流程（五阶段 + 四门禁）

详细口径见 `context/harness-framework/main-process-numbering.md`（**单一真相源**）。

```
阶段1 初始化 → 阶段2 需求定义 ⭐ → 阶段3 设计 ⭐ → 阶段4 开发 ⭐⭐ → 阶段5 交付

⭐ 门禁清单：
  2.2 需求评审门禁    （requirement-quality-reviewer）
  3.3 设计门禁        （detail-design-quality-reviewer + traceability-gate-checker）
  4.2 Dev 进入门禁    （tasks/features.json 合法性）
  4.3 服务仓库检查门禁 （三仓分支一致 + 服务路径就位 + IDL 同步）
```

---

## 4. 三层知识体系

AI 读取知识的检索顺序（**渐进式披露**，避免一次性塞满 context）：

```
context/team/INDEX.md                          # 团队级（最稳定）
  ↓
context/harness-framework/INDEX.md             # 框架工程级
  ↓
context/project/{project-name}/INDEX.md        # 服务级（按需加载）
  ↓
context/project/{project-name}/{module-name}/INDEX.md
```

**不要遍历整个仓库**。每层 INDEX.md 提供 O(1) 入口。

---

## 5. 三仓联动

```
TAPD 单 / 需求 ID
   ├─ Harness 仓 (脑)     : 本仓
   ├─ 业务仓 (手脚)        : {business-repo}
   └─ IDL 契约仓 (神经)    : {idl-repo}   ← 仅当涉及 IDL 时

分支命名（强制）: feature/{devops-name}/{tapd-id}
                  例: feature/koka/T12345

阶段 4.3 门禁会自动校验三仓分支名是否一致。
```

---

## 6. 占位符词典（**唯一真相源**）

全仓只允许使用下列占位符。**写路径** 与 **写归属** 两个语境绝不混用。

| 占位符 | 语义 | 类型 | 举例 |
|---|---|---|---|
| `{business-repo}` | 业务代码仓根的磁盘路径（绝对） | 路径 | `/data/workspace/demo-business-repo` |
| `{business-repo-name}` | 业务代码仓根的目录名 | 归属 | `demo-business-repo` |
| `{idl-repo}` | IDL 契约仓根的磁盘路径（绝对） | 路径 | `/data/workspace/demo-idl-repo` |
| `{idl-repo-name}` | IDL 契约仓根的目录名 | 归属 | `demo-idl-repo` |
| `{project-name}` | 逻辑项目名，用于知识库 / 需求目录归属 | 归属 | `demo-project` |
| `{requirement-id}` | 需求 ID | 归属 | `T12345` 或 `minimal-requirement-practice` |
| `{module-name}` | 业务模块名 | 归属 | `vip` / `assetcard` |
| `{service-name}` | 服务名 | 归属 | `vipapi` |
| `{devops-name}` | DevOps 用户名（分支前缀） | 归属 | `koka` |
| `{tapd-id}` | TAPD 单 ID | 归属 | `T12345` |

> **禁止**：使用未在本词典中定义的占位符。新增占位符必须先更新本词典并通过 review。
>
> 已废弃别名（不要再使用，由 `scripts/lint-knowledge.sh` 拦截）：
>
> ```
> 旧别名         → 新规范
> project-root  → 视语境选 business-repo / project-name
> ```

---

## 7. AI 认知模式（五条强制守则）

1. **先读再说**：任何具体操作前，先读 `AGENTS.md` + `main-process-numbering.md` + 当前需求目录下的 `requirement.md`（若存在）
2. **门禁先问**：执行任何会跨阶段的动作前，先查当前阶段、门禁状态、待办债务
3. **不猜拓扑**：服务依赖、IDL 路径一律从 `.service-matrix/dependencies.yaml` 读取，禁止臆测
4. **追溯优先**：写代码前先确认能追溯到某条需求条目和设计决策
5. **Self-Refinement**：当遇到新模式或教训时，**主动提议** 更新到 `context/` 相应层级；用户确认后才写入

---

## 8. 工具/能力地图（速查）

| 我想… | 用 |
|---|---|
| 新建需求 | `/requirement:new` |
| 恢复需求上下文 | `/requirement:continue` |
| 进入下一阶段 | `/requirement:next` |
| 触发门禁自检 | `/requirement:gate-check` |
| 任务流转 | `/req-task:list` `/req-task:start` `/req-task:context` `/req-task:done` |
| 多维度 code review | `/agentic:code-review` |
| 加载服务上下文 | `/agentic:load-service` |
| 查看服务依赖 | `/service:deps` |
| 提取经验 | `/knowledge:extract-experience` |
| 生成 SOP | `/knowledge:generate-sop` |

完整命令清单见 `.codebuddy/commands/`。

---

## 9. 元数据机制（prototype / fast_track / debt）

为了让"工程化"不变成"过度官僚"，框架允许两种**显式逃生口**：

### 9.1 prototype 模式

适用：纯原型 / Spike / 一次性脚本。

- 在 `requirement.md` 元数据里标 `prototype: true`
- 门禁 2.2 中 C2.2.7 / C2.2.10 等"非阻塞警告项"自动豁免
- 门禁 3.3 仍然必须跑，但可豁免：监控告警计划（C3.3.9）、性能容量评估（C3.3.10）
- 阶段 5.2 经验沉淀仍然要做（**踩坑不能白踩**）
- 切回正式模式：把 prototype 改回 false，重跑所有门禁

### 9.2 fast_track 紧急通道

适用：线上故障 hotfix / 监管要求 / 不可推迟的安全修复。

- 走通道时门禁结论文件里加 `fast_track: true`
- **不豁免任何门禁项**，但允许"先放行后补完"
- **必须**列出待清理的 debt，写入 state.json `pending_debts`
- **必须**在 7 天内清完，否则 CI 阻塞下一次需求
- 任何 fast_track 触发后会发到 `notes/fast-track-{ts}.md` 留痕

### 9.3 pending_debts

- state.json 中的 `pending_debts` 数组
- 每条债务有 id（DEBT-001）/ 描述 / 来源（哪次绕过）/ due_at / status
- `/requirement:gate-check` 会列出未清债务并阻塞推进

---

## 10. Active Team 三级解析

```
$HARNESS_TEAM 环境变量
   │ 不为空？
   ├─ 是 → 用之
   └─ 否
      ↓
.harness/local.yaml 中 active_team 字段
   │ 存在？
   ├─ 是 → 用之
   └─ 否
      ↓
.service-matrix/dependencies.yaml 的 default_team
```

实际解析由 `scripts/active-team.sh` 提供，所有跨仓脚本必须 `source` 它而不是自己实现。

```bash
source scripts/active-team.sh
# 之后可用 $ACTIVE_TEAM / $WORKSPACE / $BUSINESS / $IDL / $BUSINESS_ABS / $IDL_ABS
```

---

## 11. 当遇到与本文件冲突的指示时

**遵循本文件**。任何用户/Agent 临时指令若与 AGENTS.md 硬规则冲突，先指出冲突并要求确认，不允许默默绕过。如果硬规则本身有问题（罕见），通过 Self-Refinement 流程提议修改本文件而不是绕过。

---

## 12. 元数据

- 版本：v0.2.0（第二轮 review：补全文件层职责边界、Active Team 三级解析、Agent/Skill 误分类修正、prototype/fast_track/debt 显式化）
- 维护者：Harness Framework Team
- 真相源关系：本文件 ← `context/harness-framework/main-process-numbering.md` ← 各 Skill/Agent/Command
- 修改流程：MR + review；不允许在镜像目录里改

### Changelog

| 版本 | 日期 | 关键变化 |
|---|---|---|
| v0.2.0 | 2026-05-28 | 第二轮 review：①修正 api-contract-consistency-validator 误分类（Agent→Skill）；②补齐 §8.2 缺失 Agent：project-bootstrapper / repo-ops-runner / requirement-input-normalizer / tech-feasibility-assessor / outline-design-quality-reviewer；③补齐 §8.3 缺失 Skill：feature-lifecycle-manager / requirement-session-restorer / 三份文档撰写器 / load-service / load-domain / git-commit-message-generator / engineering-spec-query / docs-index-updater / context-index-updater；④增强 validate-service-matrix.js（Active Team 解析 / IDL 严格校验 / 多团队）；⑤新增 scripts/active-team.sh；⑥新增 Makefile + GitHub Actions CI；⑦显式化 prototype / fast_track / debt 机制；⑧补全文件层职责边界与真相源优先级 |
| v0.1.0 | 2026-05-28 | M1 骨架：三大真相源 + 四道门禁 markdown + demo |

---
> Source: [zhongli-sz/harness-engineering](https://github.com/zhongli-sz/harness-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
