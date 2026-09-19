## agentpulse

> > **动手前先读完本文件。** 它定义 AgentPulse 的产品方向、已拍板的架构决策、开发规范。

# AGENTS.md — 接手本项目的 AI / 开发者必读

> **动手前先读完本文件。** 它定义 AgentPulse 的产品方向、已拍板的架构决策、开发规范。
> 本文件是唯一的"北极星"。**任何偏离本文件方向的改动，动手前必须先跟项目所有者确认。**

---

## 0. 当前状态：重建中（2026-08-21 起）

本项目在 2026-08-21 由项目所有者决定**删除旧代码库、基于 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 重建**。

- 旧版（Hermes 运行时，71,968 行 / 47 张表 / 146 commits）已删，保留在 `main` 分支和 git 历史里。要看旧实现：`git show main:<path>`。
- 决策与实测事实见 **[ADR 0019](docs/decisions/0019-dsh-as-agent-runtime.md)（必读）**。
- 现有代码只有运行时种子：`services/api/`（见其 [README](services/api/README.md)）。

**为什么重建**：旧代码库一半的体量是在 Hermes 外面重建 Hermes 没提供的东西（会话账本、记忆、技能、审批传输、内置 runtime 打包）。dsh 原生提供这些。同时旧版四条战线并行、零发布 —— 重建的首要目的是**收敛到一条能发布的闭环**。

---

## 1. 北极星（产品方向 · 不可跑偏）

1. **用户是老板，不是系统管理员**：不把 prompt / schema / workflow DAG 丢给普通用户。
2. **协作模式 = 先拉群讨论，再分工执行**：要干一件事，先拉相关员工进群，**把背景/目标/分工讨论明白**，达成共识后才开工。**Agent 要像人——背景不清楚必须在群里发问，绝不允许被分配了就稀里糊涂开干。**
3. **自然语言捏 agent**：用户用一句话描述角色，系统生成不同技能的 AI 员工。
4. **每个 agent 自我学习、持续进步**：在工作中沉淀技能、积累记忆，越用越懂这家公司。
5. **没有 idle 员工**：空闲时也在想业务、攒 idea（独立的「idea 中心」）、学技能。7×24 不间断。
6. **群聊 + 私聊都要支持**：同一套会话引擎，参与者=2 就是私聊。

---

## 2. 架构决策（已拍板 · 改动前须新增 ADR）

### 分层
```
产品层        公司/部门/员工卡 · 自然语言捏 agent · 技能市场 · idea中心 · 看板
协作编排层    群讨论协议：共享 transcript + 发言路由 + 对齐门 + 人类拍板 + 终止条件
（自研·核心） 讨论达成 → 产出"共识 brief" → 才建 Task/Run
运行时层      每员工 = 一个 dsh runtime 进程 + 一份 composition
（dsh）       人格→dsh-system-prompt · 记忆/履历→dsh session 账本 · 审批→dsh-user-approval
执行/部署层   API 负责公司事实/调度；桌面端可运行本机 worker 访问用户授权项目
```

### 一句话
> **AgentPulse 自研「公司协作大脑」(拉群 → 讨论对齐 → 分工 → 拍板 → 追踪)；dsh 提供「每个员工的专业能力 + 记忆 + 自我进步」；两者靠 stdio JSON-RPC 对接。**

### 明确不做（DON'T — 防止后续 AI 跑偏）
- ❌ **不**以 Codex / Claude Code 这类"操作文件的工程师型" CLI 为基座。基座只用 dsh。
- ❌ **不**自建 Agent 协议 / Runtime / 工具系统（用 dsh 的）。
- ❌ **不**直连 DeepSeek 生成员工回复。所有员工输出只能走 dsh；DeepSeek 只是 dsh 内部可替换的模型供应商。
- ❌ **不**用 dsh 的 `experimental/agent-team` 替代自研群讨论协议（它只有 roster/mailbox/任务 DAG，**没有"先讨论对齐再开工"**）。见 [ADR 0002](docs/decisions/0002-self-built-group-discussion.md)。
- ❌ **不**指望 dsh 做 7×24 调度（`dsh-schedule` 是 session-local reminder，不是 daemon）。调度权在我们侧，见 [ADR 0003](docs/decisions/0003-server-side-24x7-idea-center.md)。
- ❌ **不**把 UnitPulse 路径隔离检查外包给 dsh sandbox。见 [ADR 0005](docs/decisions/0005-hermes-poc-safety-findings.md) + §5。

---

## 3. 关键已验证事实（P0 实测，勿重复造轮子 · 详见 [ADR 0019](docs/decisions/0019-dsh-as-agent-runtime.md)）

- **驱动 dsh**：官方 Python SDK（`deepseek-harness-sdk`），stdio newline-delimited JSON-RPC。**不用**内置 ACP（它不上报 reasoning/tool 活动，且 `session/new` 的 `mcpServers` 非空即拒）。
- **自写树外插件能加载进 bundled 二进制** —— `cordis.yml` 里 `name` 写插件文件绝对路径。这是选型的真闸门，已过。
- **两层 runtime**：bundled Python runtime（一进程一员工，无 MCP，无 Windows wheel）用于 P1 发布；npm 全量 dsh（`agent-presets` + MCP，一进程 N 员工）用于百人调度和 Windows。`RuntimeSpec` 抽象不许省 —— dsh 是 rc 版本，自承 breaking changes。
- **已踩过的坑**（ADR 0019 有完整列表）：`name` 位置不能用 `!!js`；Cordis 必须 `export const inject`；`session-persistence-sqlite` 要 `$.path` 不是 `root`；`permission-presets` 要求会约束的 bash 执行器；`strings` 抓到的插件名≠真装了的包。
- **人机桥已通**（[ADR 0022](docs/decisions/0022-human-bridge.md)）：`ctx.userQuestions.registerProvider()` + `ctx.on('approval/request', ...)`（waterfall）。桥不走 SDK wire，直接 HTTP 打回 AgentPulse。已真跑通：员工发问 → 老板回答 → 回答流回模型并被使用。

