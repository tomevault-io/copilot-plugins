## project-overview

> 这是一个名为 `@wolforest/jscommon` 的 JavaScript/TypeScript 工具库 monorepo 项目。

# JSCommon 项目概览

这是一个名为 `@wolforest/jscommon` 的 JavaScript/TypeScript 工具库 monorepo 项目。

## 项目目标
提供一个统一的工具库，集成多个常用的 JavaScript 库，支持完整的 Tree-shaking，为前端应用提供所需的全部工具函数。

## 核心特性
- 📦 支持完整的 Tree-shaking，真正按需加载
- 🔧 完整的 TypeScript 类型支持  
- 🎯 统一的 API 入口，无需重复安装多个依赖
- 🚀 零配置，开箱即用
- 📚 完整的类型提示

## 项目结构
- [package.json](mdc:package.json) - 根目录包配置，定义了 monorepo 的脚本和依赖
- [packages/core/](mdc:packages/core) - 核心包，包含所有工具函数的实现
- [examples/](mdc:examples) - 示例项目，包含 React、Vue、Taro 等框架的使用示例
- [README.zh_CN.md](mdc:README.zh_CN.md) - 详细的中文文档

## 集成的工具库
项目集成了以下常用工具库：
- lodash-es (通用工具)
- dayjs (日期处理)
- ahooks (React Hooks)
- ramda (函数式编程)
- axios (网络请求)
- @vueuse/core (Vue 工具)
- qs & query-string (URL 处理)
- pinyin-pro (中文处理)
- big.js (数值计算)
- classnames (样式处理)

---
> Source: [wolforest/jscommon](https://github.com/wolforest/jscommon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
