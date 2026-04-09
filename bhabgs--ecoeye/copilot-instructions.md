## ecoeye

> 河道守护与生态监测平台，基于 NovaAdmin 基座开发。

# EcoEye - 生态之眼

河道守护与生态监测平台，基于 NovaAdmin 基座开发。

## 项目概述

- **产品名称**: 生态之眼 EcoEye
- **定位**: 面向公众、钓鱼爱好者及社区环保志愿者的生态监测与河道守护平台
- **技术栈**: pnpm monorepo + NestJS 微服务 + React 18 + Redux Toolkit + shadcn/ui

## 快速开始

```bash
# 首次初始化（一键完成所有配置）
./scripts/init.sh

# 或手动分步执行：
pnpm docker:up     # 启动 PostgreSQL + Redis
pnpm install       # 安装依赖
./scripts/db-sync.sh  # 同步数据库

# 启动开发服务
pnpm dev           # 启动所有服务
pnpm dev:web       # 仅启动前端 (5173)
pnpm dev:gateway   # 仅启动网关 (3000)
```

## 项目结构

```
ecoeye/
├── apps/
│   ├── web/                 # React 前端 (端口 5173)
│   ├── gateway/             # API 网关 (端口 3000)
│   ├── auth-service/        # 认证服务 (端口 3001)
│   ├── rbac-service/        # 权限管理 (端口 3002)
│   ├── system-service/      # 系统配置 (端口 3003)
│   ├── ecology-service/     # 生态监测 (端口 3004)
│   ├── report-service/      # 举报管理 (端口 3005)
│   ├── task-service/        # 生态任务 (端口 3006)
│   └── community-service/   # 社区互动 (端口 3007)
├── libs/
│   └── shared/              # 共享库 (@ecoeye/shared)
├── scripts/                 # Shell 脚本
│   ├── init.sh              # 项目初始化
│   ├── db-sync.sh           # 数据库增量同步
│   ├── db-export.sh         # 数据库导出
│   └── deploy.sh            # 部署脚本
├── database/
│   └── seeds/               # 数据库种子文件
└── docker/                  # Docker 配置
```

## 微服务架构

| 服务 | 端口 | 职责 |
|------|------|------|
| gateway | 3000 | API 网关，路由代理，Swagger 聚合 |
| auth-service | 3001 | JWT 认证，登录注册 |
| rbac-service | 3002 | 用户、角色、部门、菜单管理 |
| system-service | 3003 | 系统配置、字典管理、国际化 |
| ecology-service | 3004 | 河段、物种、生态记录 |
| report-service | 3005 | 举报管理、处理流程 |
| task-service | 3006 | 生态任务、参与记录 |
| community-service | 3007 | 动态、评论、守护小队 |

## 前端架构 (apps/web)

### 目录结构

```
src/
├── components/
│   ├── ui/              # shadcn/ui 基础组件
│   ├── crud-page.tsx    # 通用 CRUD 页面组件
│   ├── crud-dialog.tsx  # 通用表单弹窗组件
│   ├── form-dialog.tsx  # 基础弹窗组件
│   ├── dict-select.tsx  # 字典下拉组件
│   └── data-table.tsx   # 数据表格组件
├── store/
│   ├── index.ts         # Redux store 配置
│   └── slices/          # Redux slices
├── hooks/
│   ├── redux.ts         # Redux hooks
│   └── useDict.ts       # 字典 hook
├── pages/               # 页面组件
├── layouts/             # 布局组件
└── utils/               # 工具函数
```

### 核心组件

#### CrudPage - 通用 CRUD 页面
```tsx
<CrudPage
  title="举报管理"
  columns={columns}
  formFields={formFields}
  data={list}
  loading={loading}
  pagination={pagination}
  onAdd={handleAdd}
  onEdit={handleEdit}
  onDelete={handleDelete}
  onRefresh={handleRefresh}
  columns2={2}  // 表单两列布局
/>
```

#### DictSelect - 字典下拉组件
```tsx
import { DictSelect } from '@/components/dict-select';
import { DICT_CODES } from '@/hooks/useDict';

<DictSelect
  dictCode={DICT_CODES.REPORT_TYPE}
  value={value}
  onChange={onChange}
  placeholder="请选择举报类型"
/>
```

### 字典系统

