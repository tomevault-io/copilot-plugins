## vue-tailwind-ui

> Vue 页面与组件样式使用 Tailwind CSS（v4）实现


# Vue + Tailwind 样式规范

## 必须

- 新 UI 用 Tailwind utility 写在 template `class` 上；不要新建同目录 `index.scss` 做布局/颜色/间距
- Tailwind 已通过 `@tailwindcss/vite` + `src/style/tailwind.css`（`@import 'tailwindcss'`）接入
- 动态样式（如进度条宽度）用 `:style` 绑定，其余用 class
- 与 Element Plus 共存：表单/弹层优先 Element 组件；外层布局、间距、颜色用 Tailwind

## 推荐写法

```vue
<template>
  <div class="fixed inset-0 z-[2000] flex items-center justify-center bg-black/50 backdrop-blur-sm">
    <p class="text-sm text-white/70">加载中...</p>
  </div>
</template>

<script setup lang="ts">
// ...
</script>
```

## 避免

- 为简单布局再写嵌套 SCSS（`#root { .box { .txt } }`）
- 在组件里重复引入 Tailwind（全局已在 `main.ts` 引入）
- 用 Tailwind 硬覆盖 Element 内部复杂结构时，改用少量 scoped CSS / `:deep()`

## 迁移

改动已有 SCSS 组件时，优先把改动部分迁到 Tailwind；整文件可删则删掉对应 `index.scss` 与 `<style src>`。

---
> Source: [zhangbo126/cultural-relics-museum](https://github.com/zhangbo126/cultural-relics-museum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
