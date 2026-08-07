## islet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概览

Islet 是一个极简像素风（Neo-brutalism / 8-bit）的浏览器新标签页扩展（Manifest V3，兼容 Chrome 与 Firefox）。技术栈：Vue 3（`<script setup>` + Composition API）+ TypeScript + Vite + Sass + Pinia + vue-i18n。包管理器为 pnpm。

**硬性设计原则**（影响功能取舍）：

- **零联网权限**：manifest 不申请任何 host_permissions / permissions。任何需要跨域接口的功能（如搜索引擎实时联想）都会破坏这一卖点，应改用纯本地方案（参见 TODO.md 中任务 1 的决策记录）。
- 极简、像素美学、扩展体积尽量小（不轻易引入依赖或内置资源）。

## 常用命令

```bash
pnpm dev                    # 开发服务器（普通网页方式调试）
pnpm build                  # 类型检查 (vue-tsc -b) + 构建到 dist/
pnpm format                 # Prettier 格式化 src/
scripts/publish.sh build    # 按当前版本构建 Chrome/Firefox 双 zip 到 release/（本地调试用）
scripts/publish.sh patch    # 发布：升级版本 -> 构建双 zip -> commit + tag -> 询问推送
```

没有测试与 ESLint；`pnpm build` 中的 vue-tsc 是唯一的静态检查，改完代码用它验证。

## 工作流约定（来自 TODO.md）

- 迭代流程：**逐个任务实现 → 交给用户手动验收 → 通过后再 commit**。不要未经验收就 commit。
- Commit message 为中文 conventional 风格（`feat: ...` / `fix: ...` / `chore: ...`），**不含任何 AI 署名信息**（不要加 Co-Authored-By 等）。
- TODO.md 记录迭代规划与已搁置的方向，完成任务后更新其中的状态标记。

## 架构要点

单页 Vue 应用即新标签页本身，无 background/content scripts。[App.vue](src/App.vue) 平铺组合所有组件：时钟、搜索栏、一言、设置按钮/面板。

**跨文件的关键机制**（改动时容易遗漏的联动）：

- **新增一个设置项需要同步四处**：
  1. [src/store/setting.ts](src/store/setting.ts) — 唯一的 Pinia store，整体持久化到 localStorage（key `islet-setting`）；
  2. [src/components/SettingPanel.vue](src/components/SettingPanel.vue) — 设置 UI（Tabs：时钟 / 系统 / 关于）；
  3. [src/utils/settingsTransfer.ts](src/utils/settingsTransfer.ts) — 导入/导出的 `TRANSFER_KEYS` 与 `VALIDATORS`（逐项类型校验，漏加则该设置不参与备份）；
  4. [src/locales/](src/locales/) 下 **zh / en / ja 三个 JSON 都要加文案**。
- **搜索引擎单一数据源**：[src/config/searchEngines.ts](src/config/searchEngines.ts)（`BUILTIN_ENGINES` + `buildSearchUrl`，`%s` 占位）。`MainSearchBar` 与 `SettingPanel` 都从这里读取；用户自定义引擎存于 store 的 `customEngines`。不要在组件里硬编码引擎。
- **主题防闪屏（FOUC）**：[public/theme-init.js](public/theme-init.js) 是 `index.html` head 中的同步外部脚本，首屏前套用主题。**必须是外部脚本**——MV3 扩展 CSP（`script-src 'self'`）禁止内联脚本。运行时切换由 [src/composables/useTheme.ts](src/composables/useTheme.ts) 负责（切换 `html.dark` 类），两者的颜色值需与 [src/style.css](src/style.css) 的 CSS 变量保持一致。
- **双浏览器 manifest**：[public/manifest.json](public/manifest.json) 以 **Firefox 完整形态**为单一数据源；[scripts/make-browser-dists.mjs](scripts/make-browser-dists.mjs) 为 Chrome 裁剪掉 `chrome_settings_overrides`（Chrome 仅 Windows/macOS 支持，Linux 上会拒绝加载整个扩展）与 `browser_specific_settings`（Firefox 专属）。该脚本被 `scripts/publish.sh` 与 CI 共用，改 manifest 相关逻辑只改这一处。

## 发布

推送 `vX.Y.Z` 形式的 tag 即触发 [.github/workflows/release.yml](.github/workflows/release.yml)：写回版本号到 `manifest.json`/`package.json` 并提交回 main → 构建 → 生成 Chrome/Firefox 双 zip → 创建 GitHub Release。本地推荐用 `scripts/publish.sh <patch|minor|major|X.Y.Z>` 完成整个流程。

## 风格

- 代码注释、commit、文档均为中文。
- UI 组件复用 [PixelSelect.vue](src/components/PixelSelect.vue) / [PixelToggle.vue](src/components/PixelToggle.vue)，配色一律引用 style.css 中的 CSS 变量（`--bg-main`、`--text-main` 等），像素字体为 "Fusion Pixel"。

---
> Source: [LumiDesk/islet](https://github.com/LumiDesk/islet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
