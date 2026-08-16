## buildadmin

> 本文件为 AGENTS 在当前代码库中工作时提供指导。

# AGENTS.md

本文件为 AGENTS 在当前代码库中工作时提供指导。

## 技术栈

后端：PHP8 + ThinkPHP8 + MySQL
前端：Vue3 + Element Plus + TypeScript + Vite + Pinia + Axios

## 回答偏好

- 回复使用中文
- 遇到有多种实现方案时，列出选项让我选择，而不是直接选一种

## 关键约定

- **源代码文件**：统一采用 UTF-8（无 BOM）编码；换行符统一使用 LF（\n）；文件末尾保留一个空行，行尾禁止空白空格
- **重写继承的后台 CRUD 方法**：将 `app/admin/library/traits/Backend.php` 中的方法复制到目标控制器后进行改写，不要直接编辑 trait。

## 常用命令

### 后端（PHP，在仓库根目录执行）

- 安装依赖：`composer install`
- 开发服务器：`php think run` -> 启动 `http://localhost:8000`（前端开发环境请求该地址）。
- 数据库迁移（通过 `think-migration` 集成的 `Phinx`）：`php think migrate:run` 命令可应用全部待执行的迁移。

### 前端（在 web/目录执行，pnpm 为当前包管理器，不固定，根据 web/pnpm-lock.yaml、web/package-lock.json 识别）

- 安装依赖：`pnpm install`
- 开发服务：`pnpm dev` -> 执行 `esno ./src/utils/dev.ts`，该脚本会启动 `vite --force`。开发服务器端口为 `1818`（见 `web/.env`）。
- 构建：`pnpm build` -> `vite build`，构建产物输出到 `web/dist`；构建产物需要手动移动到 `public/` 以供生产环境使用。
- 代码检查：`pnpm lint` / `pnpm lint-fix`（ESLint flat config）。格式化：`pnpm format`（Prettier）。类型检查：`pnpm typecheck`（`vue-tsc --noEmit`）。

## 后端架构

### 多应用模式

- 采用多应用模式（`topthink/think-multi-app`）。应用包括：`admin`（后台管理）、`api`（会员端 API + 安装器）、`common`（共享基类，不可通过 Web 访问，已列入 `deny_app_list`）。
- 入口 `public/index.php` 会区分前端请求（返回已编译的 `public/index.html`）与 API 请求（通过 `server` 参数/头、`index.php` URI 前缀或 OPTIONS 请求判定）。**在常驻内存运行时（Workerman 模块）下不走本文件**。

### 控制器继承链

- `app/BaseController.php` - ThinkPHP 框架基类。在构造方法中自动将 `controllerPath` 写入 Request（将 `.` 替换为 `/`，如 `auth.Admin` -> `auth/Admin`），并调用 `initialize()`。
- `app/common/controller/Api.php` - 继承 `BaseController`。通过 `success()` / `error()` / `result()` 返回 JSON，这些方法均抛出 `HttpResponseException`。统一响应结构：`{code, msg, time, data}`；`code == 1` 表示成功。`initialize()` 中会检查数据库连接、执行 `ip_check()` 与时区设定，并加载控制器语言包。
- `app/common/controller/Backend.php`（继承 `Api`）- 所有后台控制器的基类。初始化后台鉴权（`app\admin\library\Auth`），读取 `batoken` 请求头，并通过 `$noNeedLogin` / `$noNeedPermission` 数组强制登录与 RBAC 校验。权限校验的键为 `controllerPath/action`。它 `use` 了 `Backend` trait（`app/admin/library/traits/Backend.php`），该 trait 提供了默认的 `index / add / edit / del / sortable / select` 方法，完全由 `$this->model` 及一组可配置属性驱动（`$preExcludeFields`、`$quickSearchField`、`$dataLimit`、`$dataLimitField`、`$defaultSortField`、`$orderGuarantee`、`$indexField`、`$withJoinTable`、`$modelValidate`、`$modelSceneValidate`）。
- `app/common/controller/Frontend.php`（继承 `Api`）- 会员端控制器基类；使用 `app\common\library\Auth` 与 `ba-user-token` 请求头。

一个典型的后台控制器（如 `app/admin/controller/auth/Admin.php`）只需在 `initialize()` 中设置 `$model`，并重写需要自定义逻辑的 CRUD 方法，其余均继承自基类。`Backend::queryBuilder()` 会将前端的 `search` / `quickSearch` / `order` / `limit` / `initKey` 参数转换为 ThinkPHP 的 `where` / `order` / `paginate`；`getDataLimitAdminIds()` 负责数据权限范围过滤（`personal` / `allAuth` / `allAuthAndOthers` / `parent` / 分组 id）。

### 中间件链

