## daidaihutao

> 1. `main` 是唯一可信基线，工作分支必须从 `main` 同步。

# 核心规则

1. `main` 是唯一可信基线，工作分支必须从 `main` 同步。
2. 工作分支之间禁止直接同步，所有合并必须经过 `main`。
3. 合并回 `main` 必须使用 `--ff-only`，不得强制推送 `main`。
4. 每个阶段结束必须推送远程分支，确保工作进度不丢失。
5. Claude 仅修改 `docs/` 目录及 `PROJECT_LESSONS.md`、`PROJECT-SPEC.md`；Executor 不得修改上述范围。其他变更通过任务文档走 Executor 流程。
6. 所有 commit 必须使用 `[Agent][Pxxx]` 格式前缀（歧义问题使用 `[Qxxx]`）。
7. Review 结论只能是 PASS 或 FAIL，必须附带逐项验证清单。
8. 如存在重大歧义或高风险操作，向用户确认。
9. 优先做最小必要修改，不做无关重构。
10. 所有输出尽量清晰、可复查、可复用。
11. 【受控全自动授权】Executor 在阶段二拥有受控自动执行权。只要不违反目录权限、分支规则、`--ff-only` 合并规则和任务文档要求，应从读取任务、修改代码、测试验证、commit、push 到合并回 main 连续自动执行，不得中途询问用户。
12. 【错误降级】阶段二只有在以下情况允许中断并向用户求助：`git merge --ff-only` 失败、任务文档存在重大歧义、操作会越过允许修改目录、需要访问敏感信息或付费服务、测试/构建错误经合理修复后仍无法解决。
13. 【禁止反思提问】Executor 不得就"如何修改更好""是否确认提交""是否确认推送"向用户提问。一切以 `docs/tasks/Pxxx.md` 为最高执行依据。

## Agent 协作分工

### 角色定位

| Agent | 工作分支 | 允许修改目录 | 职责 |
|---|---|---|---|
| Claude | `claude/review` | `docs/`、`PROJECT_LESSONS.md`、`PROJECT-SPEC.md` | 规范制定、任务拆解、Review 审查、归档 |
| Executor | `executor/work` | 除 `docs/`、`PROJECT_LESSONS.md`、`PROJECT-SPEC.md` 外的所有项目目录 | 执行任务文档 |

#### Executor 参考名单

以下 Agent 可作为 Executor 使用 `executor/work` 分支执行任务：

| Agent | 擅长领域 |
|---|---|
| Codex | 代码生成、批量修改、测试与调试 |
| MiMoCode | 代码生成、批量修改、测试与调试 |
| DeveCo | 代码生成、批量修改、测试与调试 |
| Qoder | 代码生成、批量修改、测试与调试 |
| 其他 | 任何符合 Executor 权限规则的 Agent 均可接入 |

### 完整工作流（含远程推送）

#### 阶段一：Claude 规划

```
git checkout main
git fetch origin
git pull --ff-only origin main
git checkout claude/review
git rebase main
→ 创建任务文档 docs/tasks/Pxxx.md
→ git commit -m "[Claude][Pxxx] add task specification"
→ git push origin claude/review
→ git checkout main
→ git merge --ff-only claude/review
→ git push origin main
→ 等待指令
```

#### 阶段二：Executor 执行

```
git checkout main
git fetch origin
git pull --ff-only origin main
git checkout executor/work
git rebase main
→ 读取 docs/tasks/Pxxx.md
→ 严格按任务文档执行修改（无需中途停顿确认）
→ git commit -m "[<Agent名>][Pxxx] implement changes"
→ git push origin executor/work
→ git checkout main
→ git merge --ff-only executor/work
→ git push origin main
→ 自动进入静默状态，仅提示用户："阶段二已自动执行完毕，已推送至 main，等待 Claude 进行阶段三 Review。"

注：<Agent名> 为实际执行 Agent 的身份标识（如 Codex、MiMoCode、DeveCo、Qoder 等），
    commit 前缀用于追溯执行者身份，不影响分支共享机制。
```

