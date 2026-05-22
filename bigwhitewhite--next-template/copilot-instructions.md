## next-web

> description: Next官网开发基本要求

---
description: Next官网开发基本要求
globs: 

alwaysApply: false
---
### 🌐 NEXTJS-OPTIMIZED PROJECT STRUCTURE

电脑环境: window

node: 20 version,可以使用比较高版本对应的第三方库

包管理器使用: pnpm

**集成国际化、响应式设计与专业开发规范**

读取理解当前项目结构和代码思路

开发要求

```tsx
// 开发铁律：
1. 移动端优先原则：
   - 默认样式为移动端 (无断点前缀)
   - PC增强使用 `lg:` 和 `3xl:`

2. 组件差异处理：
   ```jsx
   {/* 设计稿差异大 → 响应式覆盖 */}
   <div className="block md:hidden">移动端内容</div>
   <div className="hidden md:block">PC端内容</div>
   {/* 设计稿差异小 → 响应式布局 */}
   <div className="block lg:flex"></div>
```

## 国际化要求

请注意!!!所有的文案都需要使用国际化!!!

使用next-intl

参考文档: https://next-intl.dev/

## Figma要求

```text
1. 读取编写同一个界面两种设计,例如Home的两个设计稿移动端和pc端,需要在同一个page.tsx中做出兼容
2. 全局化组件例如header和footer需要封装到项目/components/...中
3. 移动端菜单需要做展开收起效果
4. 逐页、逐区块解析 Figma 设计稿的结构、内容、样式、交互。
5. 为每个页面和区块生成对应的 React 组件、Tailwind 样式、图片、动效等。
6. 还原所有细节,包括字体、配色、间距、响应式断点、动效、UI 组件等。
```

---
> Source: [bigWhiteWhite/next-template](https://github.com/bigWhiteWhite/next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
