## prjflow

> 这是一个基于 Go + Gin + GORM 和 Vue3 + TypeScript + Ant Design Vue 的全栈项目管理软件，采用 TDD 开发方式。

# 项目管理系统 - Cursor 开发规则

## 项目概述

这是一个基于 Go + Gin + GORM 和 Vue3 + TypeScript + Ant Design Vue 的全栈项目管理软件，采用 TDD 开发方式。

## 技术栈

### 后端
- **语言**: Go 1.24+
- **框架**: Gin (v1.11.0)
- **ORM**: GORM (v1.30.5)
- **数据库**: SQLite (默认，支持 MySQL)
- **测试**: Go test + testify
- **认证**: JWT + 微信开放平台扫码登录

### 前端
- **框架**: Vue 3 (Composition API)
- **语言**: TypeScript
- **UI库**: Ant Design Vue (v4.2.6)
- **构建工具**: Vite
- **状态管理**: Pinia
- **路由**: Vue Router
- **HTTP客户端**: Axios
- **日期处理**: dayjs
- **图表**: ECharts + vue-echarts
- **Markdown**: marked + highlight.js

## 项目结构

### 后端结构
```
backend/
├── cmd/server/          # 应用入口
├── internal/
│   ├── api/            # API路由层（Handler模式）
│   ├── model/          # 数据模型（GORM）
│   ├── utils/          # 工具函数（response, pagination, database等）
│   ├── middleware/     # 中间件（auth, cors, logger, permission等）
│   ├── config/         # 配置管理
│   └── websocket/      # WebSocket支持
├── pkg/                # 可复用包（auth, wechat, permission等）
├── tests/unit/         # 单元测试
└── migrations/         # 数据库迁移
```

### 前端结构
```
frontend/src/
├── api/                # API接口定义
├── views/              # 页面组件
├── components/         # 公共组件
├── stores/             # Pinia状态管理
├── router/             # 路由配置
└── utils/              # 工具函数（request, date等）
```

## 编码规范

### Go 后端规范

#### Handler 模式
- 每个 Handler 结构体包含 `db *gorm.DB` 字段
- 使用构造函数模式：`NewXxxHandler(db *gorm.DB) *XxxHandler`
- Handler 方法接收 `*gin.Context` 作为参数
- 私有辅助方法使用小写开头，不接收 `*gin.Context`

```go
type ProjectHandler struct {
    db *gorm.DB
}

func NewProjectHandler(db *gorm.DB) *ProjectHandler {
    return &ProjectHandler{db: db}
}

func (h *ProjectHandler) GetProjects(c *gin.Context) {
    // 实现逻辑
}

func (h *ProjectHandler) getProjectStatistics(projectID uint) gin.H {
    // 私有辅助方法
}
```

#### API 响应格式
- 统一使用 `utils.Success()` 和 `utils.Error()` 返回响应
- 成功响应：HTTP 200，body 包含 `{code: 200, message: "success", data: {...}}`
- 错误响应：HTTP 200，body 包含 `{code: 错误码, message: "错误信息"}`
- 错误码：200=成功，404=资源不存在，500=服务器错误

```go
// 成功响应
utils.Success(c, gin.H{
    "list":  projects,
    "total": total,
})

// 错误响应
utils.Error(c, 404, "项目不存在")
```

#### 数据模型规范
- 使用 GORM 标签定义模型
- 软删除使用 `gorm.DeletedAt`
- 时间字段使用 `time.Time`
- JSON 数组使用自定义 `StringArray` 类型（实现 `driver.Valuer` 和 `sql.Scanner`），仅用于测试类型等场景
- 标签使用独立表（Tag）和关联表（project_tags）实现多对多关系
- 关联关系使用 GORM 的 `Preload` 和 `Joins` 避免 N+1 问题

```go
type Project struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    Name        string         `gorm:"not null" json:"name"`
    Tags        []Tag          `gorm:"many2many:project_tags;" json:"tags,omitempty"`
    DeletedAt   gorm.DeletedAt `gorm:"index" json:"-"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
}

type Tag struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    Name        string         `gorm:"size:50;not null;uniqueIndex" json:"name"`
    Description string         `gorm:"size:200" json:"description"`
    Color       string         `gorm:"size:20;default:'blue'" json:"color"`
    Projects    []Project      `gorm:"many2many:project_tags;" json:"projects,omitempty"`
}
```

#### 查询和分页
- 使用 `utils.GetPage(c)` 和 `utils.GetPageSize(c)` 获取分页参数
- 搜索使用 `c.Query("keyword")` 和 `c.QueryArray("tags")`（标签ID数组）
- 多标签搜索使用 JOIN 查询：`JOIN project_tags ON ... WHERE tag_id IN ?`
- 标签筛选使用标签ID，不是标签名称
- 分页查询必须返回 `total` 字段

```go
page := utils.GetPage(c)
pageSize := utils.GetPageSize(c)
offset := (page - 1) * pageSize

