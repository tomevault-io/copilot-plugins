## openspec-practice

> `Practice`是一个`Go`语言管理后台系统，采用前后端分离架构。

# 项目概述

`Practice`是一个`Go`语言管理后台系统，采用前后端分离架构。

- **前端**: `Vben5 + Vue 3 + Ant Design Vue + TypeScript（pnpm monorepo）`
- **后端**: `GoFrame + MySQL + JWT`
- **参考项目**: `/Users/john/Workspace/github/imdap/ruoyi-plus-vben5`（前端样式和功能交互参考）

## 默认账号

- 用户名称: `admin`
- 登录密码: `admin123`

## 目录结构

```text
apps/                → MonoRepo项目目录
  practice-core/         → GoFrame框架实现的后端源码
    api/             → 请求/响应 DTO（g.Meta 路由定义）
    internal/        → 后端核心代码实现
      cmd/           → 服务启动 & 路由注册
      consts/        → 全局常量定义
      controller/    → HTTP控制器（gf gen ctrl 自动生成骨架）
      dao/           → 数据访问层（gf gen dao 自动生成）
      model/         → 数据模型
        do/          → 数据操作对象（自动生成）
        entity/      → 数据库实体（自动生成）
      service/       → 业务逻辑层
    manifest/        → 交付清单
      config/        → 后端配置文件
      sql/           → DDL + Seed DML（版本 SQL 文件）
        mock-data/   → Mock 演示/测试数据（不随生产部署）
  practice-vben/         → Vben5 前端（pnpm monorepo）
    apps/web-antd/   → 主应用（Ant Design Vue）
    packages/        → 共享库（@core, effects, stores, utils 等）
hack/                → 项目脚本及测试用例文件
  tests/             → E2E 测试（Playwright）
    e2e/             → 测试用例文件
    fixtures/        → 测试 fixtures（auth, config）
    pages/           → 页面对象模型
openspec/            → OpenSpec相关文档
  changes/           → OpenSpec变更记录
```

## 常用命令

### 开发环境

```bash
make dev                         # 启动前后端（前端:5666, 后端:8080）
make stop                        # 停止所有服务
make status                      # 查看服务状态
make test                        # 运行完整E2E测试
make init                        # 初始化数据库（DDL + Seed 数据）
make mock                        # 加载 Mock 演示数据（需先执行 init）
make up                          # 默认用 claude 生成 commit message 并推送
make up tool=codex               # 使用 codex 生成 commit message 并推送
make up t=codex                  # tool 的短别名
make up tool=codex               # codex 默认模型为 gpt-5.1-codex-mini
make up tool=codex model=gpt-5.2 # 指定 AI 工具和模型（兼容 m=...）
```

### 后端

```bash
cd apps/practice-core
go run main.go          # 运行
make build              # 构建
make dao                # 生成 DAO/DO/Entity（修改 SQL 后）
make ctrl               # 生成控制器骨架（修改 API 定义后）
```

### 前端

```bash
cd apps/practice-vben
pnpm install                   # 安装依赖
pnpm -F @practice/web-antd dev     # 开发模式
pnpm run build                 # 构建
```

### E2E 测试

```bash
cd hack/tests
pnpm test              # 运行全部测试
pnpm test:headed       # 带浏览器界面运行
pnpm test:ui           # 交互式测试界面
pnpm test:debug        # 调试模式
pnpm report            # 查看 HTML 报告
```

测试文件命名规范：`TC{NNNN}*.ts`（如 `TC0001-login.ts`），放在 `hack/tests/e2e/` 对应模块目录下。

# 开发流程规范

本项目采用`SDD`驱动开发，使用`OpenSpec`工具辅助落地。变更记录存放在 `openspec/changes/` 目录下。每个变更包含：`proposal.md`（提案）、`design.md`（设计）、`specs/`（增量规范）、`tasks.md`（任务清单）。