- 全局（`app/middleware.php`）：`Throttle`（限流，配置文件位于 `config/throttle.php`）。
- Admin 应用（`app/admin/middleware.php`）：`AllowCrossDomain` → `AdminLog` → `LoadLangPack`。
- API 应用（`app/api/middleware.php`）：`AllowCrossDomain` → `LoadLangPack`。
- `AllowCrossDomain`（`app/common/middleware/AllowCrossDomain.php`）：根据 `buildadmin.cors_request_domain` 配置动态设置 `Access-Control-Allow-Origin`。
- `AdminLog`（`app/common/middleware/AdminLog.php`）：当 `auto_write_admin_log` 开启时，自动记录所有 POST/DELETE 请求到 `AdminLogModel`。
- 添加应用时建议首先添加 `AllowCrossDomain` 中间件，否则前端请求将报跨域。

### CRUD 代码

- 系统有可视化CRUD功能，它是 `app/admin/library/crud/Helper.php` 和 `app/admin/controller/crud/Crud.php` 根据后台 UI 中的图形化拖拽设计器提供的数据，生成：控制器 + 模型 + 验证器 + 视图 + 语言包文件，创建数据表，并写入 `admin_rule` 菜单/权限节点。
- 样板代码文件位于 `app/admin/library/stubs/` 与 `app/admin/library/crud/stubs/`。生成的控制器继承 `Backend`，因此立即具备完整 CRUD 能力；生成完毕后会通过 WEB 终端自动对前端代码执行 prettier。
- CRUD 生成器生成的内容包括（`path/name` 是前端自定义的生成路径）：写入到 `admin_rule` 表的菜单/权限节点、`app/admin/controller/path/name.php`（控制器）、`app/admin/model/path/name.php`（模型，还可以生成到 `app/common/model` 目录），`app/admin/validate/path/name.php`（验证器），`src\lang\backend\{lang}\path/name.ts`（前端语言包，默认有 en 和 zh-cn 两种语言支持），`src\views\backend\path\name\index.vue`（路由入口文件、CRUD表格组件），`src\views\backend\test\popupForm.vue`（表格表单组件）

### 其他要点

- 辅助函数位于 `app/common.php`：`__()`（多语言）、`get_sys_config()`、`full_url()`、`get_auth_token()`、`filter()` / `clean_xss()`、`hash_password()` / `verify_password()`、`action_in_arr()`、`get_route_remark()`。
- 核心库类集位于 `extend/ba/`（通过 `"" => "extend/"` 以 `PSR-0` 自动加载）：`Auth`、`Captcha`、`ClickCaptcha`、`Filesystem`、`TableManager`、`Terminal`、`Tree`、`Version`、`Random`、`Depends`。
- 模型位于 `app/admin/model/` 与 `app/common/model/`；验证器通过将类名中的 `\model\` 替换为 `\validate\` 自动发现。
- 迁移文件位于 `database/migrations/`（一个 `install` 迁移 + 各版本迁移）。
- 配置文件全部位于 `config/` 目录。

## 前端核心架构

### 路径别名与入口

- `/@/` → `src/`（tsconfig `paths` 与 vite `resolve.alias` 一致）。
- 入口 `src/main.ts`：注册 pinia（带 `pinia-plugin-persistedstate`）、router、element-plus、i18n、全局 icon。
- 路由用 `hash` 模式（`createWebHashHistory`）。静态路由位于 `src/router/static.ts`（自动加载 `./static/*.ts` 以获取 `adminBase` 与 `memberCenterBase` 基础路由）。菜单路由从后端拉取，并由 `src/utils/router.ts`（`addRouteAll` / `addRouteItem`）在运行时注册，通过 `import.meta.glob('/src/views/{backend,frontend}/**/*.vue')` 匹配组件。权限节点被组装进 `authNode` Map，供 `auth()` 公共函数使用。

### 状态管理

Pinia stores 全部位于 `src/stores/`，通过 `pinia-plugin-persistedstate` 持久化：
- `adminInfo` — 管理员 token 与基本信息。
- `userInfo` — 会员 token 与基本信息。
- `siteConfig` — 站点全局配置（站点名、CDN、上传配置、备案号等），从后端加载。
- `config` — 布局/主题/语言配置。包含完整的布局参数（多种布局模式、菜单颜色/宽度、暗黑模式、顶栏配置等），通过 `getColorVal()` 根据 `isDark` 返回配色数组中的对应值。
- `navTabs` — 后台标签页管理。维护 `tabsView`（已打开的标签）、`tabsViewRoutes`（菜单路由树）、`childrenMenus`（次级菜单）、`authNode`（权限节点 Map）。
- `memberCenter` — 会员中心的路由和权限节点。
- `terminal` — WEB 终端相关状态。

### HTTP

- `src/utils/axios.ts` 是自定义的 axios 封装。它会自动注入 `batoken`（后台）/ `ba-user-token`（会员）请求头，在收到 **`code === 409`** 时调用 `refreshToken` 并将进行中的请求排队重放，并根据 `code === 302`（重定向）/ `code === 303`（需登录 -> 清除 token 并跳转登录页）进行路由。重复请求会被自动取消。`VITE_AXIOS_BASE_URL` 为后端地址（开发环境 `http://localhost:8000`，生产环境 `getCurrentDomain`）。当后台入口路径非默认时，axios 会将 `/admin/...` 重写为 `{path}.php/...`。
- 新增 `API` 请求函数放 `src/api/`，参考 `src\api\common.ts`。