#### 阶段三：Claude Review

##### 创建并合并 Review

```
git checkout main
git fetch origin
git pull --ff-only origin main
git checkout claude/review
git rebase main
→ 创建 docs/reviews/Pxxx-review.md
→ git commit -m "[Claude][Pxxx] review pass" 或 "review fail"
→ git push origin claude/review
→ git checkout main
→ git merge --ff-only claude/review
→ git push origin main
```

##### PASS → 归档

```
git checkout main
git fetch origin
git pull --ff-only origin main
git checkout claude/review
git rebase main
→ git mv docs/tasks/Pxxx.md docs/archive/
→ git mv docs/reviews/Pxxx-review.md docs/archive/
→ git commit -m "[Claude][Pxxx] archive completed task"
→ git push origin claude/review
→ git checkout main
→ git merge --ff-only claude/review
→ git push origin main
```

##### FAIL → 修复循环

```
FAIL 包含：发现的问题、影响范围、修复建议

Executor 拉取最新 main、修复、合并回 main：
git checkout main
git fetch origin
git pull --ff-only origin main
git checkout executor/work
git rebase main
→ 根据 Review 修复
→ git add .
→ git commit -m "[<Agent名>][Pxxx] fix review issues"
→ git push origin executor/work
→ git checkout main
→ git merge --ff-only executor/work
→ git push origin main
→ Claude 重新 Review

循环模型：
Task → Agent 执行 → Claude Review → FAIL → Agent 修复 → Claude Review → ... → PASS → 归档
```

### 提交前缀

| 用途 | Agent | 格式 |
|---|---|---|
| 任务文档 | Claude | `[Claude][Pxxx] add task specification` |
| 执行修改 | 任意 | `[<Agent名>][Pxxx] implement changes` |
| 修复 Review 问题 | 任意 | `[<Agent名>][Pxxx] fix review issues` |
| Review 通过 | Claude | `[Claude][Pxxx] review pass` |
| Review 失败 | Claude | `[Claude][Pxxx] review fail` |
| 规范更新 | Claude | `[Claude][Pxxx] update project specification` |
| 归档 | Claude | `[Claude][Pxxx] archive completed task` |
| 歧义问题 | 任何 Agent | `[Qxxx] request clarification` |

### 推送前规范一致性审查

**每次合并到 `main` 后、推送 `main` 到 `origin/main` 之前**，Claude 必须审查以下三份文档是否与当前项目最新代码一致：

#### 1. PROJECT-SPEC.md

- 任务涉及的配置路径变更是否已反映在规范中
- Agent 分工描述（允许修改/限制目录）是否仍然准确
- 文档结构是否与实际 `docs/` 目录一致
- 项目结构（目录树）是否与实际一致
- 分支模型描述是否仍然适用
- `.gitignore` 等基础设施变更是否已同步到规范

#### 2. PROJECT_LESSONS.md

- 本次任务的复盘结果是否已记录
- 已有经验教训是否仍准确有效

#### 3. README.md

- 使用说明、项目介绍、配置示例等是否与实际一致

如发现过时或不一致内容，Claude 应在 `claude/review` 分支更新对应文档，然后重新合并到 `main`，再推送：

```bash
git checkout claude/review
git rebase main

# 更新文档
git add <文件路径>
git commit -m "[Claude][Pxxx] update <文件名> before push"

git checkout main
git merge --ff-only claude/review
git push origin main
```

未经此审查不得推送 `main` 到 `origin/main`。

<!-- superpowers-zh:begin (do not edit between these markers) -->
# Superpowers-ZH 中文增强版

本项目已安装 superpowers-zh 技能框架（20 个 skills）。

## 核心规则

1. **收到任务时，先检查是否有匹配的 skill** — 哪怕只有 1% 的可能性也要检查
2. **设计先于编码** — 收到功能需求时，先用 brainstorming skill 做需求分析
3. **测试先于实现** — 写代码前先写测试（TDD）
4. **验证先于完成** — 声称完成前必须运行验证命令

## 可用 Skills