**执行流程**：
1. 通过`/opsx:explore`斜杠指令在给定需求描述的前提下进行探索式对话，分析问题、设计方案、评估风险。
2. 当探索式对话结束，形成清晰的解决方案时，通过`/opsx:propose`斜杠指令将其转化为正式的`OpenSpec`变更提案文档。命令形如`/opsx:propose feature-name`，其中`feature-name`为当前变更的描述性名称（使用`kebab-case`格式，如`user-auth`、`data-export`）。随后会在`openspec/changes`目录下会自动生成一个新的变更文件夹，包含增量规范系列文档(`spec/`)、技术实现方案(`design.md`)、变更提案与思路(`proposal.md`)和实现任务清单(`tasks.md`)。
3. 随后执行`/opsx:apply`开始按照`tasks.md`中的任务清单逐条执行，完成代码实现、测试、文档更新等工作。任务完成后需要调用`/openspec-review`技能进行代码和规范审查。如果涉及前端页面交互的功能，那么都需要创建`e2e`测试用例，并且在执行过程中自动运行测试用例，确保功能实现的正确性。
4. 用户反馈的问题或者改进点，需要调用`/openspec-feedback`技能进行修复和验证，并更新相关`OpenSpec`文档。任务完成后需要调用`/openspec-review`技能进行审查。
5. 用户确认本次迭代功能已完成没有问题后，则执行`/opsx:archive`斜杠指令将本次变更归档。归档前需要调用`/openspec-review`技能进行全面的变更审查，确保代码质量和规范遵循。

**关键规则**：
- 当用户报告问题缺陷/改进建议时（无论中文或英文），如果当前项目存在活跃的`OpenSpec`变更，那么必须调用`openspec-feedback`技能。**无论反馈内容是否与当前活跃迭代的主要功能相关，都必须追加到当前活跃迭代中**，便于统一管理和归档。
- 审查技能`/openspec-review`自动在以下节点触发：`/opsx:apply`任务完成后、`/opsx:feedback`任务完成后、`/opsx:archive`归档前。

# 架构设计规范

## 模块设计规范

### 模块功能设计规范

1. 业务模块的枚举值都应当使用字典模块维护其对应的字典类型和字典数据，而不是在代码中硬编码枚举值。比如：操作日志中的操作类型、操作结果，登录日志中的登录状态、文件管理的业务场景等都应该使用字典类型进行维护。

### 模块解耦设计原则

所有前后端模块必须采用解耦设计，业务模块支持按需启用/禁用。设计和实现时须遵循以下原则：

1. **模块可禁用**：每个业务模块（如部门、岗位、字典等）应当是独立的，可以通过配置禁用。禁用某模块后，所有依赖该模块的功能必须自动降级或隐藏，不能出现报错或空白区域。
2. **前端联动隐藏**：当一个模块被禁用时，前端所有涉及该模块的`UI`元素（菜单项、表单字段、表格列、搜索条件、按钮等）必须完全隐藏，而非仅禁用或置灰。例如：禁用"部门"和"岗位"模块后，用户管理页面中不应出现任何部门和岗位相关的筛选条件、表格列或表单字段。
3. **后端松耦合**：后端服务间的依赖应通过接口或可选引用实现，避免硬依赖。当被依赖的模块被禁用时，相关字段返回零值或忽略即可，不应抛出错误。
4. **数据完整性**：模块禁用仅影响功能和展示层，不应删除或破坏已有数据。重新启用模块后，历史数据应能正常恢复使用。

# 接口设计规范

所有前后端`API`必须严格遵循`RESTful`设计规范，`HTTP`方法与操作语义必须一一对应：

| HTTP 方法 | 语义 | 适用场景 |
|-----------|------|---------|
| **GET** | 读取（无副作用） | 列表查询、详情获取、树形数据、导出、下拉选项等所有只读操作 |
| **POST** | 创建资源/执行动作 | 新增记录、文件上传、导入、登录、登出等 |
| **PUT** | 更新资源 | 修改记录、状态变更、重置密码等 |
| **DELETE** | 删除资源 | 单条或批量删除 |

**强制规则**：

