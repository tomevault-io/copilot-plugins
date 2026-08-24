## ktblog

> 本项目不使用 JWT；通过 `token` 表直接存储 token 与用户的对应关系。

# 基本情况
本项目不使用 JWT；通过 `token` 表直接存储 token 与用户的对应关系。

## 要求
- 如果一个函数无法自解释，需要补充 JSDoc。
  - 文档应描述函数作用、参数与返回值。
  - 不要写“你做了什么/怎么做的过程”。
- `console` 的前端设计使用分割线，而不是卡片（其他组件仍使用默认状态）。
- `render`（SolidJS）中读取 resource（如 `post()`）必须包在局部 `<Suspense>` 边界内；
  否则数据加载会挂起 App 顶层包裹页面过渡的 Suspense，打断 Transition 导致路由无法跳转。

---
> Source: [kilokite/ktblog](https://github.com/kilokite/ktblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
