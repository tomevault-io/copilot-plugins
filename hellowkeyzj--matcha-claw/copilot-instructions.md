## matcha-claw

> MatchaClaw is a cross-platform **Electron desktop app** (React 19 + Vite + TypeScript) providing a GUI for the OpenClaw AI agent runtime. It uses pnpm as its package manager (pinned version in `package.json`'s `packageManager` field).

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

MatchaClaw is a cross-platform **Electron desktop app** (React 19 + Vite + TypeScript) providing a GUI for the OpenClaw AI agent runtime. It uses pnpm as its package manager (pinned version in `package.json`'s `packageManager` field).

### Linked References

- `@docs/gateway-rpc-api.md` — OpenClaw Gateway RPC 开发基线（握手、帧结构、方法清单、实现层差异与调用边界）。
- `@docs/gateway-events-api.md` — OpenClaw Gateway 事件开发基线（事件总表、`agent/chat` 语义、慢连接策略、`node.event` 上报事件）。
- `@docs/hook-extension-points.md` — OpenClaw 插件 Hook 完整插入点（触发位置、优先级/执行顺序、返回值与异常语义）。

> 约束：凡涉及 OpenClaw 接口、事件流或插件扩展的开发，必须先对齐上述文档，再进入实现。

### Quick reference

Standard dev commands are in `package.json` scripts and `README.md`. Key ones:

| Task | Command |
|------|---------|
| Install deps + download uv | `pnpm run init` |
| Dev server (Vite + Electron) | `pnpm dev` |
| Lint (ESLint, auto-fix) | `pnpm run lint` |
| Type check | `pnpm run typecheck` |
| Unit tests (Vitest) | `pnpm test` |
| E2E tests (Playwright) | `pnpm run test:e2e` |
| Build frontend only | `pnpm run build:vite` |

### Non-obvious caveats

- **pnpm version**: The exact pnpm version is pinned via `packageManager` in `package.json`. Use `corepack enable && corepack prepare` to activate the correct version before installing.
- **Electron on headless Linux**: The dbus errors (`Failed to connect to the bus`) are expected and harmless in a headless/cloud environment. The app still runs fine with `$DISPLAY` set (e.g., `:1` via Xvfb/VNC).
- **`pnpm run lint` race condition**: If `pnpm run uv:download` was recently run, ESLint may fail with `ENOENT: no such file or directory, scandir '/workspace/temp_uv_extract'` because the temp directory was created and removed during download. Simply re-run lint after the download script finishes.
- **Build scripts warning**: `pnpm install` may warn about ignored build scripts for `@discordjs/opus` and `koffi`. These are optional messaging-channel dependencies and the warnings are safe to ignore.
- **`pnpm run init`**: This is a convenience script that runs `pnpm install` followed by `pnpm run uv:download`. Either run `pnpm run init` or run the two steps separately.
- **Gateway startup**: When running `pnpm dev`, the OpenClaw Gateway process starts automatically on port 18789. It takes ~10-30 seconds to become ready. Gateway readiness is not required for UI development—the app functions without it (shows "connecting" state).
- **No database**: The app uses `electron-store` (JSON files) and OS keychain. No database setup is needed.
- **AI Provider keys**: Actual AI chat requires at least one provider API key configured via Settings > AI Providers. The app is fully navigable and testable without keys.
- **Token usage history implementation**: Dashboard token usage history is not parsed from console logs. It reads OpenClaw session transcript `.jsonl` files under the local OpenClaw config directory, extracts assistant messages with `message.usage`, and aggregates fields such as input/output/cache/total tokens and cost from those structured records.
- **Renderer/Main API boundary (important)**:
  - Renderer must use `src/lib/host-api.ts` and `src/lib/api-client.ts` as the single entry for backend calls.
  - Do not add new direct `window.electron.ipcRenderer.invoke(...)` calls in pages/components; expose them through host-api/api-client instead.
  - Do not call Gateway HTTP/RPC endpoints directly from renderer (`fetch('http://127.0.0.1:18789/...')`, `gateway:rpc`, etc.). Expose product behavior as runtime-host domain APIs and call them through `hostapi:fetch`.
  - Transport policy is Main-owned and fixed as `runtime-host -> IPC fallback`; renderer should not implement protocol switching UI/business logic.
- **Doc sync rule**: After any functional or architecture change, review `README.md`, `README.zh-CN.md`, and `README.ja-JP.md` for required updates; if behavior/flows/interfaces changed, update docs in the same PR/commit.

## 全局开发策略

- **重构兼容性策略**：当前项目尚未发布正式版本，任何重构默认不要求保持历史版本兼容性；优先保证结构清晰与长期可维护性。
- **默认回复风格**：默认简洁回复，先给结论，再补必要原因；非用户明确要求，不展开长篇背景说明。

## 改问题 / 重构执行纪律

1. 先走读完整链路，再给方案
- 先看清楚“用户操作 -> 状态变化 -> 页面渲染 -> 最终行为”整条链路。
- 不允许只看局部代码就直接下结论。

2. 先判断是不是模型问题
- 如果一个问题需要靠很多 if、effect、raf、时序补丁才能解释，默认不是小 bug，而是数据模型或状态模型有问题。
- 这种情况优先重构模型，不优先补丁。

3. 单一事实源
- 同一件事只能有一个地方说了算。
- 例如：“最后一条消息是谁”“现在该不该自动滚到底部”，都不能由多个变量、多个 effect、多个组件一起决定。

4. 区分三种东西
- 用户主动行为
- 程序自动行为
- 布局/渲染带来的被动变化
- 这三种信号不能混用。混用以后，程序很容易把自己的行为误判成用户意图。

5. 先写失败测试，再改实现
- 先用真实用户路径复现问题，再开始改代码。
- 测试要优先覆盖真实入口，不要只测内部理想路径。

6. 命令要有完成确认
- 不能假设“触发过一次就算成功”。
- 像滚动、同步、恢复定位这种行为，必须有“完成前持续 pending，完成后再清掉”的设计。

7. 改完后必须清旧代码
- 新模型落地后，旧的状态、旧 helper、旧注释、旧测试假设要一起删。
- 不允许新旧两套逻辑并存。

## Spec 开发流程约束

### 什么时候要进入 Spec 判断

- 先判断用户需求复杂度。
- 如果需求可以在一次对话内讲清楚、实现范围小、改动集中、验证简单，就按普通开发流程处理，不强制进入 Spec。
- 如果需求明显不是一次对话或短期内能稳定做完的工作，例如跨模块改造、涉及多阶段交付、需要多人交接、依赖关系复杂、边界还需要先澄清，就要先问用户是否启动 Spec 开发流程。

### 用户确认后怎么做

- 用户确认进入 Spec 开发流程后，必须按 `spec/000-Spec规范/` 里的规范和模板创建或更新对应 Spec。
- 入口文档是 `Spec/000-Spec规范/Codex-Spec规范文档.md`。
- 模板目录是 `Spec/000-Spec规范/Spec模板/`。
- 新 Spec 至少要包含 `requirements.md`、`design.md`、`tasks.md`；需要补充资料时再增加 `docs/`。
- `AGENTS.md` 只定义入口规则和执行纪律。Spec 的字段、模板结构、状态规则、自检清单，以 `spec/000-Spec规范/Codex-Spec规范文档.md` 为准。

### Spec 文档必须怎么写

- 后续只要在 `Spec/` 目录里新建或修改 Spec，正文都必须写成人话，先让刚接手项目的人看懂“这是要做什么”，再考虑追踪信息。
- 不要把黑话当正文。像“闭环”“抽象层”“赋能”“治理”“编排”“底座”“能力沉淀”这类词，能不用就不用；必须用时，后面立刻补一句人话解释。
- 任务文档里的每个任务，都必须先回答这些问题：
  - 这一步到底做什么
  - 做完以后能看到什么结果
  - 这一步依赖什么
  - 主要改哪些文件
  - 这一步明确不做什么
  - 怎么验证是真的做完了
- 可以保留“对应需求”“对应设计”这类追踪信息，但追踪信息只能辅助理解，不能代替正文。
- 模板也必须保持人话风格。如果发现模板重新变成黑话，先修 `spec/000-Spec规范/Spec模板/`，不要把坏模板继续复制到新 Spec 里。

### Spec 执行过程中必须回写任务状态

- 在 Spec 开发流程中，`tasks.md` 是实时进度记录，不是事后补写的总结。
- 每完成一个 task，都必须在同一轮工作里及时回写更新对应 `tasks.md` 任务，至少同步更新任务勾选状态和 `状态` 字段。
- 如果任务进入 `BLOCKED`、`IN_REVIEW`、`CANCELLED`，也必须立即回写原因、阻塞条件或替代方案。
- 如果实际实现范围和原任务描述已经不一致，先更新 `tasks.md`，再继续往后做。
- 除非用户明确要求暂时不要更新，否则不允许攒几个已完成任务后一次性补写状态。

## git提交规范（全局）

以下规则用于**所有“要求我提交代码”**的场景，默认执行：

- **提交粒度**：一次仅提交一个完整能力或一个完整修复，保持原子性，不混入无关改动。
- **实现流程**：先明确边界与改动清单，再落地实现，再验证通过后再提交。
- **架构约束**：必须遵循现有框架边界（如 `host-api/api-client`、既有 `store/i18n/路由` 模式），禁止引入平行实现或重复轮子。
- **冗余治理**：与本次能力无关的代码不带入；旧残留、冲突逻辑和临时兜底应在同次提交内清理干净。
- **测试要求（TDD）**：
  - 先补失败用例，再实现修复，再跑通过
  - 提交前至少覆盖本次改动的关键行为回归
  - 用户要求“都测吧”时，执行相关测试集而非仅 smoke
- **提交信息要求**：
  - 提交标题和摘要使用中文
  - 必须明确写出“新增了什么功能、修改了什么行为、删了什么冗余”
  - 禁止信息不足的标题（例如仅写“迁移/调整/优化”而无具体内容）
  - 允许使用规范前缀（如 `feat/fix/refactor(scope): ...`）

---
> Source: [hellowKeyzj/Matcha-claw](https://github.com/hellowKeyzj/Matcha-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