Skills 位于 `.claude/skills/` 目录，每个 skill 有独立的 `SKILL.md` 文件。

- **brainstorming**: 在任何创造性工作之前必须使用此技能——创建功能、构建组件、添加功能或修改行为。在实现之前先探索用户意图、需求和设计。
- **chinese-code-review**: 中文 review 沟通参考——话术模板、分级标注（必须修复/建议修改/仅供参考）、国内团队常见反模式应对。仅在用户显式 /chinese-code-review 时调用，不要根据上下文自动触发。
- **chinese-commit-conventions**: 中文 commit 与 changelog 配置参考——Conventional Commits 中文适配、commitlint/husky/commitizen 中文模板、conventional-changelog 中文配置。仅在用户显式 /chinese-commit-conventions 时调用，不要根据上下文自动触发。
- **chinese-documentation**: 中文文档排版参考——中英文空格、全半角标点、术语保留、链接格式、中文文案排版指北约定。仅在用户显式 /chinese-documentation 时调用，不要根据上下文自动触发。
- **chinese-git-workflow**: 国内 Git 平台配置参考——Gitee、Coding.net、极狐 GitLab、CNB 的 SSH/HTTPS/凭据/CI 接入差异与镜像同步配置。仅在用户显式 /chinese-git-workflow 时调用，不要根据上下文自动触发。
- **dispatching-parallel-agents**: 当面对 2 个以上可以独立进行、无共享状态或顺序依赖的任务时使用
- **executing-plans**: 当你有一份书面实现计划需要在单独的会话中执行，并设有审查检查点时使用
- **finishing-a-development-branch**: 当实现完成、所有测试通过、需要决定如何集成工作时使用——通过提供合并、PR 或清理等结构化选项来引导开发工作的收尾
- **mcp-builder**: MCP 服务器构建方法论 — 系统化构建生产级 MCP 工具，让 AI 助手连接外部能力
- **receiving-code-review**: 收到代码审查反馈后、实施建议之前使用，尤其当反馈不明确或技术上有疑问时——需要技术严谨性和验证，而非敷衍附和或盲目执行
- **requesting-code-review**: 完成任务、实现重要功能或合并前使用，用于验证工作成果是否符合要求
- **subagent-driven-development**: 当在当前会话中执行包含独立任务的实现计划时使用
- **systematic-debugging**: 遇到任何 bug、测试失败或异常行为时使用，在提出修复方案之前执行
- **test-driven-development**: 在实现任何功能或修复 bug 时使用，在编写实现代码之前
- **using-git-worktrees**: 当需要开始与当前工作区隔离的功能开发，或在执行实现计划之前使用——通过原生工具或 git worktree 回退机制确保隔离工作区存在
- **using-superpowers**: 在开始任何对话时使用——确立如何查找和使用技能，要求在任何响应（包括澄清性问题）之前调用 Skill 工具
- **verification-before-completion**: 在宣称工作完成、已修复或测试通过之前使用，在提交或创建 PR 之前——必须运行验证命令并确认输出后才能声称成功；始终用证据支撑断言
- **workflow-runner**: 在 Claude Code / OpenClaw / Cursor 中直接运行 agency-orchestrator YAML 工作流——无需 API key，使用当前会话的 LLM 作为执行引擎。当用户提供 .yaml 工作流文件或要求多角色协作完成任务时触发。
- **writing-plans**: 当你有规格说明或需求用于多步骤任务时使用，在动手写代码之前
- **writing-skills**: 当创建新技能、编辑现有技能或在部署前验证技能是否有效时使用

## 如何使用

当任务匹配某个 skill 时，使用 `Skill` 工具加载对应 skill 并严格遵循其流程。绝不要用 Read 工具读取 SKILL.md 文件。

如果你认为哪怕只有 1% 的可能性某个 skill 适用于你正在做的事情，你必须调用该 skill 检查。
<!-- superpowers-zh:end -->

---
> Source: [huyanluanyuqaq/daidaihutao](https://github.com/huyanluanyuqaq/daidaihutao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
