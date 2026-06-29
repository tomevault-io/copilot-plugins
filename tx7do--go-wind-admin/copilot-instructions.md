## go-wind-admin

> 本文件定义此项目的编码约定和架构规范，供 AI Agent 在生成和修改代码时遵循。

# GoWind Admin — Agent 开发规范

本文件定义此项目的编码约定和架构规范，供 AI Agent 在生成和修改代码时遵循。

## 项目身份

**go-wind-vue3-element-admin** — Vue 3 + Vite + TypeScript + Element Plus 后台管理模板。

## 技术栈

Vue 3.5, TypeScript 5.9, Vite 8, Element Plus 2.x, vxe-table 4.x, Pinia 3, @tanstack/vue-query 5, vue-router 5, vue-i18n 11, UnoCSS + SCSS, vee-validate + zod, axios (gRPC-Web)

## 架构概览

```
src/api/generated/    → gRPC 自动生成（禁止修改）
src/api/client.ts     → ApiClient 单例（懒加载各服务 Client）
src/api/composables/  → Vue Query hooks + 枚举工具（通过 apiClient 调用）
src/components/Pro/   → 配置化 CRUD 组件库（ProPage/ProTable/ProModal...）
src/core/             → transport, i18n, router, access
src/pages/app/        → 业务页面（按模块分目录）
src/locales/          → i18n 翻译资源（zh-CN / en-US）
src/router/routes/    → 路由配置（modules/app/*.ts 为动态路由）
```

## 规则

### TypeScript
- 接口无 I 前缀（`User` 非 `IUser`）
- 路由类型用 `RouteRecordRaw`
- 路由数组 map+sort 后断言 `RouteRecordRaw[]`
- 类型窄化用 `typeof`，不用 `isFunction`
- `defineExpose` 暴露属性直接访问

### 国际化（强制）
- 所有可见文本用 `$t()`/`t()`，禁止硬编码中文
- `$t()` 用于模板和 computed；`t()` 用于 composable 顶层
- key 命名：`pages.<module>.<field>` / `enum.<module>.<field>.<VALUE>` / `routes.<module>.<page>` / `common.<category>.<key>`

### API 分层规则

**ApiClient 单例** (`src/api/client.ts`):
- 全局唯一实例，通过 `ClientTransport` 适配 axios 请求
- 懒加载属性访问器按需创建各服务 Client（如 `apiClient.userService`）
- protobuf 重新生成后自动包含新服务

**Composable 层** (`src/api/composables/`):
- 从 `generated/` 只导入类型（`type` import），运行时调用通过 `apiClient`
- 导出：`use*` Hook + `fetch*` 函数 + 枚举工具
- queryKey：`["操作名", 参数]`
- 列表查询：`apiClient.xxxService.List(query.toRawParams())`
- **创建参数用 `{ data: {...} }` 包裹**
- **更新操作用 `makeUpdateMask` 生成掩码**
- 枚举列表：`computed(() => [...])` + `t()`
- 在 `composables/index.ts` 添加 `export *`

### 组件
- `ElDrawer` 必须设置 `:append-to-body="true"`
- `ElDialog` 的 `appendTo` 是字符串选择器
- `ElTreeSelect` value 不接受 undefined，用 null
- 暗黑模式文本色用 `var(--el-text-color-*)`

### 样式
- CSS 变量前缀 `--gowind-*`（避让 `--el-*`）
- 主题色存 HSL 数值
- 用 `style.setProperty()` 更新变量

### 路由
- 动态路由：`router/routes/modules/app/<module>.ts`
- 顶层用 `Layout` 包裹
- `meta.title` 用 i18n key，`meta.icon` 用 `lucide:` 前缀
- `meta.authority` 控制权限
- `export default routes`

## 新建 CRUD 模块步骤

1. `src/api/composables/<module>.ts` — hooks + 枚举（通过 apiClient 调用）
2. `src/api/composables/index.ts` — 添加 `export *`
3. `src/locales/{zh-CN,en-US}/pages/<module>.json` — 页面翻译
4. `src/locales/zh-CN/enum.json` — 枚举翻译
5. `src/locales/zh-CN/routes.json` — 路由标题
6. `src/router/routes/modules/app/<module>.ts` — 路由
7. `src/pages/app/<module>/<module>/index.vue` — ProPage 列表页
8. `src/pages/app/<module>/<module>/<module>-drawer.vue` — useProModal 弹窗

## 常见陷阱

- gRPC 创建接口：必须 `{ data: {...} }` 包裹
- PowerShell 命令连接：用 `;` 不用 `&&`
- `ElLink` underline：废弃 boolean，用字符串
- vue-i18n `$te`：不支持 ns 选项对象

## 构建命令

```bash
pnpm dev          # 开发服务器
pnpm build        # 类型检查 + 构建
pnpm type-check   # 类型检查
pnpm lint         # 全量 lint
```

Node: `^20.19.0 || >=22.12.0` | 包管理器: pnpm | Git: Conventional Commits

---
> Source: [tx7do/go-wind-admin](https://github.com/tx7do/go-wind-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
