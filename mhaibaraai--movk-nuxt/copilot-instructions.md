## movk-nuxt

> This file provides guidance for AI coding agents working on this repository.

# AGENTS.md

This file provides guidance for AI coding agents working on this repository.

## 项目概述

`@movk/nuxt` 是一个 Nuxt 4 模块，提供 Schema 驱动的 AutoForm（基于 Zod v4）、API 集成系统（useApiFetch + 进度监控）、独立 UI 组件和通用 Composables。

**peerDependencies**：`@nuxt/ui >=4.6.0`、`zod >=4.3.6`

**moduleDependencies**（模块会自动校验版本）：

| 模块 | 最低版本 |
| --- | --- |
| `@vueuse/nuxt` | >=14.2.1 |
| `@nuxt/image` | >=2.0.0 |
| `@nuxt/ui` | >=4.6.0 |
| `nuxt-auth-utils` | >=0.5.29 |

## 常用命令

```bash
# 开发前必须先执行（生成 stub 和类型）
pnpm dev:prepare

# 启动默认 playground
pnpm dev

# 启动 dashboard playground
pnpm dev:dashboard

# 构建模块
pnpm build

# 类型检查（覆盖 src + 两个 playground + docs）
pnpm typecheck

# Lint
pnpm lint
pnpm lint:fix

# 单元测试
pnpm test

# 运行单个测试文件
pnpm test -- useDateFormatter

# 启动文档站
pnpm docs

# 清理构建产物
pnpm clean
```

**重要**：修改 `src/` 后，如果 playground 未热更新，需重新执行 `pnpm dev:prepare`。

**环境变量**：dashboard playground 通过 `NUXT_PUBLIC_API_BASE` 覆盖 API 基础路径（默认 `/api`）。

## 仓库结构

```
src/
  module.ts              # 模块入口，defineNuxtModule + 配置处理
  theme.ts               # 主题初始化（appConfig.theme/ui 默认值 + theme plugin）
  fonts.ts               # 字体配置（Alibaba PuHuiTi CDN 注入）
  providers/             # 自定义字体 provider（alibaba-puhuiti.ts）
  runtime/
    style.css            # 模块全局样式（自动注入）
    types/               # 所有类型定义（api.ts、auto-form.ts、module.ts、theme.ts）
    constants/           # 运行时常量（api-defaults.ts、auto-form.ts、grid-cols.ts）
    plugins/
      api.factory.ts     # $fetch.create() 多端点实例，含认证/业务检查/Toast 拦截器
      theme.ts           # 主题 plugin（客户端读 localStorage，SSR 注入 anti-flash 脚本）
    composables/         # 自动导入的 composables（useApiFetch、useAutoForm、useTheme 等）
    components/
      auto-form-renderer/ # AutoForm 内部渲染组件（被 ignore 不注册为全局组件）
      input/             # 输入增强组件（WithCopy、WithClear 等）
      theme-picker/      # ThemePicker 组件
      SearchForm.vue     # 带折叠的搜索表单（复用 AutoForm 控件系统）
      AutoForm.vue、DatePicker.vue、StarRating.vue 等独立组件
    auto-form/           # AutoForm 纯逻辑层（不含 Vue 组件）
      schema-introspector.ts  # Zod Schema 解析（字段类型推断）
      controls.ts        # 控件注册表
      metadata.ts        # 字段元数据处理
      field-utils.ts     # 字段工具函数
      provider.ts        # AutoForm 上下文 Provider
      reactive-utils.ts  # 响应式工具
    utils/
      api-utils.ts       # API 工具函数（认证头、Toast、业务检查、数据解包）
      theme.ts           # 主题工具（themeIcons 映射）
      meta.ts            # package.json 元数据读取（自动设置 nuxt.options.site.name）
playgrounds/
  play/                  # 基础功能验证 playground
  dashboard/             # 完整应用场景 playground（含系统管理模块）
docs/                    # 文档站（Nuxt Content）
test/                    # Vitest 单元测试
```

## 关键架构

