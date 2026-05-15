## openvera

> 以下文件/目录包含本地密钥或临时数据，**任何情况下都不得提交**：

# Vera — 项目约束

## 敏感文件保护

以下文件/目录包含本地密钥或临时数据，**任何情况下都不得提交**：

| 路径 | 原因 |
|---|---|
| `.vera/settings.json` | LLM API Key，已 gitignore |
| `.qwen/` | Qwen 本地配置，已 gitignore |
| `.claude/settings.local.json` | Claude Code 本地设置，已 gitignore |
| `.claude/worktrees/` | 临时 worktree，已 gitignore |
| `.gemini/` | Gemini 本地配置，已 gitignore |
| `*.orig` | 合并/备份临时文件 |

**规则：**
- 提交前检查 `git status`，确认上述文件不在 staged 中
- 不要 `git add .` 或 `git add -A` 无脑全加，按文件/目录选择性添加
- 如果 `git status` 显示上述文件有修改，用 `git restore` 丢弃或保持 unstaged
- 写 `.vera/settings.json` 时永远用 `settings.example.json` 的占位符，不要填入真实 Key
- 永远不要在 CLAUDE.md、README、代码注释、commit message 中粘贴 API Key

## 项目架构

Vera = Harness 为内核的 agent runtime。两层结构：

- **Core** (`packages/core`)：单次 LLM 调用闭环。adapter → loop → tool → result。不感知 Harness。
- **Harness** (`packages/harness`)：多步 workflow。ExecutionPlan 状态机 → Flow State → Critique → Replan。依赖 Core。

依赖方向：`harness → core`，Core 不可 import Harness。

## 技术栈

- TypeScript ESM（`module: "nodenext"`）
- pnpm workspace monorepo
- React + Ink（REPL UI）
- 默认 LLM adapter：Anthropic Claude API

## 提交风格

- 英文 commit message
- `feat:` / `fix:` / `docs:` / `chore:` 前缀
- 单次提交聚焦一个功能模块

## 文档

- 入口：`docs/README.md`
- 路线图：`docs/roadmap.md`
- 变更日志：`docs/changelog.md`（入口索引）+ `docs/changelog/<YYYY-MM-DD-HH>.md`（详细）
- 当前 P0 进度：全部完成 ✅（intent routing / tool runtime / rendering / session / infinite context / plan mode / critique / flow control）

## 提交前检查清单

提交代码前必须按顺序完成以下步骤：

### 1. 测试与质量

- **覆盖率 ≥ 90%**：运行 `pnpm --filter @vera/core run test:coverage`，确认 lines 覆盖率不低于 90%
- **无 error 级别质量问题**：运行 `bash .claude/skills/quality-scan/scan.sh`，oxlint / sonarjs 不允许任何 `error` 级别发现（warning 可接受）
- 新增业务逻辑必须有对应 unit test；纯类型定义、配置文件、文档除外

### 2. Roadmap 同步

- 本次提交完成了 roadmap 中的某项能力 → 在 `docs/roadmap.md` 对应条目标记 ✅
- 发现新的遗留问题或技术债 → 追加到 roadmap 的"已知缺陷与技术债"或"P0 后对齐项"对应分区

### 3. Changelog 更新

- 在 `docs/changelog.md` 索引表追加一行（`YYYY-MM-DD · HH:xx` + 一句话摘要 + 链接）
- 在 `docs/changelog/<YYYY-MM-DD-HH>.md` 写入本批次详细记录，格式：
  - **变更**：commit 表格（hash / 模块 / 内容）
  - **Roadmap 同步**：标记了哪些条目
  - **遗留事项**：本次未完成、已知待修复的问题

## Skills 维护规则

项目 skills 位于 `.claude/skills/`，索引文档为 `.claude/skills/README.md`。

**每次新增或修改 skill 后，必须同步更新 `.claude/skills/README.md`**，包括：
- 新增：在对应分类下追加条目（名称、描述、数据源、输出路径、用法示例）
- 修改：更新受影响的描述或参数说明
- 删除：移除对应条目并标注废弃原因（若有替代品）

当前 skills：`agent-changes-report` · `claude-session-review` · `cursor-session-review` · `quality-scan`

---
> Source: [open-vera/OpenVera](https://github.com/open-vera/OpenVera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