---

## 4. 下一步

**P1：一条闭环 + 一次发布（零服务器，见 [ADR 0021](docs/decisions/0021-employee-workstation-and-resource-leases.md)）。** 拉群 → 讨论对齐 → 员工请老板拍板 → 员工真执行（能动你本机的项目文件）→ 结果回群。

已完成：运行时（0019）· 人机桥（0022）· 房间（0023）· 落库（0024）· HTTP（0025）· 前端（0026）。
桌面壳与 dmg 已完成（[ADR 0027](docs/decisions/0027-desktop-shell-and-packaging.md)）。

BYOK 已完成（[ADR 0028](docs/decisions/0028-byok-model-credentials.md)）—— 需要的只有一把 DeepSeek key，
老板在界面里填、Fernet 加密存本机、明文不落库不出接口。

**P1 只剩两件**：
1. `v0.1.0` Release（push + 发布）。
2. 录屏验收（含手机端回答员工问题）。

⚠️ **凭证约定**：API key 不由 AI 代填。建入口让老板自己填，全程不接触明文。见 ADR 0028。

前端设计规范在根目录 [PRODUCT.md](PRODUCT.md)（方向/反参照）和 [DESIGN.md](DESIGN.md)（token/排版/布局）——
动界面前先读，尤其反参照那节。

出口物（硬性，缺一不可）：
1. 干净 macOS 机器上 dmg 跑通全程，录屏。
2. GitHub Release `v0.1.0`。

**不许在 P1 出口物完成前开第二条战线。** 旧版就是四条并行、零发布死掉的。

P1 之后：自然语言捏团队 → 业务工具门（需 npm 全量 runtime 拿 MCP）→ 技能/反思 → idea 中心 → 渠道 → 百人调度 → 云端。

---

## 5. 开发规范（必须遵守）

### Git 身份 —— ⚠️ 最容易出错
- 本仓库是 **Clycheng 的私有仓库**。提交署名**必须**是 `Clycheng <30332511+Clycheng@users.noreply.github.com>`（已配在本仓库 local config）。**绝不能用其他身份（尤其 unitpulse / UP 的邮箱）提交** —— 本机有多套 git 身份，别串。
- commit message 用**英文**、祈使句。

### 与 UnitPulse 零关联 —— ⚠️ 硬性原则，不是配置细节
- AgentPulse 和 UnitPulse（本机另一个真实项目）**必须零关联** —— 不只是 git 身份，执行环境、文件系统操作也不能有任何交集。项目所有者原话："他们可千万不能互相扯到一起。"
- **任何会真实执行进程/写文件系统的操作**（起 dsh、装依赖、跑测试…），ambient 工作目录**绝不能位于 UnitPulse 仓库/worktree 内**。见 [ADR 0005](docs/decisions/0005-hermes-poc-safety-findings.md)：2026-07-05 曾因会话 cwd 在 UnitPulse worktree 里，导致 agent 把测试文件写进了 UnitPulse 主仓库。被定性为不可接受的结构性风险。
- 理想做法：这类操作在 cwd 锚定 `/Users/liuxiajiang/Desktop/code/agentpulse` 的会话里做；任何 agent runtime 的 workdir 必须显式设为绝对路径。
- 代码层：`DshBackend` 拒绝相对 workdir。**这个检查不许删。**

### 声明"完成"前
- **跑起来看真实行为**（测试/截图/录屏），别空口说做完了。单测全过 ≠ 任务完成。
- 旧版有过真实事故：一个 worker AI 只看单测，实现了功能等效但绕过架构入口的死码，挂了两周才被发现。

### 做完后必须记录
1. **架构/方向级决策** → `docs/decisions/` 新增 ADR。
2. **相关文档过时了** → 同步更新（尤其本文件 §0/§2/§3）。

### 文档如何生长
- 本文件保持精简可扫（目标 <150 行），是"入口 + 规则 + 索引"；厚重内容放 `docs/`。
- 子项目专属规范 → 该目录放嵌套 `AGENTS.md`（就近生效）。

---

## 6. 文档索引

| 文件 | 内容 |
|---|---|
| [README.md](README.md) | 产品愿景（高层，稳定） |
| [AGENTS.md](AGENTS.md) | **本文件**：方向 + 架构 + 规范（接手第一读） |
| [ADR 0019](docs/decisions/0019-dsh-as-agent-runtime.md) | **dsh 重建决策 + P0 实测事实（第二读）** |
| [docs/decisions/](docs/decisions/) | 全部架构决策记录。0001/0004/0007/0015/0016 已被 0019 取代（保留作历史） |
| [services/api/README.md](services/api/README.md) | 后端现状与怎么跑 |

---
> Source: [Clycheng/AgentPulse](https://github.com/Clycheng/AgentPulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
