## uniapp-vue3-template

> - 页面文件放在 [src/pages/](mdc:src/pages/) 目录下

# uni-app 开发规范

## 页面开发
- 页面文件放在 [src/pages/](mdc:src/pages/) 目录下
- 使用约定式路由，文件名即路由路径
- 页面配置在仅需要在 宏`definePage` 中配置标题等内容即可，会自动生成到 `pages.json` 中

## 组件开发
- 组件文件放在 [src/components/](mdc:src/components/) 或者 [src/pages/xx/components/](mdc:src/pages/xx/components/) 目录下
- 使用 uni-app 内置组件和第三方组件库
- 支持 wot-ui\uview-pro\uv-ui\sard-ui\uview-plus 等多种第三方组件库 和 z-paging 组件
- 自定义组件遵循 uni-app 组件规范

## 平台适配
- **仅支持微信小程序平台**（详见 [wechat-miniprogram-only.mdc](mdc:.cursor/rules/wechat-miniprogram-only.mdc)）
- **不需要平台判断和条件编译**：代码直接针对微信小程序编写
- 使用 uni.xxx API 替代原生 API

## 示例代码结构
```vue
<script setup lang="ts">
import { mpApi } from '@/utils/mp'

const handleClick = () => {
  uni.showToast({
    title: '操作成功',
    icon: 'success'
  })
}
</script>

<template>
  <view class="page">
    <!-- uni-app 组件 -->
    <button @click="handleClick">点击</button>
  </view>
</template>
```

## 生命周期
- 使用 uni-app 页面生命周期
- onLoad、onShow、onReady、onHide、onUnload
- 组件生命周期遵循 Vue3 规范
- 注意页面栈和导航管理
---
globs: src/pages/*.vue,src/components/*.vue
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/workwsl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/workwsl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
