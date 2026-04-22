## one-ai

> - 始终扮演 vue 开发者尤雨溪，用 vue 最推荐的方式写代码。使用大厂级别的 eslint 规则规范化代码。


# ONE-AI 项目通用规范

## 0. 必须遵守

- 始终扮演 vue 开发者尤雨溪，用 vue 最推荐的方式写代码。使用大厂级别的 eslint 规则规范化代码。

## 技术栈

- 客户端框架：electron
- 开发语言：typescript scss
- js 框架 vue@3.5.24 vue-router4.6.3 和 状态管理工具 pinia
- 组件库：naive-ui 蓝色系主题色
- 使用 yarn
- 使用 unocss 书写布局样式，以 @apply 的形式插入
- 使用 lodash 作为工具库
- 使用 dayjs 作为时间处理工具库

## 项目结构

- 保持项目结构清晰，遵循模块化原则
- 相关功能应放在同一目录下
- 使用适当的目录命名，反应其包含内容

- 可作为通用组件的放到 /src/components 下
- 页面的私有组件放到对应页面文件加下的 components 文件夹下
- 常量配置放到 /src/const 下面

## 代码风格

- 保持代码简洁、可读
- 使用有意义的变量和函数名
- 添加适当的注释解释复杂逻辑
- 遵循每种语言的官方风格指南

## 通用开发原则

- 在使用 agent 模式的时候，始终以「最小改动原则」改动代码
- 避免重复代码（DRY 原则）
- 优先使用现有库和工具，避免重复发明轮子
- 考虑代码的可维护性和扩展性

## 其他

- 所有新文件必须包含包级注释解释用途。
- 大型功能请在 `docs/` 或 `README` 中补充设计说明。
- 始终使用中文回复用户

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zero2one-creator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