1. **查询操作禁止使用POST**：所有查询、列表、搜索、导出、获取详情等读操作必须使用`GET`方法，查询参数通过`URL Query String`传递
2. **创建操作禁止使用GET**：任何会产生副作用（新增数据、上传文件等）的操作禁止使用`GET`方法，必须使用`POST`方法
3. **删除操作必须使用DELETE**：不允许用`POST`或`GET`方法执行删除
4. **更新操作使用PUT**：修改已有资源必须使用`PUT`方法，不允许用`POST`方法
5. **URL 设计使用名词复数或资源名**：如 `/user`、`/dept`、`/dict/type`，避免在 URL 中使用动词（如 `/getUser`、`/deleteUser`）
6. **子资源使用嵌套路径**：如 `/dept/{id}/users`、`/user/{id}/status`

# 代码开发规范

## 后端代码规范

### Go代码开发规范
- 必须使用`goframe-v2`技能
- 不能修改通过脚手架工具维护的代码文件
- 所有的公开方法、变量、结构体属性必须要有注释介绍
- `DAO/DO/Entity`源码文件由`gf gen dao`自动生成，不要手动创建或修改
- `Controller`源码文件由`gf gen ctrl`自动生成骨架，在生成的文件中填写业务逻辑
- **优先使用GoFrame框架提供的组件和方法**：所有`Go`方法调用优先使用`GoFrame`框架已有的方法，避免重复造轮子。例如：
  - 错误处理：使用`GoFrame`的 `gerror` 包进行结构化错误处理
  - 日志记录：使用 `g.Log()` 并传入上下文进行日志记录
  - 配置访问：使用 `g.Cfg()` 获取配置项
  - 数据校验：使用 `GoFrame` 的校验标签和`gvalid`包
  - 遍历目录：使用 `gfile.ScanDirFile`，而非自行实现目录遍历逻辑

### Go代码生成流程
- **API变更**: 修改 `api/{resource}/v1/*.go` → `make ctrl`
- **数据库变更**: 新增或修改 `manifest/sql/{序号}-{迭代名称}.sql`（如 `008-user-auth.sql`）→ `make init`将`sql`文件更新到数据库中 → `make dao`生成或更新`Go`源码文件

### SQL文件管理规范
- **SQL文件命名规范**：数据库变更`SQL`文件采用`{序号}-{当前迭代名称}.sql`格式命名，存放在 `manifest/sql/` 目录下。其中序号为三位数字（如`001`、`002`），服务升级时按序号顺序执行即可完成数据库迁移。当前迭代若不涉及数据库变更，则不用生成该迭代的`sql`文件。
- **SQL文件版本管理**：每次迭代应新建`SQL`文件来维护数据库变更，而非修改旧迭代创建的`SQL`文件。例如：`001-project-init.sql`为旧版本迭代文件，当前迭代应新建如`008-user-auth.sql`而非修改`001-project-init.sql`。仅在用户明确要求时才允许修改旧迭代`SQL`文件。
- **同迭代单文件原则**：宿主 `manifest/sql/` 目录下，同一个业务迭代只保留 **1 个** 版本`SQL`文件，不允许在同一迭代中拆分出多个编号不同但语义同属一次迭代的宿主`SQL`文件。若该迭代后续继续发生数据库变更，应继续追加或整理到当前迭代对应的同一个`SQL`文件中，而不是再新增第二个同迭代`SQL`文件。
- **SQL数据分类管理**：迭代`SQL`文件（如 `002-dict-dept-post.sql`）中只允许包含`DDL`（建表/改表）和 `Seed DML`（系统运行所必需的初始化数据，如字典类型、管理员账号等）。演示/测试用的`Mock`数据（如测试用户、演示部门/岗位等）必须放到 `manifest/sql/mock-data/` 目录下的独立`SQL`文件中，文件名以数字前缀控制执行顺序（如 `01_mock_depts.sql`、`02_mock_posts.sql`）。

### 接口层实现要求

接口层代码（`api/`）必须遵循以下模式：

