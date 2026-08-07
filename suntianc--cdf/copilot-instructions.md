## cdf

> 本文件是给 Codex、Claude、Cursor 等代码代理使用的项目工作指南。请优先遵守本文件；更细的项目说明和设计约定分别参考 `README.md`、`DESIGN.md`。`Claude.md` 仅作为指向本文件的兼容入口，不承载额外规则。

# AGENTS.md

本文件是给 Codex、Claude、Cursor 等代码代理使用的项目工作指南。请优先遵守本文件；更细的项目说明和设计约定分别参考 `README.md`、`DESIGN.md`。`Claude.md` 仅作为指向本文件的兼容入口，不承载额外规则。

## 沟通与工作原则

- 必须使用中文与用户沟通。
- 先读代码和现有文档，再修改。不要凭记忆假设项目结构、命令或 API。
- 只做用户请求范围内的修改。不要顺手重构、改格式、清理无关代码或调整无关文档。
- 当前产品尚无外部用户。涉及数据模型纠正或功能重做时，除非用户明确要求，否则不为旧实现设计兼容层、历史数据迁移或遗留行为保留；优先采用简单、彻底且长期正确的模型，开发期数据可直接重置。
- 优先采用现有模式、组件、store、IPC 结构和测试风格；不要为了单次需求引入新抽象。
- 修改前明确目标和验证方式；修改后运行与变更范围匹配的测试或构建。
- 工作流应与变更风险匹配：简单局部修复遵循“定位、可重复验证、最小改动、定向验证”的最小闭环；不要为了流程增加无关分解、重复审查或大范围验证。
- 工作区可能已有用户改动。不要回滚、覆盖或整理非本次任务产生的改动。
- `.codex/` 等本地代理状态目录不再纳入版本控制；不要主动重新添加或提交这些目录。

## 项目定位

CDF 是一个离线优先的 Electron 桌面端 Agent 工作站，不是普通聊天页。用户在本地组织任务、上下文、Agent、能力、工作流、过程和产物。

核心技术栈：

- Electron + electron-vite
- React 19 + TypeScript + Vite
- Tailwind CSS v4 + Radix UI + Lucide React
- Zustand
- React Flow / `@xyflow/react`
- LangChain、LangGraph、deepagents、MCP adapter
- `better-sqlite3`、`electron-store`
- Vitest 4 + Testing Library

## 目录约定

- `src/main/`：Electron 主进程、IPC、数据库、LLM、deepagent、工作流运行时。
- `src/preload/`：`contextBridge` 预加载脚本。渲染进程能力必须从这里安全暴露。
- `src/renderer/src/`：React 渲染进程、组件、hooks、stores、样式和 i18n。
- `src/shared/`：主进程与渲染进程共享类型和常量。
- `resources/`：应用资源。
- `patches/`：`patch-package` 补丁。
- `scripts/`：项目脚本。

## 常用命令

使用 `pnpm`，不要切换到 npm/yarn。项目要求 Node.js `>=22`，包管理器为 `pnpm@11.5.1`。

```bash
pnpm install
pnpm run dev:electron
pnpm run dev
pnpm test
pnpm run test:watch
pnpm run build
pnpm run preview
```

命令说明：

- `pnpm run dev:electron`：推荐的开发启动命令，会先把 `better-sqlite3` rebuild 到 Electron ABI。
- `pnpm run dev`：更快，但要求原生模块已经是 Electron ABI。
- `pnpm test`：全量 Vitest，`pretest` 会把 `better-sqlite3` rebuild 到 Node ABI。
- 跑过 `pnpm test` 后，如需启动应用，优先使用 `pnpm run dev:electron`，避免 ABI 不匹配。
- 单测可按文件或名称过滤，例如 `pnpm test src/main/deepagent/agent-tools.test.ts` 或 `pnpm test -t "case name"`。

## 代码风格

- TypeScript 使用严格模式；保持类型边界清晰，避免无意义的 `any`。
- 遵循当前文件风格。仓库中部分文件使用分号，部分文件不使用；编辑时以局部文件风格为准。
- 优先使用命名清晰的小函数；只有在能减少真实复杂度时才新增抽象。
- 主进程可使用 Node API；渲染进程不要直接使用 Node/Electron 能力。
- 共享跨进程类型放在 `src/shared/`，不要在 renderer 和 main 之间复制类型定义。
- 日志沿用现有 `src/main/logger.ts` 模式；不要用大量临时 `console.log` 留在主进程代码中。
- i18n 文案需同步维护 `src/renderer/src/i18n/locales/en-US.json` 与 `zh-CN.json`，不要只改一种语言。

## Electron 与安全边界

