## resume-dodolalorc-cn

> 这是一个 Vue 3 + TypeScript + Vite 的纯前端简历编辑器。用户数据只保存在浏览器内；项目不含后端、认证或服务端存储。

# AI 协作指南

这是一个 Vue 3 + TypeScript + Vite 的纯前端简历编辑器。用户数据只保存在浏览器内；项目不含后端、认证或服务端存储。

## 开始工作

1. 阅读 `.agents/README.md`，再按任务类型读取对应的 workflow。
2. 先定位“数据模型 → 状态 → 编辑器 → 预览卡片 → 导出”链路，避免只改显示层。
3. 改动前运行 `git status --short`，不得覆盖已有用户改动。
4. 完成后至少运行 `bun run type-check` 和 `bun test:unit --run`；UI、导入或导出改动还应执行 `bun run build`。

## 架构边界（现状与目标）

- `src/types/resume.ts` 是简历 JSON 的契约。字段变更必须同时考虑默认数据、编辑器、卡片、导入导出和兼容策略。
- `src/data/cv.json` 是演示数据，不应承载用户持久化逻辑。
- `src/themes/` 只负责主题元数据、配色和布局变量；不要在主题定义中写业务状态。
- `src/views/components/` 是简历预览组件；`src/views/modules/resume-editor/` 是编辑组件。
- `src/utils/resume-export.ts` 依赖实际 DOM/CSS，调整简历 DOM 或 CSS 时要回归 HTML/PDF 导出。
- `src/views/resume-view.vue` 当前承担状态、持久化、导入导出、设置和渲染编排，已过大。新增逻辑优先抽到 composable 或模块，不要继续堆入此文件。

## 必须遵守的约束

- 不要修改或格式化无关文件；不要提交 `dist/`、`coverage/`、本地简历数据或浏览器存储导出。
- 保持对历史 `LocalizedText`（字符串或 `{ zh, en }`）的读取兼容，除非任务明确包含数据迁移。
- 导入外部 JSON 前先做运行时校验；`JSON.parse` 成功不代表简历数据合法。
- 新增主题必须提供完整 `ResumeThemeConfig`，并运行 `bun run theme:new -- "Theme Name"` 作为脚手架入口。
- 不得为了通过检查而使用 `any`、吞掉异常或删除测试。修复测试时应先明确产品行为。

## 验收清单

- 类型检查和单测通过。
- 数据模型改动覆盖默认数据、导入、编辑、预览、JSON/HTML/PDF 导出。
- 主题改动覆盖默认主题、切换、持久化和打印样式。
- 新增或修复的逻辑有针对性测试；仅快照不足以验证导入与状态规则。

## 任务路由

| 任务 | 必读文件 |
| --- | --- |
| 功能开发 | `.agents/workflows/feature.md` |
| 缺陷修复 | `.agents/workflows/bugfix.md` |
| 主题/排版 | `.agents/workflows/theme.md` |
| 数据模型与导入导出 | `.agents/skills/resume-data.md` |
| 重构 | `.agents/plans/refactor-roadmap.md` |

---
> Source: [dodolalorc/resume.dodolalorc.cn](https://github.com/dodolalorc/resume.dodolalorc.cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
