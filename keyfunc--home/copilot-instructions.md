## home

> React 19 / TypeScript / Vite 8 / TailwindCSS 4 项目。路由使用 TanStack Router，服务端状态使用 TanStack Query，全局客户端状态按需使用 Zustand，请求层使用 Axios，接口代码由 Orval 生成。

# AGENTS

## 项目背景

React 19 / TypeScript / Vite 8 / TailwindCSS 4 项目。路由使用 TanStack Router，服务端状态使用 TanStack Query，全局客户端状态按需使用 Zustand，请求层使用 Axios，接口代码由 Orval 生成。

## 基础约束

- 包管理统一使用 `pnpm`；依赖变更必须同步 `pnpm-lock.yaml`
- 代码风格以 `biome.json` 为准，不引入 ESLint / Prettier；使用 tab 缩进、双引号、保留分号
- 常用命令：`pnpm dev`、`pnpm build`、`pnpm lint`、`pnpm format`、`pnpm api:generate`、`pnpm api:watch`
- 业务代码优先使用 TypeScript，避免 `any`；能复用已有类型时不要声明等价类型
- 导入 `src` 内模块优先使用 `@/` 别名，并保持 Biome 自动整理 import 的结果
- 新增业务函数、公共类型、组件 props、store state、请求参数与响应类型时，补充简短中文注释
- 不添加无业务依据的兜底、防御性代码或静默吞错；接口契约不清晰时先补类型或明确调用方约束
- 修改生成文件、依赖配置、请求拦截器、路由入口等公共能力时，需要确认影响范围并运行校验

## 目录职责

- `src/router/`：TanStack Router 文件路由、布局、页面 UI、页面级组件
- `src/component/`：跨页面复用组件，目录使用 `PascalCase`
- `src/hook/`：跨页面复用 hook；路由私有 hook 放在对应路由模块内
- `src/service/`：请求基础设施、Orval 生成接口，以及必要时的领域级接口封装
- `src/util/`：通用基础工具
- `public/`：图片、字体、音视频等静态资源

## React 与组件

- 页面和组件使用函数组件；组件文件用 `.tsx`，普通逻辑和类型文件用 `.ts`
- 组件命名使用 `PascalCase`，hook 命名使用 `useXxx`，store hook 命名使用 `useXxxStore`
- 页面 UI 优先写在对应的 `src/router/` 文件路由中；不要为了分层新建 `src/page`
- 路由文件以渲染和交互编排为主；私有请求、状态和副作用抽到当前路由目录的 `-hook/useXxx.ts`
- 只有跨页面复用的 hook 才放到 `src/hook/useXxx.ts`
- 可复用全局组件放在 `src/component/<ComponentName>/index.tsx`
- 单个路由私有组件优先放在该路由目录的 `-component/`
- 项目启用 React Compiler，不要习惯性添加 `useMemo` / `useCallback`；只有引用稳定性是 API 契约或确有性能证据时再使用

## 路由

- TanStack Router 文件路由放在 `src/router/`，使用 `createFileRoute` / `createRootRoute`
- `src/router.gen.ts` 是生成文件，禁止手动编辑
- 路由组件导出名保持为 TanStack Router 约定的 `Route`
- 嵌套路由布局放在对应目录的 `route.tsx`，索引页放在 `index.tsx`
- `src/router/` 下新建与路由生成无关的文件或目录，必须使用 `-` 前缀，例如 `-component/`、`-data/`、`-hook/`、`-utils.ts`

## 数据与状态

- 接口请求统一通过 `src/util/request.ts` 导出的 `request` 实例，不直接创建新的 Axios 实例
- Orval 配置入口为 `orval.config.js`，默认读取 `http://127.0.0.1:8080/openapi.json`
- Orval 生成代码输出到 `src/service/generated/`，禁止手动修改；调整生成结果时修改 OpenAPI 契约、Orval 配置或 mutator
- Orval 请求适配器在 `src/service/mutator/orval.ts`，生成接口必须复用现有 `request`
- 业务侧优先在 hook 中直接复用 `src/service/generated/`；只有跨接口聚合、复杂参数转换或响应标准化时，才新增 `src/service/<domain>/index.ts`
- 服务端状态优先使用 TanStack Query；query key 使用稳定数组并带领域前缀
- Zustand 只存跨页面共享的客户端状态，不存服务端缓存

## 样式

- 样式优先使用 TailwindCSS 工具类，布局优先 `flex` / `grid`
- 全局样式入口是 `src/index.css`，Tailwind 通过 `@import "tailwindcss";` 引入
- 当前视觉以黑白和 `neutral` 色阶为主；新增 UI 必须适配暗色模式，避免突兀的高饱和主色
- 可复用主题值优先沉淀为 CSS 变量
- 组件内 `className` 保持可读；复杂条件样式先整理为清晰变量或小型工具函数
- 响应式优先使用 Tailwind 断点；移动端和桌面端都不能出现文字溢出或控件重叠

## 校验

- 提交信息遵循 Conventional Commits
- 提交前至少运行 `pnpm lint`；涉及类型、路由、构建配置或公共请求层时运行 `pnpm build`
- lint-staged 会对暂存的 JS / TS / JSON / CSS / Markdown 文件执行 `biome check --write`
- 若子目录存在 `AGENTS.md`，以子目录规则为准

---
> Source: [keyfunc/home](https://github.com/keyfunc/home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
