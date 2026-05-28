## ai-coding-os

> 本仓库是 AI Coding Project OS 方法套件、agent skills 和 `goal-diffusion` CLI 的源码仓库。

# Agent 协作规则

本仓库是 AI Coding Project OS 方法套件、agent skills 和 `goal-diffusion` CLI 的源码仓库。

## 沟通与文档语言

- 始终用中文回复用户。
- `README.zh-CN.md`、`docs/**`、Goal Pack 叙述性正文优先使用中文。
- YAML 字段名、CLI 命令、代码符号、协议名、schema 示例和可复制模板可保留英文。
- 不写情绪价值开场白；分歧时按事实和约束说明。

## 当前顶层口径

AI Coding Project OS 面向高智能 agent，不面向弱模型防御式流程。

默认用户入口是 `$ai-coding-project-os`。它是轻量路由 skill，不拥有持久 artifact；按用户意图路由到：

- `$goal-diffusion`：目标计划、Goal Pack、跨会话延续、长期执行。
- `$docs-governance`：docs 分层、权威放置、文档清理和审计。
- `$headless-product-harness`：headless proof command、smoke evidence、fixture/replay、证据 envelope。
- inline work：一轮内可完成且有明确验证路径的小改动。

Goal Diffusion 是本套件的目标计划和滚动执行载体。只要用户要求“生成目标计划 / Goal Plan / Goal Pack / 使用 Goal Diffusion”，就进入 Goal Diffusion；随口小改动不创建 Goal Pack，直接实施并验证。

默认工作流是：

```text
human intent
-> agent writes goal charter
-> agent finds current edge
-> agent executes largest safe useful slice
-> agent records receipt
-> continue | plan_required | blocked | audit
-> final audit maps evidence to completion
-> done
```

Goal Pack 语义：

```text
charter.yaml             目标授权与人类意图压缩
state.yaml               当前运行态
receipts.jsonl           追加式证据检查点
implementation-plan.md   仅 plan_required 时存在
notes/                   长材料，不承载当前状态
```

当前仓库主路径使用 v1 Goal Pack schema。新增或重写方法论文档、CLI、checker、template、测试和 dogfood Goal Pack 时，使用 `charter.yaml`、`completion`、`engineering_guidance`、`checks`、`evidence_map` 口径，并同步更新相关文档和测试。

## 目录结构

- `packages/cli/src/`：CLI 命令和库代码。
- `packages/cli/test/`：CLI 行为测试。
- `scripts/`：发布脚本和脚本级测试。
- `skills/`：AI Coding Project OS suite 的 agent skill 镜像，由 `skill-manager` / `skillshare` 从 SSoT 分发。
- `docs/`：项目文档分层入口。先读 `docs/README.md`。
- `docs/goal-diffusion/goals/`：Goal Pack 示例或历史工作记录。
- `assets/`：README 横幅等媒体资源。

## Docs 治理

遵循 `docs-governance` 口径：

- `docs/README.md` 是文档路由入口，不承载领域真相。
- `docs/product/**` 放产品/方法论定位和用户价值。
- `docs/ssot/**` 放当前事实、术语和不变量。
- `docs/standards/**` 放可执行规则、命令、质量门和协作 SOP。
- `docs/adr/**` 放已采纳取舍。
- `docs/architecture/**` 放系统结构、模块关系和运行时视图。
- `docs/roadmap/**` 放迁移顺序、状态、证据链接和后续波次。
- `docs/goal-diffusion/**` 放 Goal Diffusion 方法产物；不要放项目级 docs 治理规则。

新增或迁移文档时：

1. 先判断文档层。
2. 更新最近的 README / 索引。
3. 不保留两个 current home。
4. 不删除仍被引用的证据。
5. 报告只承载证据摘要，不写过程日记。

## 开发命令

使用 Bun。

```bash
bun install
bun run build
bun run typecheck
bun run test
bun run check
```

- `bun run build` 构建 CLI 到 `dist/`。
- `bun run typecheck` 运行 TypeScript 检查。
- `bun run test` 运行 Bun 测试。
- `bun run check` 运行 build、typecheck、test；提交前使用。
- `bun run pack:dry` 检查 npm 包内容。

## 代码规范

- TypeScript 使用 ES modules。
- 保持现有风格：2 空格缩进、分号、双引号、显式 `node:*` imports。
- 源码文件用 kebab-case；函数和变量用 camelCase。
- CLI 行为必须确定性；优先结构化解析，避免 ad hoc 字符串处理。

## Agent 实施规则

- 不重写真实 Goal Pack 的历史 `receipts.jsonl`；需要修正解释时追加新 receipt。
- 默认继续推进：只要目标、边界、证据路径和风险仍在 `charter` 内，就继续执行。
- 不把普通目标膨胀成完整任务树；只找当前 edge，执行最大安全有用 slice。
- 只有 public API / schema / protocol、安全、权限、私有数据、破坏性迁移、release/compliance、多 agent 严格协作等场景才升级到 strict。
- `implementation-plan.md` 只用于 `plan_required` 高风险 slice；它不是第二套任务系统。
- 改公开命令、Goal Pack schema、receipt 语义、skill 口径时，同步更新 README、skills、templates、checker rules 和测试。

## Pull Request

提交应保持 scope 小。PR 需要包含：

- 变更摘要；
- 用户可见命令或 schema 变化；
- 文档 / skill / template 是否同步；
- 验证命令，通常是 `bun run check`。

---
> Source: [yoyooyooo/ai-coding-os](https://github.com/yoyooyooo/ai-coding-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