- **接口文件拆解**：在功能模型中，不要将该功能模块的所有的接口都定义到一个`Go`文件中，而应当按照把不同的接口用途拆解到不同的`Go`文件中。例如：用户管理模块中，用户列表查询接口、用户详情接口、用户创建接口等都应该拆解到不同的`Go`文件中，这样可以避免单个`Go`文件过大，导致可读性和维护性变差。
- **接口文档标签规范**：所有`API`定义的结构体必须包含完善的文档标签，确保自动生成的`OpenAPI/Swagger`文档内容清晰、准确、可用。具体要求如下：
  - **输入参数标签统一使用`json`**：所有输入 DTO（请求结构体）的字段统一使用 `json:"fieldName"` 声明参数名，包括路径参数、Query 参数和请求体字段，禁止在输入结构体中混用 `p` 与 `json` 标签
  - **输出参数继续使用`json`**：所有输出 DTO（响应结构体）的字段继续使用 `json:"fieldName"` 标签，保持响应序列化和文档一致性
  - **`g.Meta`必须包含`dc`标签**：描述该接口的完整功能和使用场景，不是简单重复`summary`，而是补充说明业务逻辑、约束条件、使用场景等。例如：`dc:"分页查询用户列表，支持按用户名、手机号、状态等条件筛选，返回用户基本信息及关联的部门和岗位名称"`
  - **所有输入输出字段必须包含`dc`标签**：对字段含义进行清晰描述，包括取值说明、业务规则、关联关系等。例如：`dc:"部门ID，0表示查询所有用户"` 而非简单的 `dc:"部门ID"`
  - **所有输入输出字段必须包含`eg`标签**：提供真实可用的示例值，方便接口调试和理解。例如：`eg:"admin"`、`eg:"1"`、`eg:"2025-01-01"`
  - **枚举值在`dc`中说明**：状态、类型等枚举字段必须在`dc`中列出所有可选值及含义。例如：`dc:"状态：1=正常 0=停用"`、`dc:"公告类型：1=通知 2=公告"`
  - **可选参数说明默认行为**：筛选条件等可选字段应说明不传时的默认行为。例如：`dc:"按状态筛选：1=正常 0=停用，不传则查询全部"`

### 服务层实现要求

服务层代码（`internal/service/`）必须遵循以下模式：

- **结构化封装**：使用`Service`作为服务实现的默认结构体名称，当服务层逻辑较复杂时应当解耦拆分为多个结构体来封装业务逻辑
- **文件命名规范**：`service/`目录下每个组件（子目录）的源文件必须以组件名作为前缀，使用下划线`_`分割子模块。例如：`config`组件下的文件应命名为`config.go`（主文件）、`config_session.go`、`config_jwt.go`、`config_upload.go`等；`file`组件下应命名为`file.go`、`file_storage.go`、`file_storage_local.go`等。禁止使用无前缀的子模块文件名（如直接命名为`session.go`、`storage.go`）
- **子模块拆分**：同一`service`组件下不同子模块的业务逻辑必须拆分到独立的`Go`文件中实现，不要将所有逻辑都写在单个文件中。例如：`config`组件应按配置分组拆分为`config_jwt.go`、`config_session.go`、`config_upload.go`等，每个文件只负责一个子模块的配置读取逻辑
- **定时任务管理**：所有定时任务（`cron job`）必须在`service/cron`独立组件中统一管理，禁止在`cmd/`或其他`service`组件中直接编写定时任务逻辑。`cron`组件提供统一的`Start(ctx)`入口方法，由`cmd`层一次性调用启动所有定时任务。每个定时任务的具体实现拆分到独立文件中（如`cron_session.go`、`cron_servermon.go`），使用`GoFrame`的`gcron`组件注册定时任务
- **定时任务解耦规范**：定时任务的具体业务逻辑必须在对应业务模块中实现，`cron`模块只负责任务注册和调度。例如：监控数据清理逻辑封装在`servermon.CleanupStale()`方法中，在线会话清理逻辑封装在`session.CleanupInactive()`方法中，`cron`模块只负责调用这些业务方法。禁止在`cron`模块中直接操作数据库或编写业务逻辑
- **上下文管理**：第一个参数始终传入 `ctx context.Context`
- **数据库操作**：
  - **数据交互**：与数据库交互时，必须使用`DO`对象，不使用 `g.Map`来传递`Data`参数
  - **事务管理**：使用 `dao.Xxx.Transaction()`闭包处理多步操作，该方法支持嵌套事务，其中`Xxx`为对应的`Dao`对象名称
  - **跨数据库兼容**：所有数据库操作必须使用跨数据库类型的通用语法，禁止使用特定数据库的内置函数（如`MySQL`的 `FIND_IN_SET`、`GROUP_CONCAT`、`IF()`，`PostgreSQL`的 `ANY(ARRAY[...])`等）。例如对于层级数据（如部门树）的递归查询，应通过应用层迭代查询实现：先通过 `parent_id` 逐层查询收集所有子级`ID`，再使用 `WHERE IN` 进行批量查询，而非依赖数据库特有的递归语法

