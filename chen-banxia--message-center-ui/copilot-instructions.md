## components

> 项目使用 Vue 3 的组合式 API 和 `<script setup>` 语法。组件文件应遵循以下结构：

# 组件规则

## 组件结构

项目使用 Vue 3 的组合式 API 和 `<script setup>` 语法。组件文件应遵循以下结构：

```vue
<script setup>
// 导入
import { ref } from 'vue'

// 状态和逻辑
const count = ref(0)
</script>

<template>
  <!-- 模板内容 -->
</template>

<style scoped>
/* 组件样式 */
</style>
```

---
> Source: [chen-banxia/message-center-ui](https://github.com/chen-banxia/message-center-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
