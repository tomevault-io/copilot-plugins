## turborepo-project

> - 包管理: Yarn Workspaces

# 技术栈规范
- 框架: Next.js
- 语言: TypeScript
- 构建工具: Turborepo
- 包管理: Yarn Workspaces
- 3D相关：three.js @react-three/fiber @react-three/drei等相关包
- CSS 框架：Tailwind CSS
- 状态管理：zustand

# TypeScript 规范
- 所有组件必须使用 TypeScript 编写
- 组件 props 必须定义明确的类型接口
- 所有组件必须是函数组件
- 代码生成完需要检查有没有 TypeScript 错误

# Next.js 最佳实践
- 使用 app 目录结构进行路由管理
- 使用 `use client` 明确标记客户端组件，如果使用到客户端的API，一定要在文件开头先标记`'use client'`
- 代码需要符合 Next.js 的最佳实践，优先考虑Next中有没有现成的可以实现的功能
- 函数组件需要使用 memo 进行性能优化

# 目录结构规范
- 项目应用必须放在 apps 目录下
- 共享 UI 组件必须放在 packages/ui 目录
- 通用业务钩子必须放在 packages/shared-hooks 目录，里面不需要多级层级结构，不需要测试代码，不需要src目录，直接在shared-request目录下编写代码
- 工具函数库必须放在 packages/shared-utils 目录，里面不需要多级层级结构，不需要测试代码，不需要src目录，直接在shared-request目录下编写代码
- 请求封装必须放在 packages/shared-request 目录，里面不需要多级层级结构，不需要测试代码，不需要src目录，直接在shared-request目录下编写代码

# Turborepo 规范
- 需要注意项目应用打包的时候是否有依赖 packages 中的包
- 共享代码必须放在 packages 目录下的共享包中

# 组件编写规范
- 写组件的时候需要考虑复用性和扩展性

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Eimi-Fukada) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