### 表格

`src\components\table` 基于 Element Plus `el-table` 封装，需与 `src\utils\baTable.ts` 配合使用。

- **`baTable.ts`**：表格管家类，它是表格数据管理器，其中封装好了一个表格应有的 属性 和 方法，预设了 查看前、查看后、编辑前 等钩子，在需要表格的组件中实例化后，`provide` 给子组件
- **`index.vue`**：表格主体，遍历 `baTable.table.column` 渲染列，支持 `slot` 自定义渲染
- **`header/index.vue`**：表头工具栏，支持刷新、编辑、删除、公共搜索、快速搜索、列显示等按钮
- **`comSearch/index.vue`**：公共搜索组件，支持多字段组合条件搜索
- **`types\table.d.ts`**：表格类型定义（表格列可用属性定义、各种操作前后钩子、公共搜索可用操作符号、表格表单数据等）
- **`fieldRender/`**：单元格渲染器，一个文件等于一个渲染器，使用列配置的 `render` 属性指定渲染器（`buttons,color,customRender,customTemplate,datetime,icon,image,images,switch,tag,tags,url,slot`）

`baTable` 列配置（`column` 数组）关键属性：

| 属性                    | 说明                                     |
|-----------------------|----------------------------------------|
| `prop`                | 字段名，支持 `.` 嵌套（如 `user.nickname`        |
| `label`               | 列标题                                    |
| `render`              | 渲染器类型（`image`、`tag`、`switch`、`slot` 等） |
| `operator`            | 搜索操作符（`eq`、`LIKE`、`RANGE`、`IN` 等）      |
| `formatter`           | 自定义格式化函数                               |
| `showOverflowTooltip` | 超出显示 `tooltip`                         |

### 输入组件

我们封装了 `src\components\baInput\index.vue` 输入组件和 `src\components\formItem\index.vue` 表单项组件，都可以通过传递 `type` 属性渲染对应的输入框，如：`<FormItem label="array" type="array" v-model="state.array" />`，支持的所有 type 可以查阅：`src\components\baInput\index.ts` 中的 `inputTypes`

### 国际化（vue-i18n）

语言包位于 `src/lang/`——`globs-{locale}.ts`（全局根层）、`common/{locale}/`（公共语言包，总是加载）、`backend/{locale}/`（后台语言包，仅后台加） + `backend/{locale}.ts`、`frontend/{locale}/`（前台语言包，仅前台加载） + `frontend/{locale}.ts`。关键流程：

1. **初始化**（`lang/index.ts` 的 `loadLang()`）：加载 Element Plus 语言包 + 全局 `globs-{locale}.ts` + 所有 `common/{locale}/` 下的语言包文件（eager import），合并后创建 i18n 实例。同时将 `backend/{locale}/**` 和 `frontend/{locale}/**` 的 glob 句柄存入 `window.loadLangHandle`，供后续按需加载。
2. **按需加载**（`router/index.ts` 的 `beforeEach`）：每次路由切换时，根据 `langAutoLoadMap`（`src/lang/autoload.ts`）的静态映射 + 路由 path 推导的命名空间，调用 `loadAndMergeMessages()` 异步加载对应语言包文件并合并到 i18n 实例。

### 样式

`src/styles/` 使用 SCSS 管理样式，入口 `index.scss`。

- **`var.scss`**：CSS 变量定义（`--ba*` 前缀）
- **`element.scss`**：Element Plus 组件样式覆盖
- **`dark.scss`**：暗黑模式样式
- **`loading.scss`**：全局加载样式
- **`app.scss`**：应用基础样式

总是优先使用 element plus 的 `--el` 开头和框架自定义的 `--ba` 开头的 CSS 变量。

### 前端鉴权

`utils/common.ts` 中的 `auth()` 函数根据当前路由 path 在 `authNode` Map 中查找权限节点；也可以提供路由的 name 对象进行鉴权。

### 工具

`src/utils/` 存放通用工具函数。

- **`axios.ts`**：axios 封装
- **`common.ts`**：通用工具（`fullURL` 资源地址拼接、`auth` 鉴权等）
- **`router.ts`**：路由工具（`handleAdminRoute` 动态路由注册、`openMenu` 菜单打开、菜单处理、权限节点组装）
- **`layout.ts`**：布局工具（`mainHeight` 计算内容区高度、`setNavTabsWidth` 设置导航标签宽度）
- **`storage.ts`**：本地存储封装（`localStorage` 和 `sessionStorage`）
- **`validate.ts`**：表单验证规则
- **`random.ts`**：随机数生成
- **`horizontalScroll.ts`**：横向滚动辅助

---
> Source: [build-admin/buildadmin](https://github.com/build-admin/buildadmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
