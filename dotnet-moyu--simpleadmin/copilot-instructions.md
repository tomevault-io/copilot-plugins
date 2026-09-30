## simpleadmin

> - `api`：.NET 后端模块，解决方案位于 `api/SimpleAdmin/SimpleAdmin.sln`，核心项目包括 `SimpleAdmin.Application`、`SimpleAdmin.Core`、`SimpleAdmin.System`，启动项目为 `SimpleAdmin.Web.Entry`。

# 仓库指南

## 项目结构与模块组织
本仓库由 3 个相对独立的模块组成：

- `api`：.NET 后端模块，解决方案位于 `api/SimpleAdmin/SimpleAdmin.sln`，核心项目包括 `SimpleAdmin.Application`、`SimpleAdmin.Core`、`SimpleAdmin.System`，启动项目为 `SimpleAdmin.Web.Entry`。
- `web`：Vue 3 + Vite 管理端，核心代码位于 `web/src/`（如 `api`、`views`、`stores`、`components`、`routers`、`utils`）。
- `uniapp`：Uni-app 移动端，核心代码位于 `uniapp/src/`（如 `pages`、`api`、`store`、`router`、`layouts`、`utils`）。

提交时尽量只改动目标模块，非必要不要跨模块修改。

## 构建、测试与本地开发命令
请在对应模块目录执行：

- 后端构建：`cd api/SimpleAdmin && dotnet restore && dotnet build SimpleAdmin.sln`
- 后端运行：`cd api/SimpleAdmin && dotnet run --project SimpleAdmin.Web.Entry`
- Web 启动：`cd web && npm install && npm run dev`
- Web 检查：`cd web && npm run type:check && npm run lint:eslint && npm run build:pro`
- Uniapp 启动（H5）：`cd uniapp && pnpm install && pnpm dev:h5`
- Uniapp 检查：`cd uniapp && pnpm type-check && pnpm lint && pnpm build:h5`

## 仓库路径映射说明（避免目录歧义）
- 文档中提到“后端模块”默认指 `api` 目录（具体为 `api/SimpleAdmin` 解决方案）。
- 文档中提到“管理端 / Web 模块”默认指 `web` 目录。
- 文档中提到“移动端 / Uniapp 模块”默认指 `uniapp` 目录。
- 若未来目录有迁移，优先同步更新本文件的“项目结构与模块组织”和“构建、测试与本地开发命令”两节。

## 代码风格与命名约定
- 提交前遵循各模块 `.editorconfig` 与格式化配置。
- 前端统一使用 UTF-8、LF、空格缩进；`web` 使用 ESLint + Prettier + Stylelint，`uniapp` 使用 ESLint + Prettier。
- 后端 C# 命名（见 `.editorconfig`）：私有/内部字段使用 `_camelCase`；常量和静态只读字段使用 `ALL_UPPER`。
- Vue 目录建议按功能分组，页面或组件入口优先使用 `index.vue`。

## Web 新增页面必做清单（web）
新增管理端页面时，至少完成以下文件与同步项：

- 页面文件：`web/src/views/<域>/<功能>/index.vue`（例如 `web/src/views/biz/ops/test/index.vue`）。
- 页面子组件：如有弹窗/表单/授权组件，放 `web/src/views/<域>/<功能>/components/*.vue`。
- API 请求封装：`web/src/api/modules/<域>/<功能>.ts`。
- API 类型定义：`web/src/api/interface/<域>/<功能>.ts`。
- 导出聚合更新：同步更新以下 `index.ts`，保证可通过 `@/api` 导入：
  `web/src/api/modules/<域>/index.ts`、`web/src/api/interface/<域>/index.ts`，必要时继续上抛到 `web/src/api/modules/index.ts`、`web/src/api/interface/index.ts`。
- 菜单资源配置（必须）：在“菜单管理”中新增资源并正确填写：
  `path`（路由地址）、`name`（组件名，对应 `script setup name`）、`component`（相对 `web/src/views/` 的路径，不带 `.vue`）。
- 动态路由约束：`component` 最终必须能匹配 `@/views/**/*.vue`，否则动态路由无法加载。

推荐页面最小骨架：
- `index.vue`（列表/入口）
- `components/form.vue`（新增/编辑弹窗）
- `<feature>.ts`（接口）
- `<feature>.ts`（类型）

## 后端新增服务与控制器必做清单（System + Biz）
新增后端功能时，按“服务层 -> 控制器 -> 路由契约”落地：

- 服务放置规范：
  - 系统能力：`api/SimpleAdmin/SimpleAdmin.System/Services/<域>/<功能>/` 下新增 `I<Feature>Service.cs` 与 `<Feature>Service.cs`。
  - 业务聚合能力：`api/SimpleAdmin/SimpleAdmin.Application/Services/<域>/<功能>/` 下新增对应接口与实现。
- 服务接口规范：接口必须继承 `ITransient`；实现类使用构造函数注入，不在控制器中写复杂业务。
- DTO 规范：输入输出模型放在对应服务目录 `Dto/`（如 `MenuInput.cs`），控制器仅负责参数接收与调用服务。
- 控制器放置规范：
  - System 控制器：`api/SimpleAdmin/SimpleAdmin.Web.Core/Controllers/System/...`。
  - Biz 控制器：`api/SimpleAdmin/SimpleAdmin.Web.Core/Controllers/Application/...`。
- 控制器声明规范（必须）：
  - 类上声明 `[Route("...")]` 与 `[ApiDescriptionSettings(Tag = "...")]`。
  - 方法上声明 `[HttpGet]/[HttpPost]`、`[DisplayName("...")]`。
  - 按权限模型选择特性（如 `[SuperAdmin]`、`[RolePermission]`）。
- 路由一致性：前端 API 前缀必须与控制器路由一致（例如 `/sys/limit/role/*` 对应 `RoleController`）。

## 前后端联调最小闭环（提交前）
- 页面可从菜单进入，刷新后仍可进入（验证动态路由与权限菜单）。
- 页面接口全部走 `web/src/api/modules/...` 封装，不直接散落请求。
- 新增接口在前端有对应类型定义，避免 `any` 漫延。
- 至少验证一个“新增/编辑/删除或详情”主流程与一个权限受限场景（无权限提示或拦截正确）。

## 测试说明
当前仓库未配置独立自动化测试项目。每个 PR 至少应完成：

- 对改动模块执行 lint、type-check、build。
- 手工验证关键流程（登录、路由跳转、权限控制、相关 CRUD/API）。
- 若新增后端测试，请创建独立 `*.Tests` 项目，并在 PR 中补充运行命令与结果。

## 提交与合并请求规范
- 使用 Conventional Commits：`type(scope): summary`。
- 推荐类型：`feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`build`、`ci`、`revert`。
- `web` 与 `uniapp` 已启用 `commitlint` 与 `lint-staged`，推送前确保本地钩子通过。
- PR 必须包含：变更目的、影响模块、验证步骤/命令、关联 Issue；涉及界面改动请附截图。

---
> Source: [DotNet-MoYu/SimpleAdmin](https://github.com/DotNet-MoYu/SimpleAdmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
