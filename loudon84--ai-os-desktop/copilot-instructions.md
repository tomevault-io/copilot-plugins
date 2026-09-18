## workbuddy-product-line

> Work 专家工作台（v1.3）产品线约束 — 改 Hermes Screen 前必读 Spec Pack


# Work 专家工作台（v1.3 Product Line）

`screens/Hermes` 是 **Work 专家工作台**（顶栏 Tab：`local-hermes` / Work Expert Workspace），不是通用 Hermes 配置台。

## Spec Pack（单一事实源）

改 Layout 壳层或 Screen 页面前 **必须先读**：

```text
docs/specs/v1.3-workbuddy-product-line/00-overview.md
docs/specs/v1.3-workbuddy-product-line/03-layout-boundary.md   # Layout / 三栏 / hermes-page 模板
docs/specs/v1.3-workbuddy-product-line/13-ai-coding-structure.md  # UI 输出质量 Checklist
```

按任务打开 06–10 页面 Spec。任务前缀模板：`16-cursor-execution-prompt.md`

## 主流程

```text
Workbench → Experts / ExpertTeams → Summon → ExpertRuns → Artifacts
```

## 硬约束

- 代码标识：`Work*`、`workApi`（**禁止** `workbuddy` 作为代码名）
- 分层：`pages → features → workApi`；`components` 不调 API
- UI：复用 `Hermes.css` 的 `hermes-*` class（见 03-layout-boundary §5）
- i18n：`workspaces.*` en + zh-CN 同步
- **禁止**改 `Layout.tsx` / `MainPage`（Hermes 内页任务）
- **禁止** Renderer 直接 `window.hermesExperts` / `ipcRenderer` / fetch nodeskclaw

## 导航

- primary 6 项；capability / advanced 默认折叠
- `requiresGateway` 离线 → disabled + Shell redirect workbench（不隐藏入口）

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
