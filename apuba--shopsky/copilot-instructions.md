## shopsky

> 凡是新增、修改或重构 HTML、H5、Vue、uni-app、微信小程序页面，以及页面相关 CSS、组件和图片资源，必须先阅读并遵循：

# ShopSky 开发必读规范

## UI 页面开发前置要求

凡是新增、修改或重构 HTML、H5、Vue、uni-app、微信小程序页面，以及页面相关 CSS、组件和图片资源，必须先阅读并遵循：

`design-system/shopsky-mall/MASTER.md`

该文件是 ShopSky 项目的全局设计系统，内容包括：

- 通用配色、字体、字号和文字层级
- 间距、圆角、边框和阴影
- Banner、商品图、门店图等图片比例
- 搜索框、按钮、卡片、标签和图标组件规范
- uni-app、H5、微信小程序的跨端实现规则
- 响应式、交互、可访问性和页面验收标准

## 强制执行规则

1. 写页面代码前，必须先读取 `design-system/shopsky-mall/MASTER.md`。
2. 设计系统中的颜色、字号、间距、圆角和图片比例是默认标准，不得随意新增或覆盖。
3. 如果某个页面确实需要特殊规则，必须先在 `design-system/pages/` 创建对应页面规范，并说明偏离原因。
4. 页面实现必须优先复用现有组件、设计 token 和资源命名规则。
5. 涉及 uni-app 页面时，必须同时考虑 H5 和微信小程序兼容性。
6. 页面完成后，必须检查文字溢出、图片比例、响应式布局、固定导航遮挡和颜色对比度。
7. 涉及前端构建时，至少验证：
   - `npm run build:h5`
   - `npm run build`

## 目录约定

- 主 uni-app 项目：`apps/user-miniapp`
- H5 与微信小程序项目：`apps/user-miniapp`
- 全局设计系统：`design-system/shopsky-mall/MASTER.md`
- 页面级设计覆盖：`design-system/pages/`
- 公共图片资源：各应用下的 `static` 或 `public` 目录

## 规范优先级

规则优先级从高到低如下：

1. 用户对当前页面的明确要求
2. 当前页面对应的 `design-system/pages/[page-name].md`
3. `design-system/shopsky-mall/MASTER.md`
4. 框架默认行为

如果用户没有明确提出特殊要求，必须严格按照全局设计系统实现。

---
> Source: [apuba/shopsky](https://github.com/apuba/shopsky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