### 模块初始化流程（`src/module.ts`）

1. `setupTheme()` — 设置 `appConfig.theme`/`appConfig.ui` 默认值，注册 theme plugin；`theme.enabled: false` 时跳过，且 `ThemePicker` 组件也被排除注册
2. `setupFonts()` — 按配置注入字体；`fonts.enabled: false` 时跳过
3. 注册 `#movk` 别名指向 `src/runtime/`（内部代码用 `#movk/types`、`#movk/utils` 等路径引用，避免相对路径嵌套）
4. 注入 `runtime/style.css`
5. `addComponentsDir()` — 以 `options.prefix`（默认 `M`）为前缀注册组件，`auto-form-renderer/**` 始终排除
6. `addImportsDir()` — 自动导入 `runtime/composables/`
7. 若 `api.enabled !== false`：构建 API 运行时配置，将私有字段（`headers`）拆分到 `runtimeConfig.movkApi`，公共字段到 `runtimeConfig.public.movkApi`，注册 `api.factory` plugin
8. 读取宿主 `package.json` 元数据，自动设置 `nuxt.options.site.name`（供 `useSiteConfig()` 使用）

### API 系统

`$api` 类型为 `$Fetch & { use(endpoint: string): ApiInstance }`，通过 `Object.assign($fetchInstance, { use })` 构造，**不是**自定义类包装。

核心逻辑在 `api.factory.ts` 的 `$fetch.create()` 拦截器中：
- `onRequest`：注入认证头（读取 `useUserSession()`）
- `onResponse`：业务状态码检查 + 数据解包（`response.data = extractData(data)`）+ 成功 Toast
- `onResponseError`：HTTP 错误处理 + 错误 Toast + 401 自动处理（清除 session + 跳转登录）

`useApiFetch` 是薄封装，通过 `$fetch` 选项注入 `$api`，通过 `context` 选项传递 `toast`/`skipBusinessCheck`。

**Nuxt hooks 扩展点**（在 `nuxt/app` 的 `RuntimeNuxtHooks` 中声明）：
- `movk:api:request` — 请求发送前（认证注入后），可附加自定义 headers
- `movk:api:response` — 响应成功后（业务检查 + 解包后），可读取解包后的数据
- `movk:api:error` — 任何错误（业务错误 + HTTP 错误）
- `movk:api:unauthorized` — 401 专用，`result.handled = true` 可跳过默认重定向行为

### AutoForm 系统

`useAutoForm()` 返回 `afz`（扩展 Zod 实例，支持 `.meta()` 链式调用）。`afz.object({...})` 定义的 Schema 携带 UI 元数据，`AutoForm.vue` 通过 `auto-form/schema-introspector.ts` 解析字段类型并渲染对应控件。

`SearchForm.vue` 是在 AutoForm 控件系统之上构建的带折叠搜索栏组件，通过 `cols`（支持响应式断点对象）和 `visibleRows` 控制布局，`grid-cols.ts` 中的 `GRID_COL_SAFELIST` 需要同步维护以保证 Tailwind 动态类不被清除。

### 主题系统

`appConfig.ui.colors`（primary/neutral 等）和 CSS 变量（radius、font、blackAsPrimary）通过 `useTheme()` composable 读写，持久化到 `${kebabCase(site.name)}-ui-*` 格式的 localStorage 键。

Theme plugin 采用 anti-flash 模式：SSR 阶段在 `<head>` 注入内联 `<script>`（`tagPriority: -1`），在 hydration 之前从 localStorage 读取并覆盖 CSS 变量，避免主题闪烁。客户端阶段直接修改 `appConfig.ui`。

## TypeScript 注意事项

- 类型扩展使用 `declare module 'nuxt/app'`（不是 `#app`，`vue-tsc` 中 `#app` 别名不可用）
- `useApiFetch<T, DataT>` 双泛型：`T` 是业务数据类型，`DataT` 是 transform 后的类型
- 默认参数需写 `= {} as UseApiFetchOptions<T, DataT>` 以满足泛型约束

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mhaibaraai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
