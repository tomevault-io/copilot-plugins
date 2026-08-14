## oh-my-dsh

> > 本文件指导 swarm 子代理（以及任何后续接手的 agent）如何在本仓库工作。

# AGENTS.md — oh-my-dsh swarm 操作手册

> 本文件指导 swarm 子代理（以及任何后续接手的 agent）如何在本仓库工作。
> 核心：**类永动机** — 调研差距 → ledger → 插件补齐 → 验证 → 继续。

## 1. 目标

持续探寻 DSH（DeepSeek Harness SDK）与其他开源 harness（opencode、oh-my-pi、Codex CLI、Claude Code、pi 等）的**功能差距**，并以 DSH 插件方式补齐，**不破坏骨架与性能**。

## 2. 关键事实（环境）

- **DSH 源码克隆**：`/Users/laplace/Desktop/dsh 1/harness-analysis/`（packages/ 工作区、docs/、.agents/notes/）
- **DSH 插件铁律**：
  - 一切皆插件（含 agent loop）；新行为走**文档化扩展接缝**（`ctx.tools`、`ctx.bash`、`ctx.fs`、`ctx.skills`、`ctx.subagents`、`ctx.workflows`、`ctx.goals`、事件瀑布等）
  - 注册是副作用：`ctx.effect()` / `ctx.on()`；`register()` 返回 disposer
  - 能力接缝三件套：interface / implementation / consumer 三个包
  - 显式 > 隐式：默认值是实现里显式 `resolve(request): Spec`，不是 `run()` 里 `?? default`
  - **禁止**：修改 `agent-loop` 骨架、热路径加开销、硬编码可调项
- **已调研材料**：`/Users/laplace/Desktop/dsh 1/harness-analysis/analysis/notes/comparison/`（codex.md、claude-code.md、pi.md、opencode.md、report-final.md——12 条 DSH 巧思）

## 3. Swarm 工作流（每轮）

1. **调研**：并行派发子代理，各调研一个对照项目（或一个差距维度），产出 `swarm/ledger/GAP-xxxx.md` 差距条目。
2. **记账**：更新 `GAP-LEDGER.md`（新差距追加，状态 `open`）。
3. **设计**：每个差距写设计方案（用什么接缝、三件套怎么分、性能影响评估）→ 状态 `designing`。
4. **实现**：在 `plugins/<gap-id>/` 写插件（npm 包 `@oh-my-dsh/<gap-id>`，ESM，遵循 DSH 包规范）。
5. **验证**：`pnpm test`（vitest）+ 性能基线对比 → 状态 `verifying`。
6. **关闭**：差距关闭（`closed`），回到步骤 1。

## 4. 差距条目格式（GAP-xxxx.md）

```markdown
# GAP-xxxx：<差距名称>
- 来源项目：<opencode / oh-my-pi / codex / claude-code / pi / 其他>
- 发现日期：YYYY-MM-DD
- 差距描述：<DSH 缺什么，对照项目有什么>
- 证据：<URL / 源码路径 / 行号>
- 影响评估：<功能面 / 体验面 / 性能面>
- 方案：<接缝选择、三件套拆分>
- 状态：open | designing | implementing | verifying | closed
```

## 5. 插件实现规范

- 包名：`@oh-my-dsh/<gap-id>`
- 结构：`plugins/<gap-id>/` 下 `package.json`（`"type": "module"`）、`src/index.ts`、`tests/`
- 必须：`name` / `inject` / `Config` / `apply` 导出（函数插件）或默认导出服务类（服务包）
- 必须：注册返回 disposer；事件监听用 `ctx.on()`；变更用 `ctx.effect()`
- 禁止：改 `agent-loop`、全局状态污染、同步热路径开销、硬编码配置
- 测试：vitest 单测 + 必要时快照测试；测试不得依赖真实 API key

## 6. 提交纪律

- 每个插件一个 commit：`feat(plugins/<gap-id>): <描述>`
- ledger 更新随同提交：`docs(swarm): GAP-xxxx -> closed`
- 提交前 `git diff --cached --check` 无空白错误

## 7. 性能纪律

- 插件注册本身必须轻量（无同步 I/O、无阻塞）
- 热路径（每请求/每工具调用）不得新增同步分配或日志
- 参照 `docs/performance.md` 基线；性能回退视为 bug

---
> Source: [LaplaceYoung/oh-my-dsh](https://github.com/LaplaceYoung/oh-my-dsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