// 标签筛选（使用JOIN）
if tagIDs := c.QueryArray("tags"); len(tagIDs) > 0 {
    query = query.Joins("JOIN project_tags ON project_tags.project_id = projects.id").
        Where("project_tags.tag_id IN ?", tagIDs).
        Group("projects.id")
}

var total int64
query.Model(&model.Project{}).Count(&total)

query.Preload("Tags").Offset(offset).Limit(pageSize).Order("created_at DESC").Find(&projects)
```

#### 日期处理
- 前端传递日期字符串格式：`"YYYY-MM-DD"` 或 `"YYYY-MM-DD HH:mm:ss"`
- 后端使用 `time.Time` 类型
- 日期解析使用 `time.Parse("2006-01-02", dateStr)` 或 `time.Parse("2006-01-02 15:04:05", dateStr)`
- 返回给前端的日期使用 JSON 序列化（自动转换为 RFC3339 格式）

#### 工时和进度计算
- 任务和 Bug 包含 `EstimatedHours`（预估工时）和 `ActualHours`（实际工时）
- `ActualHours` 自动从 `ResourceAllocation` 计算得出
- `Progress` 自动计算：`ActualHours / EstimatedHours * 100`（范围 0-100%）
- 更新工时时自动创建/更新 `ResourceAllocation` 记录

#### 数据库迁移
- 使用 GORM 的 `AutoMigrate` 进行自动迁移
- 对于 SQLite 的 NOT NULL 字段添加，使用手动迁移逻辑
- 使用 `gorm.io/gorm/migrator.Migrator` 接口进行细粒度控制
- 避免使用数据库特定的 SQL 语法，保持 SQLite 和 MySQL 兼容

### TypeScript/Vue 前端规范

#### 组件结构
- 使用 `<script setup>` 语法（Composition API）
- 使用 TypeScript 类型定义
- 组件文件使用 PascalCase 命名

```vue
<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import type { Project } from '@/api/project'

const projects = ref<Project[]>([])
const loading = ref(false)

const loadProjects = async () => {
  // 实现逻辑
}
</script>
```

#### API 调用
- API 函数定义在 `src/api/` 目录下
- 使用统一的 `request` 实例（已配置拦截器）
- 响应数据自动提取 `data` 字段
- 错误自动显示 message 提示

```typescript
import request from '@/utils/request'

export interface Tag {
  id: number
  name: string
  description?: string
  color?: string
}

export interface Project {
  id: number
  name: string
  tags?: Tag[]  // 标签对象数组
}

export const getTags = async (): Promise<Tag[]> => {
  return request.get('/tags')
}

export const getProjects = async (params?: {
  keyword?: string
  tags?: number[]  // 标签ID数组
  page?: number
  size?: number
}): Promise<{ list: Project[]; total: number }> => {
  return request.get('/projects', { params })
}
```

#### 日期格式化
- 统一使用 `dayjs` 进行日期处理
- 日期时间显示格式：`"YYYY-MM-DD HH:mm:ss"`
- 日期显示格式：`"YYYY-MM-DD"`
- 使用 `src/utils/date.ts` 中的格式化函数

```typescript
import dayjs from 'dayjs'

// 格式化日期时间
const formatted = dayjs(date).format('YYYY-MM-DD HH:mm:ss')

// 格式化日期
const dateOnly = dayjs(date).format('YYYY-MM-DD')
```

#### 表单处理
- 使用 Ant Design Vue 的表单组件
- 标签选择使用 `a-select` 的 `mode="multiple"`，从API加载所有标签
- 标签字段使用 `tag_ids`（标签ID数组），不是 `tags`（标签名称数组）
- 表单验证使用 Ant Design Vue 的验证规则
- 日期选择器使用 `a-date-picker`

```vue
<a-form-item label="标签" name="tag_ids">
  <a-select
    v-model:value="form.tag_ids"
    mode="multiple"
    placeholder="选择标签（支持多选）"
    allow-clear
    :options="tagOptions"
    :field-names="{ label: 'name', value: 'id' }"
  />
