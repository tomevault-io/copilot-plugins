## project-conventions

> 中国古文物数字博物馆项目核心约定（Vue3 + Pinia + Element Plus + Three.js）


# 项目核心约定

- 技术栈：Vue 3 + Vite + TypeScript + Pinia + Element Plus + Three.js；包管理用 `pnpm`
- 路径别名：始终用 `@/` 指向 `src`
- Vue：优先 `<script setup lang="ts">`；组件目录用 `index.vue`（可配 `index.scss` / `config.ts`）
- UI 文案：中文（枚举标签、提示、loading 文案等）
- 全局：Element Plus `size: 'small'`；图标走 `globalComponent`；loading 指令 `v-zLoading`
- 展厅 Three 逻辑放在 `src/utils/museumScene.ts`、`src/composables/useMuseumHall.ts`，不要塞进 SFC 模板逻辑
- 内容数据优先改 `src/data/museumArtifacts.ts` / `museumHotspots.ts`
- 提交信息：Conventional Commits（`feat` / `fix` / …），遵循 `.commitlintrc.cjs`
- 格式：Prettier（单引号、semi、width 80）+ ESLint + Stylelint

## 样式优先级

**新写或改动的页面/组件样式默认用 Tailwind CSS utility。**  
仅在以下情况保留/新增 SCSS：全局 reset、复杂动画关键帧、Element Plus 深度覆盖、遗留文件渐进迁移。

---
> Source: [zhangbo126/cultural-relics-museum](https://github.com/zhangbo126/cultural-relics-museum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
