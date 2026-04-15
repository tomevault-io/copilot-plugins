## oclaw

> Oclaw 项目约定与扩展方式


# Oclaw

本仓库为 **Oclaw**（与 OpenClaw 结合使用的桌面浏览器），技术栈：Tauri 2 + Vue 3 + TypeScript + Vite + Pinia + UnoCSS + Element Plus。

- **路由**：`src/pages/` 下 `.vue` 即路由（unplugin-vue-router）。
- **布局**：`src/layouts/default.vue` 为默认布局。
- **状态**：Pinia stores 在 `src/stores/`，可自动导入。
- **Tauri**：命令按领域写在 `src-tauri/src/` 的模块中（如 webview、sidecar、app），在 `lib.rs` 注册；前端通过 **`src/api/`** 封装后调用，不在页面/组件里直接 invoke。

**架构与规范**（前端目录、API 层、composables、Rust 模块化）见 **`.agents/skills/oclaw/SKILL.md`**；总览见 `.agents/AGENTS.md`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bichenxi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bichenxi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