</a-form-item>
```

#### 数组参数序列化
- GET 请求的数组参数需要配置 `paramsSerializer: { indexes: null }`
- 确保数组参数序列化为 `tags=value1&tags=value2` 格式

#### 状态管理
- 使用 Pinia 进行状态管理
- Store 文件放在 `src/stores/` 目录
- 认证状态使用 `useAuthStore()`
- 权限状态使用 `usePermissionStore()`

## 测试规范

### 单元测试
- 测试文件放在 `backend/tests/unit/` 目录
- 测试文件命名：`*_test.go`
- 使用内存数据库（`:memory:`）进行测试
- 使用 `SetupTestDB(t)` 和 `TeardownTestDB(t, db)` 管理测试数据库
- 使用 `testify/assert` 和 `testify/require` 进行断言
- 测试覆盖率目标：100%

```go
func TestProjectHandler_GetProjects(t *testing.T) {
    db := SetupTestDB(t)
    defer TeardownTestDB(t, db)
    
    handler := api.NewProjectHandler(db)
    
    t.Run("获取所有项目", func(t *testing.T) {
        gin.SetMode(gin.TestMode)
        w := httptest.NewRecorder()
        c, _ := gin.CreateTestContext(w)
        c.Request = httptest.NewRequest(http.MethodGet, "/api/projects", nil)
        
        handler.GetProjects(c)
        
        assert.Equal(t, http.StatusOK, w.Code)
        // 验证响应
    })
}
```

### TDD 开发流程
1. **编写详细设计文档**：包含数据模型、API设计、业务逻辑、测试用例
2. **编写单元测试**：先写完整的单元测试（覆盖率目标100%）
3. **运行测试**：确保测试失败（Red阶段）
4. **实现功能**：编写最小代码使测试通过（Green阶段）
5. **重构优化**：在测试通过的基础上重构代码（Refactor阶段）
6. **测试覆盖率检查**：使用 `go test -cover` 确保覆盖率100%

## 特殊约定

### 标签管理
- 标签使用独立表（Tag）管理，不是JSON数组
- 标签与项目通过关联表（project_tags）实现多对多关系
- 标签API：`GET /api/tags` 获取所有标签，支持CRUD操作
- 标签搜索：使用标签ID进行筛选，`?tags=1&tags=2`（标签ID数组）
- 前端标签选择：从API加载所有标签，使用 `a-select` 的 `mode="multiple"`
- 项目创建/更新：使用 `tag_ids` 字段（标签ID数组），不是 `tags`（标签名称数组）
- 标签显示：显示标签名称和颜色，标签对象包含 `{id, name, color}` 字段

### 工时管理
- 任务和 Bug 支持预估工时和实际工时
- 实际工时从资源分配记录自动计算
- 更新工时时自动创建资源分配记录
- 进度自动计算：实际工时/预估工时 * 100%

### Markdown 支持
- 需求、Bug、任务、计划、版本等模块支持 Markdown 格式
- 使用 `MarkdownEditor.vue` 组件进行编辑
- 使用 `marked` 和 `highlight.js` 进行渲染

### 软删除
- 使用 GORM 的 `DeletedAt` 实现软删除
- 查询时自动过滤已删除记录
- 恢复删除使用 `db.Unscoped().Model(&user).Update("deleted_at", nil)`

### 并发控制
- 用户创建时处理唯一约束冲突（username, wechat_open_id）
- 使用重试机制处理并发创建冲突

### 微信登录
- 使用微信开放平台扫码登录
- 支持初始化配置和用户添加两种回调场景
- 使用 WebSocket 推送扫码通知

## 数据库兼容性

- 使用 GORM 抽象层，支持 SQLite 和 MySQL
- 避免使用数据库特定的 SQL 语法
- 日期时间使用 `time.Time` 类型，由 GORM 自动处理
- JSON 数组使用 `StringArray` 类型存储为 JSON 文本（仅用于测试类型等场景）
- 标签使用独立表和关联表，不使用JSON数组
- 迁移时注意 SQLite 的 NOT NULL 字段添加限制
- 数据迁移：从JSON字段迁移到关联表时，需要提取数据并创建关联关系

## 错误处理

### 后端
- 使用 `utils.Error()` 返回统一格式的错误响应
- 错误信息要清晰明确，便于前端显示
- 数据库错误要转换为用户友好的错误信息

### 前端
- 使用 Axios 拦截器统一处理错误
- 401 错误自动跳转登录页
- 其他错误显示 message 提示
- 网络错误显示友好提示

## 代码提交规范

- 提交信息使用中文
- 格式：`功能描述：具体修改内容`
- 示例：`修复项目管理页标签搜索：支持多选标签搜索`
- 每次提交前运行测试确保通过

## 注意事项

1. **TDD开发**：严格遵循TDD流程，先写测试再实现功能
2. **数据库兼容**：使用GORM抽象层，保持SQLite和MySQL兼容
3. **日期格式**：前端统一使用 dayjs 格式化，后端使用 time.Time
4. **响应格式**：统一使用 utils.Success 和 utils.Error
5. **测试覆盖率**：目标100%，使用 `go test -cover` 检查
6. **代码规范**：遵循Go和TypeScript最佳实践
7. **API设计**：RESTful风格，统一响应格式
8. **前端交互**：使用Ant Design Vue组件，保持UI一致性

## 开发命令

### 后端
```bash
# 启动开发服务器
cd backend && go run cmd/server/main.go

# 运行测试
go test ./tests/unit

# 检查测试覆盖率
go test ./tests/unit -cover
```

### 前端
```bash
# 安装依赖
cd frontend && npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build
```

## 文件命名规范

- Go文件：`snake_case.go`（如 `project_handler.go`）
- 测试文件：`*_test.go`（如 `project_test.go`）
- Vue组件：`PascalCase.vue`（如 `Project.vue`）
- TypeScript文件：`camelCase.ts`（如 `project.ts`）
- API文件：`camelCase.ts`（如 `project.ts`）

## 注释规范

- Go代码：使用中文注释，函数注释说明功能、参数、返回值
- TypeScript代码：使用中文注释，复杂逻辑需要注释说明
- API接口：在Handler方法上添加注释说明接口功能

---
> Source: [funnywwh/prjflow](https://github.com/funnywwh/prjflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