字典类型定义在 `hooks/useDict.ts`:
```typescript
export const DICT_CODES = {
  REPORT_TYPE: 'report_type',
  REPORT_STATUS: 'report_status',
  RIVER_RISK_LEVEL: 'river_risk_level',
  SPECIES_PROTECTION_LEVEL: 'species_protection_level',
  SPECIES_HABITAT_TYPE: 'species_habitat_type',
  TASK_TYPE: 'task_type',
  TASK_STATUS: 'task_status',
  TEAM_STATUS: 'team_status',
  TEAM_JOIN_POLICY: 'team_join_policy',
  ECOLOGY_RECORD_TYPE: 'ecology_record_type',
  ECOLOGY_RECORD_STATUS: 'ecology_record_status',
  POST_TYPE: 'post_type',
} as const;
```

## 后端架构 (NestJS)

### 服务模板结构

```
service/src/
├── main.ts              # 入口文件
├── app.module.ts        # 根模块
└── module-name/
    ├── entity.ts        # TypeORM 实体
    ├── dto/             # DTO 定义
    ├── service.ts       # 业务逻辑
    ├── controller.ts    # API 控制器
    └── module.ts        # 模块定义
```

### Gateway 路由配置

路由映射在 `apps/gateway/src/proxy/proxy.controller.ts`:
```typescript
const SERVICE_MAP = {
  auth: 'http://localhost:3001',
  rbac: 'http://localhost:3002',
  system: 'http://localhost:3003',
  ecology: 'http://localhost:3004',
  report: 'http://localhost:3005',
  task: 'http://localhost:3006',
  community: 'http://localhost:3007',
};
```

## 数据库

- **PostgreSQL**: 主数据库
- **Redis**: 缓存、会话

### 数据库管理脚本

```bash
# 增量同步（推荐，不清除数据）
./scripts/db-sync.sh           # 同步所有数据
./scripts/db-sync.sh --dict    # 仅同步字典
./scripts/db-sync.sh --menus   # 仅同步菜单

# 导出当前数据
./scripts/db-export.sh

# 进入 PostgreSQL 命令行
pnpm psql
```

### 增量更新策略

修改 seed 文件后，**不要全量重置**，使用增量同步：

```bash
# 创建临时脚本 database/seeds/_sync-temp.sql
docker exec -i ecoeye-postgres psql -U postgres -d ecoeye < database/seeds/_sync-temp.sql
rm database/seeds/_sync-temp.sql
```

SQL 使用 `ON CONFLICT DO UPDATE` 确保幂等性。

### 字典数据

字典数据在 `database/seeds/init-dict-data.sql`

## 开发规范

### 新增 CRUD 页面

1. **创建 Redux Slice** (`store/slices/xxxSlice.ts`)
2. **注册到 Store** (`store/index.ts`)
3. **创建页面组件** (`pages/xxx/index.tsx`)
4. **添加菜单** (数据库 sys_menu 表)

### 新增微服务

1. 在 `apps/` 下创建服务目录
2. 更新 `package.json` 添加启动脚本
3. 更新 Gateway 路由映射
4. 更新 Docker Compose 配置

### 代码风格

- 使用 TypeScript 严格模式
- React 函数组件 + Hooks
- Redux Toolkit 管理状态
- shadcn/ui 组件库
- 字典值统一使用 DictSelect 组件

## 常用命令

```bash
# === 开发 ===
pnpm dev              # 启动所有服务
pnpm dev:web          # 仅启动前端
pnpm dev:gateway      # 仅启动网关

# === 构建 ===
pnpm build            # 构建所有
pnpm build:web        # 构建前端

# === Docker (开发环境) ===
pnpm docker:up        # 启动 PostgreSQL + Redis
pnpm docker:down      # 停止容器
pnpm docker:logs      # 查看日志
pnpm psql             # 进入 PostgreSQL 命令行

# === 数据库 ===
./scripts/db-sync.sh  # 增量同步数据
./scripts/db-export.sh # 导出数据

# === 初始化 & 部署 ===
./scripts/init.sh     # 首次初始化项目
./scripts/deploy.sh build   # 构建镜像
./scripts/deploy.sh up      # 启动生产服务
./scripts/deploy.sh down    # 停止服务
./scripts/deploy.sh logs    # 查看日志

# === API 类型生成 ===
pnpm generate-api     # 根据 Swagger 生成前端类型
```

## 环境变量

### 前端 (.env.development)
```
VITE_API_BASE_URL=http://localhost:3000/api
```

### 后端
各服务读取根目录 `.env.development`:
```
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=ecoeye
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres
REDIS_HOST=localhost
REDIS_PORT=6379
JWT_SECRET=your-secret-key
```

## 文档

- 产品需求: `doc/生态之眼 EcoEye — 产品需求文档.md`
- 技术方案: `doc/生态之眼 EcoEye — 技术方案.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bhabgs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bhabgs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
