## ui-guidelines

> UI/UX 指南、样式规范和组件使用


# UI 与样式指南

## 🎨 样式系统
- **引擎**: UnoCSS (原子化 CSS) 是**首选**的样式方案。
- **配置**: `uno.config.ts`。
- **预处理**: SCSS 用于复杂的组件样式（极少需要）。
- **主题**: 通过 `src/theme.json` 和 CSS 变量支持亮色/暗色模式切换。

### UnoCSS 约定
- 使用工具类: `flex`, `items-center`, `text-primary`, `m-4`.
- 响应式前缀: `sm:`, `md:` (在移动端优先的 uni-app 中较少使用)。
- 图标: 通过 UnoCSS preset 使用 `i-carbon-{icon-name}`。

## 🧩 组件库
- **核心库**: `wot-design-uni` (`wd-` 前缀)。
- **文档**: [wot-design-uni](https://wot-ui.cn).
- **自定义组件**: 在 `src/components` 中创建。

## 📢 全局反馈
- **Toast/Message**: 请勿直接使用 `uni.showToast`。
- **标准**: 使用 `GlobalToast`, `GlobalMessage`, `GlobalLoading` 组件。
- **Skill**: 参考 **`global-feedback`** skill 查看使用示例。

## 📱 布局
- **系统**: `vite-plugin-uni-layouts`。
- **默认**: `src/layouts/default.vue`。
- **TabBar**: `src/layouts/tabbar.vue`。

---
> Source: [wot-ui/wot-starter](https://github.com/wot-ui/wot-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
