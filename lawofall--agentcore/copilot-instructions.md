## dev-process

> AI 开发工作流——新会话冷启动地图、角色分工、决策权限、设计纪律、编辑安全


# 开发工作流

项目采用「AI 全程开发、人做决策」模式，多 Agent 并行开发（不同模块同时推进）。

## 新会话冷启动

AgentCore = 面向大众的 Multi-Agent AI 工作台，核心差异是**真正的 Agent 团队协作**（非「单 Agent + 子任务派发」）。Monorepo：`apps/server`（FastAPI · runtime 执行引擎 · LLM 网关）+ `apps/desktop`（Electron+React）+ `apps/mobile` + `apps/admin` + `packages/`（跨端契约）。

定位任务/找文档 → 先读 `docs/索引.md`「任务路由」表，别盲 grep；跑通项目 → `docs/02-架构/本地开发.md`；接缝/漂移排查 → `seam-audit.mdc`；桌面前端自检渲染 → `frontend-preview.mdc`；沉淀可复用经验 → `memory.mdc`。

## 决策权限

| AI 自主决定 | AI 提案 → 人确认 | 人直接决定 |
|---|---|---|
| 实现细节、Bug 修复（不改契约）、测试/重构 | 架构选型、新依赖、设计文档修改 | 功能范围、需求变更、发布决策 |

遇需人决策的节点，立即暂停并清晰描述问题和备选方案。

## 设计纪律

- 正向推导：**用户需求 → 产品设计 → 技术实现**，禁止用「技术方便」反推需求；动手前答三问：**谁用 / 解决什么真实问题 / 产品上如何呈现**。
- 出现「为复用旧实现而裁剪需求」「实现冒充需求」→ 立即停，回到三问或提报决策。
- **开发期无真实数据**：讨论/优化质量前先声明「目前无真实数据，以下为假设」，以 `conformance/vectors/`、`evals/` 合成样本为据，别把不存在的真实数据当既有事实。

## 补丁绊线

修复满足任一，**先停**、提根因重设计提案：1. 需新增兜底/对账/自愈/特例逻辑；2. 同一根因改 ≥2 层；3. 同一接缝第 3 次打补丁。

硬/软拦截提案纪律（禁默认加闸、误伤优先）→ `intercept-discipline.mdc`。

## 开发准则

生产级代码（完整错误处理、类型标注、合理日志）；优先写接口定义（Protocol/ABC/Pydantic）再实现；模块间经抽象接口解耦，共享类型放 `shared/` 或 `core/types/`；核心逻辑必须有单元测试；子代理委派规范见 `multitask.mdc`。

## 文件编辑安全

禁止用脚本批量改源码（易静默损坏文件）；用 StrReplace + `replace_all` 逐文件改；改完用 **Read 读文件**自检（不应出现大量删除/清空）。

## 并行开发 Git 约束

`git status` / `git diff` 不代表本会话的真实改动：1. 校验自己的改动用 **Read**；2. git 输出对不上是正常噪音（其他并行 worktree），一律接受、继续任务；3. 禁止排查 git 内部（worktree 嵌套/路径大小写/配置损坏）；4. 仅用户明确要求才跑 git 命令，仍遵守「非请勿 commit」。

## 产品 AI 日志查询

用户给出 32 位 hex ID / UUID → **先读 `conversation-logs.mdc`** 再动手，禁止在文件系统（含 `agent-transcripts/`）搜索这类 ID。

文档治理规范见 `doc-governance.mdc`。

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