- 保持 `contextIsolation: true`，保持 `nodeIntegration: false`。
- 渲染进程需要主进程能力时，通过 `src/preload/index.ts` 和 IPC 暴露最小 API。
- 不要引入 `remote` 模块或同步 IPC。
- 文件系统、shell、MCP、网络和模型调用等高权限能力应留在 `src/main/`。
- 离线优先是产品约束。新增功能默认本地存储、可本地运行；外部网络依赖必须是显式、可配置、可失败降级的。

## 前端与设计约定

- 遵守 `DESIGN.md`：CDF 是工作站界面，不是 SaaS dashboard、聊天气泡页或 hero landing。
- 使用现有设计 token：`--bg-*`、`--text-*`、`--accent`、`--border`、`--block-*` 等。
- Light 主题是奶白画布 + 粉彩 color block + single-shot magenta accent；Dark 主题是冷黑画布 + violet accent。
- 不要随意新增颜色、阴影体系或装饰性渐变。色彩用于焦点、状态、协作和风险信号。
- UI 组件优先复用 `src/renderer/src/components/ui/`、Radix、Lucide React 和现有组件模式。
- 图标按钮优先使用 `lucide-react`；复杂交互要有可访问的 label/title。
- 保持桌面生产力工具的信息密度和稳定布局，避免营销页式大卡片堆叠。

## 状态、数据与工作流

- 渲染进程全局状态优先使用已有 Zustand store。
- SQLite 和长期本地数据逻辑放在主进程，避免 renderer 直接持久化关键业务数据。
- 工作流相关代码集中在 `src/main/workflow-run/` 和 `src/renderer/src/components/WorkflowEditor/`（旧 `src/main/workflow/` 引擎已随 ADR-0054 C-lite 重构删除）。
- deepagent 相关代码集中在 `src/main/deepagent/`；修改 provider、tool、runtime 行为时必须检查相邻测试。

## 测试策略

- 测试文件与源码同目录，使用 `.test.ts` 或 `.test.tsx`。
- `src/main/**/*.test.ts` 在 node 环境运行，可 mock Electron、访问 Node API 和 `better-sqlite3`。
- `src/renderer/**/*.test.{ts,tsx}` 在 jsdom 环境运行，使用 Testing Library。
- 组件测试优先验证用户可见行为，不要过度绑定 DOM 内部结构。
- 修 bug 时优先补一个能失败的回归测试，再修到通过。
- 修改 IPC、数据库、deepagent、工作流运行时或共享类型时，至少跑相关单测；影响面不清时跑 `pnpm test`。
- 交付前如条件允许跑 `pnpm run build`，尤其是改到 Electron/Vite 配置、preload、共享类型或跨进程调用时。

## 原生模块 ABI 注意事项

本项目依赖 `better-sqlite3`。Node.js 测试环境和 Electron 运行环境需要不同 ABI：

- `pnpm test` 前的 `pretest` 会 rebuild 到 Node ABI。
- `pnpm run dev:electron` 和 `postinstall` 会 rebuild 到 Electron ABI。
- 如果测试后开发启动失败，先运行 `pnpm run dev:electron`，不要手动替换依赖或删除 lockfile。

## 依赖与配置

- 不要手动编辑 `pnpm-lock.yaml`，除非确实变更依赖并通过 `pnpm install` 更新。
- 不要移除 `patches/` 中的补丁，除非已经验证上游依赖不再需要。
- 不要提交本地代理状态目录，例如 `.codex/`、`.claude/`、`.agents/`、`.pi/`、`.impeccable/`。
- 修改 `electron.vite.config.ts`、`vitest.config.ts` 或 TypeScript 配置时，要同步考虑 main、preload、renderer 三个构建目标。
- Renderer alias：
  - `@` 指向 `src/renderer/src`
  - `@shared` 指向 `src/shared`

## 版本与发布

项目存在两套版本号，不要混用：

- 产品发布版本：以 git tag 为准，例如 `v0.1.x`，只在 `master` 分支打 annotated tag。

除非用户明确要求，不要 bump `package.json` 的 `version`，不要创建 commit、branch、tag 或 release。

## 提交前检查清单

- 变更是否严格对应用户请求？
- 是否保留了用户已有未提交改动？
- 是否遵守 Electron 安全边界和离线优先约束？
- 是否同步了中英文 i18n 文案？
- 是否为新增或修复行为补了必要测试？
- 是否运行了最小必要验证命令，并在最终回复中说明结果？

## Agent skills

### Issue tracker

Issues 追踪在 GitHub Issues（仓库 suntianc/CDF），外部 PR 不纳入分诊。详见 `docs/agents/issue-tracker.md`。

### Triage labels

使用默认标签词汇（needs-triage / needs-info / ready-for-agent / ready-for-human / wontfix）。详见 `docs/agents/triage-labels.md`。

### Domain docs

单一上下文布局——根目录一个 `CONTEXT.md` + `docs/adr/`。详见 `docs/agents/domain.md`。

---
> Source: [suntianc/CDF](https://github.com/suntianc/CDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