### 控制器层实现要求

控制器层代码（`internal/controller/`）必须遵循以下模式：

- **依赖注入**：所有控制器依赖的`service`必须在控制器结构体中定义为字段，在`NewV1()`构造函数中初始化。**禁止在方法内部临时调用`service.New()`创建实例**。

  ```go
  // 错误：在方法内部临时创建 service 实例
  func (c *ControllerV1) GetInfo(ctx context.Context, req *v1.GetInfoReq) (res *v1.GetInfoRes, err error) {
      roleSvc := role.New()  // 错误！
      menuSvc := menu.New()  // 错误！
      // ...
  }

  // 正确：service 作为控制器字段，在构造函数中初始化
  type ControllerV1 struct {
      userSvc *usersvc.Service // user service
      roleSvc *role.Service    // role service
      menuSvc *menu.Service    // menu service
  }

  func NewV1() userapi.IUserV1 {
      return &ControllerV1{
          userSvc: usersvc.New(),
          roleSvc: role.New(),
          menuSvc: menu.New(),
      }
  }

  func (c *ControllerV1) GetInfo(ctx context.Context, req *v1.GetInfoReq) (res *v1.GetInfoRes, err error) {
      roleNames, err := c.roleSvc.GetUserRoleNames(ctx, user.Id)  // 正确！
      // ...
  }
  ```
- **控制器文件结构**：每个控制器的`_new.go`文件定义控制器结构体和构造函数，其他文件实现具体的接口方法

### 软删除与时间维护规范

`GoFrame`框架提供了自动化的软删除和时间维护特性，**必须正确理解并使用**，避免编写冗余代码。

#### 自动时间维护

当数据表包含 `created_at`、`updated_at`、`deleted_at` 字段时，`GoFrame`会自动处理：

| 字段 | 自动行为 |
|------|---------|
| `created_at` | `Insert/InsertAndGetId` 时自动写入，后续更新/删除不会改变 |
| `updated_at` | `Insert/Update/Save` 时自动写入/更新 |
| `deleted_at` | `Delete` 时自动写入（软删除）或查询时自动过滤 |

**强制规则**：

1. **禁止手动设置时间字段**：
   ```go
   // 错误：手动设置 created_at 和 updated_at
   dao.User.Ctx(ctx).Data(do.User{
       Name:      "john",
       CreatedAt: gtime.Now(),  // 冗余！框架会自动处理
       UpdatedAt: gtime.Now(),  // 冗余！框架会自动处理
   }).Insert()

   // 正确：让框架自动处理
   dao.User.Ctx(ctx).Data(do.User{
       Name: "john",
   }).Insert()
   ```

2. **禁止手动添加 `WhereNull(cols.DeletedAt)` 条件**：
   ```go
   // 错误：手动添加软删除条件
   dao.User.Ctx(ctx).
       Where(do.User{Status: 1}).
       WhereNull(cols.DeletedAt).  // 冗余！框架会自动添加
       Scan(&list)

   // 正确：框架自动添加 deleted_at IS NULL
   dao.User.Ctx(ctx).
       Where(do.User{Status: 1}).
       Scan(&list)
   ```

#### 软删除操作

