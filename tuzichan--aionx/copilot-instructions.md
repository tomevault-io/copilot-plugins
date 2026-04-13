## aionx

> - 原项目主参考路径固定为 `F:\Work\AionUi`。

# Project Instructions

- 原项目主参考路径固定为 `F:\Work\AionUi`。
- 后续所有“原项目对照”“高保真复刻”“页面尺寸/样式/布局/组件位置核查”都必须优先以 `F:\Work\AionUi` 为准。
- 当前仓库 `C:\Users\Victory\.codex\worktrees\f01b\AionX` 中的旧文件和 git 历史只能作为补充参考源，不能替代 `F:\Work\AionUi` 的优先级。
- 视觉复刻必须在现有 `layout-route + SettingsLayout + registry + typed API + page/components/hooks` 架构不变的前提下完成，不允许为了复刻效果直接照抄旧实现或回退架构。
- 设置页迁移与高保真复刻的持续执行计划以仓库根目录的 `plans.md` 为准，执行过程中需要持续维护该文件和 `docs/设置页迁移实施清单.md`。
- 任何涉及前端基础组件、表单控件、弹层、提示、导航原语或样式系统的工作，必须优先使用 `shadcn` 官方能力：先查 `shadcn` skill，再优先使用 `shadcn` CLI 和 Shadcn MCP；只有在官方组件、CLI 生成结果或 MCP 能力不能满足当前需求时，才允许做项目内定制实现，并且定制 API 需要尽量保持与 `shadcn` 官方 API 一致。
- 当前项目的 UI 组件迁移目标已明确升级为“完整切换到 `shadcn/ui` 体系”。这意味着凡是基础 UI 组件，只要 `shadcn/ui` 存在对应能力，就不能继续保留项目内旧 API、旧封装或“长得像 shadcn 但接口不同”的兼容层；必须先落到 `shadcn/ui` 默认组件与默认 API，再在其上做项目定制。
- 即使仓库中已经存在功能相近的旧组件、Arco 包装层或自定义组件，也不得以“已有类似实现”为理由继续沿用旧接口。迁移时应优先把调用方改到 `shadcn/ui` 官方组合方式，例如 `Button`、`Dialog`、`Sheet`、`Tabs`、`Select`、`Tooltip`、`DropdownMenu`、`Input`、`Textarea`、`Table`、`Sidebar` 等，再在样式、variant、语义包装层做最小必要扩展。
- 项目内允许的定制顺序固定为：先确认 `shadcn` 官方是否已有组件或推荐组合；再使用 `shadcn` 默认 API 接入；最后才允许在 `src/shared/ui/*` 或更高层语义组件中补项目定制。禁止先造自定义 API，再把视觉样式贴成 shadcn 风格。
- 若现有业务代码与 `shadcn/ui` 默认 API 不一致，迁移优先级应是“改业务调用方”，而不是“改造 shadcn 组件去兼容旧调用方”。除非有明确且不可回避的业务约束，否则不要保留旧 props 名、旧事件签名、旧 slot 结构或旧 DOM 约定。

## Project Skill Priority

- 本仓库存在项目级 skills：`E:\Work\AionX\.agents\skills\shadcn`、`E:\Work\AionX\.agents\skills\frontend-design`、`E:\Work\AionX\.agents\skills\vercel-react-best-practices`、`E:\Work\AionX\.agents\skills\find-skills`。处理本项目任务时，优先使用这些项目级 skills，再考虑全局同名 skill。
- 涉及 `components.json`、`src/shared/ui/*`、基础组件、表单控件、弹层、下拉、导航原语、主题 token、Tailwind 类、shadcn registry/CLI 的任务，必须优先使用 `shadcn` skill，并先走“查现有组件/查 docs/优先组合官方能力”的路径。
- 涉及页面视觉重构、布局复刻、交互观感、响应式界面、设计系统落地的任务，在遵守 `shadcn` 优先前提下，继续优先使用 `frontend-design` skill，确保输出保持高质量界面设计，而不是回退到通用样式或低保真拼装。
- 涉及 React 组件编写、React 页面重构、状态流、渲染性能、包体积、异步加载、交互卡顿治理的任务，优先使用 `vercel-react-best-practices` skill，特别是在修改 `src/features/*`、`src/pages/*`、`src/widgets/*`、`src/app/*` 中的 React 代码时。
- 当任务超出当前项目 skills 覆盖范围，或需要寻找新的测试、文档、自动化、专项工作流能力时，优先使用 `find-skills` skill 查找是否已有可安装能力，再决定是否直接手写实现。
- 默认组合顺序如下：UI 原语/组件任务用 `shadcn`；页面与视觉任务用 `shadcn + frontend-design`；React 重构与性能任务用 `shadcn + vercel-react-best-practices`；发现能力缺口时再补用 `find-skills`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TuziChan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TuziChan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
