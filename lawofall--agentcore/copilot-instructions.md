## frontend-preview

> 改桌面前端组件/样式/布局后想自检渲染效果、或想离线查看某个 AI 态（辩论/派单/升级/审批/计划复核/工具/引用…）时读——#/preview 离线回放 conformance 向量、帧滑块逐帧看流式中间态、pnpm dev:web 纯浏览器跑渲染层、pnpm shoot 无头截图自检、协作图三入口视口探针 shoot:graph-probe、CI 渲染冒烟门禁。别再靠跑真实 AI 看前端效果。改 AI 小镇 3D（apps/town Unity）自检见文末——用 Offline Demo + pnpm town:shoot:webgl，别问用户能否跑 Unity。


# 前端离线预览与截图自检

桌面渲染层几乎就是个普通 Vite + React 网页：把仓库里那批 **conformance 向量**（`pnpm conformance` 同一份）经**真实** `dispatchSSEEvent` 回放进**真实**组件，即可离线复现每种 AI 态——零后端、零 LLM、零 token、永不 drift。**改完前端别再去跑真实 AI 看效果。**

> 本页是 AI 自检渲染。**给人录产品演示**（真实运行 → 磁带 → 真实桌面壳按原节奏可控重放）是另一套 → [`demos/README.md`](/demos/README.md)。

## 三个入口

| 谁 | 怎么进 |
|---|---|
| 人·应用内 | 命令面板（`Ctrl/Cmd+K`）搜「前端预览」→ `#/preview`；深链单场景 `#/preview?s=<场景名>`；工具栏帧滑块拖动看流式中间态（深链 `?k=<n>`） |
| 人·纯浏览器 | `pnpm -C apps/desktop dev:web` → 开 `http://localhost:5199/index.web.html#/preview` |
| AI·截图自检 | `pnpm -C apps/desktop shoot [场景名过滤]` → 逐场景出图到 `shoot-out/<name>.png`，`SHOOT_FRAMES=N` 连中间帧，**读图核对**。同族独立预览：首启体验 `#/preview/onboarding` + `shoot:onboarding`（草稿空态两态 + composer 生成中插话态，平台代付无接入门；非 conformance 向量、纯 UI 场景）；产品手册 `shoot:manual`（四章代表场景，含机制真图与辩论室/拍板卡嵌入预览；改手册内容源 `pages/toolbox/manual/content/` 或手册渲染器后必跑） |
| AI·协作图视口探针 | `pnpm -C apps/desktop shoot:graph-probe -- <场景名> [输出前缀]` → 对聊天内联/画布/全屏三入口量 emptyTopRatio · fullyInside · 裁切数并出截图/JSON 到 `shoot-out-graph-probe/`；**改图宿主/布局/fit/视口后必跑**（辩论场景 `multi_agent_debate`、`multi_agent_debate_multibeat`） |

## 纪律

- 改桌面组件/样式/布局 → `pnpm shoot` → 读 `shoot-out/*.png` 自检，形成「改→截→看→迭代」紧回路。
- 改交互相关代码（发消息 / 审批卡 / 开工·checkpoint resume / 协作图入口 / AuthGate·会话切换等用户操作链路）→ `pnpm -C apps/desktop e2e`（mock 后端 + webapp 壳 + Playwright；失败留 `e2e-results/` 截图与 trace）。与「改渲染 → `pnpm shoot`」并列，别靠跑真实 AI / 活后端验点击行为。
- **`pnpm shoot` 是渲染冒烟门禁**（`ci.yml` frontend job **在跑**；近期风险是 CI 红如 Lint，早失败则 shoot 等后续 skipped）：任一场景渲染崩（未捕获报错 / `#/preview` 挂不上）即红。**发布前本地必跑** `pnpm shoot` / `release:gate`。
- 新 AI 态补向量：后端 `conformance/vectors/` 加场景 → `cd apps/server && uv run python -m agentcore.conformance.export` 重导 fixture → `#/preview` 与 `pnpm conformance` 同步刷新。

## AI 小镇 3D（`apps/town` Unity，不走 #/preview）

小镇观测层是**独立 Unity 客户端**，有自己的离线自检链（无后端 / 无 LLM），**别用 #/preview**：

- Offline Demo：Play 点左侧「离线 Demo」/ `AgentTown.exe --demo` / WebGL `?demo=1`（本地合成帧）
- 截图自检 `pnpm town:shoot:webgl` → `apps/town/shoot-out/*.png`（须先 `town:build:webgl`）；逻辑冒烟 `pnpm town:verify`
- 无 DeepSeek 连后端走 scripted（`SIMULATION_SCRIPTED=true` 或建 run `scripted:true`）

**开发机已装 Unity 6，别再问用户能否跑 Unity**——按上面自检。命令/前置详见 [`apps/town/README.md`](/apps/town/README.md)。

完整机制 / 浏览器打桩 / AuthGate 离线旁路 / 被否决方案（Storybook、手写 mock 态）→ [`前端技术与架构.md §12.3`](/docs/04-前端/前端技术与架构.md)。

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