当表存在 `deleted_at` 字段时，`Delete()` 方法会自动转为软删除（`UPDATE SET deleted_at = NOW()`）：

```go
// 正确：使用 Delete() 方法，框架自动处理软删除
dao.User.Ctx(ctx).Where(do.User{Id: id}).Delete()
// 实际执行: UPDATE `sys_user` SET `deleted_at`=NOW() WHERE `id`=?

// 错误：手动 Update 设置 deleted_at
dao.User.Ctx(ctx).
    Where(do.User{Id: id}).
    Data(do.User{DeletedAt: gtime.Now()}).  // 冗余！
    Update()
```

#### 硬删除场景

某些业务场景需要硬删除（如字典类型），此时需要确保表中没有 `deleted_at` 字段，或者：

```go
// 字典类型使用硬删除（表中没有 deleted_at 字段）
dao.SysDictType.Ctx(ctx).Where(do.SysDictType{Id: id}).Delete()
// 实际执行: DELETE FROM `sys_dict_type` WHERE `id`=?
```

## 前端代码规范

- 路径别名 `#/*` 指向 `./src/*`
- 路由模块放 `src/router/routes/modules/`
- 视图文件放 `src/views/` 对应目录
- API 文件放 `src/api/` 对应目录
- 适配器层 `src/adapter/`：`component`（组件映射）、`form`（表单配置）、`vxe-table`（表格配置）
- 全局组件在 `src/components/global/` 注册（如`GhostButton`用于表格操作列）
- 表格页面使用 `useVbenVxeGrid` + `Page` 组件，操作列用 `ghost-button` + `Popconfirm`
- 前端样式和交互参考`ruoyi-plus-vben5`项目保持一致

## E2E测试规范

- 测试用例必须要完整覆盖业务模块的各项操作（如增删改查等操作），保证功能的完整性和可用性
- 所有的用例需要在`tasks.md`中有工作记录，并且使用`openspec-e2e`技能生成和管理对应的测试用例
- 修复`bug`或新增功能涉及**用户可观察行为变化**时，必须编写或更新对应的`E2E`测试用例
- 修改完成后必须运行相关`E2E`测试并确认通过，再标记任务完成
- 纯内部重构（无`UI`变化）可豁免，但需运行已有测试套件确认无回归
- 使用测试工具（如`Playwright`）在涉及创建文件的场景（如截图），应该将创建的文件放置到项目根目录的`temp/`目录下

## UI设计规范

重要：所有前端`UI`设计和实现必须参考`ruoyi-plus-vben5`项目，保持`UI`的一致性和用户体验的一致性。

在实现任何前端页面或组件时，必须遵循以下规范：

1. **交互设计**: 弹窗（`Modal/Drawer`）、表单、表格、搜索栏等交互模式必须与参考项目保持一致
2. **页面样式**: 布局、间距、字体、颜色等视觉元素参考参考项目的实现
3. **组件使用**: 优先使用与参考项目相同的组件和配置方式，包括：
   - 表单使用 `useVbenForm`，弹窗使用 `useVbenModal`，抽屉使用 `useVbenDrawer`
   - `RadioGroup`单选项使用 `optionType: 'button'` + `buttonStyle: 'solid'`（按钮样式）
   - 文件上传使用 `Upload.Dragger`（拖拽上传样式）
   - 文件下载使用 `requestClient.download` 方法
   - 操作列的"更多"下拉菜单使用 `Dropdown` + `Menu` + `MenuItem`
4. **弹窗规范**: 导入弹窗包含拖拽上传区域、文件类型提示、下载模板链接、覆盖开关；重置密码弹窗包含用户信息展示（`Descriptions`）和密码输入
5. **图标使用**: 使用 `IconifyIcon` 组件（来自 `@vben/icons`），图标名使用`Iconify`格式（如 `ant-design:inbox-outlined`）

开发新页面前，**必须先查看参考项目中对应页面的实现**，确保`UI`和交互保持一致。

---
> Source: [gqcn/openspec-practice](https://github.com/gqcn/openspec-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
