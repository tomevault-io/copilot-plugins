## xiuer

> You are an expert full-stack developer proficient in **Python 3.12**, **FastAPI**, **Next.js 14**, **TypeScript**, **PostgreSQL**, **Docker**, and modern web development best practices.

# 小红书数据分析平台 - Cursor Rules

You are an expert full-stack developer proficient in **Python 3.12**, **FastAPI**, **Next.js 14**, **TypeScript**, **PostgreSQL**, **Docker**, and modern web development best practices.

每次你选择引用某条规则，请在输出中显式写出该规则（可使用缩写）。

## 🎯 Core Expertise
- Backend: FastAPI + AsyncSQLAlchemy + Pydantic v2 + Alembic
- Frontend: Next.js 14 (App Router) + TypeScript + Tailwind CSS + React Query + Zustand
- Database: PostgreSQL + Redis (caching)
- DevOps: Docker + GitHub Actions + MkDocs
- Crawler: Python + PyExecJS + Loguru
- Testing: pytest + Jest + React Testing Library

## 📁 Monorepo Structure

```
project_root/
├── fastapi_backend/         # FastAPI 后端服务
│   ├── app/
│   │   ├── core/           # 核心配置 (config, database, logger)
│   │   ├── models.py       # SQLAlchemy ORM 模型
│   │   ├── schemas/        # Pydantic 数据模型
│   │   ├── services/       # 业务逻辑层
│   │   ├── routes/         # API 路由层
│   │   └── main.py         # FastAPI 应用入口
│   ├── tests/              # 后端测试
│   ├── alembic/            # 数据库迁移
│   └── pyproject.toml
├── nextjs-frontend/         # Next.js 前端应用
│   ├── app/                # App Router 页面
│   ├── components/         # UI 组件
│   ├── lib/                # 工具库和 hooks
│   └── __tests__/          # 前端测试
├── Spider_XHS/              # 小红书爬虫系统
│   ├── apis/               # API 接口封装
│   ├── xhs_utils/          # 爬虫工具函数
│   ├── src/                # 核心爬虫代码
│   ├── main.py             # 爬虫入口
│   └── pyproject.toml
├── docs/                   # MkDocs 文档
├── mdc-examples/           # MDC 组件示例
├── .github/workflows/      # GitHub Actions
├── docker-compose.yml      # 开发环境
└── mkdocs.yml             # 文档配置
```

## 🏗️ Architecture Principles

### Backend (FastAPI)
- **分层架构**: Routes → Services → Models
- **依赖注入**: 使用 FastAPI 的 Dependency Injection
- **异步优先**: 所有数据库操作使用 AsyncSession
- **类型安全**: 严格的 Pydantic 数据验证
- **错误处理**: 统一的 HTTPException 处理

### Frontend (Next.js)
- **App Router**: 使用 Next.js 14 最新路由系统
- **服务端组件**: 优先使用 RSC，必要时才用客户端组件
- **类型安全**: TypeScript strict mode + Zod 验证
- **状态管理**: Zustand (全局) + React Query (服务端状态)
- **样式系统**: Tailwind CSS + CSS Modules

### Spider (Python)
- **异步爬取**: 使用 asyncio 提升性能
- **错误重试**: 内置重试机制和容错处理
- **数据管道**: 爬取 → 清洗 → 存储 → Webhook 通知
- **反反爬**: 动态 User-Agent + 请求间隔控制

## 🔧 Development Rules

### Naming Conventions
- **文件名**: `snake_case.py` (Python), `kebab-case.ts` (工具), `PascalCase.tsx` (组件)
- **变量/函数**: `snake_case` (Python), `camelCase` (TypeScript)
- **类名**: `PascalCase` (所有语言)
- **常量**: `UPPER_SNAKE_CASE` (所有语言)
- **数据库表**: `snake_case` 复数形式 (如 `xhs_notes`)

### Import Organization
#### Python 导入顺序
1. `from __future__ import annotations`
2. 标准库导入
3. 第三方库导入
4. 本地模块导入

#### TypeScript 导入顺序
1. React 相关导入
2. Next.js 相关导入
3. 第三方库导入
4. 本地组件和工具导入
5. 类型导入 (使用 `import type`)

### Error Handling
#### Backend
- 业务逻辑错误: `HTTPException(status_code=400, detail="message")`
- 权限错误: `HTTPException(status_code=403, detail="message")`
- 资源不存在: `HTTPException(status_code=404, detail="message")`
- 服务器错误: 统一在 middleware 处理

#### Frontend
- 使用 try-catch 包装异步操作
- 用户友好的错误提示
- 错误边界处理组件崩溃
- 网络错误重试机制

## 📝 Documentation Standards

### MDC Components
参考 `mdc-examples/` 目录中的示例：
- **API 文档**: 使用 `::api-endpoint` 组件
- **组件演示**: 使用 `::component-demo` 组件
- **属性表格**: 使用 `::props-table` 组件
- **代码高亮**: 使用 `::code-block` 组件
- **警告提示**: 使用 `::alert` 组件

### API 文档规范
- 每个端点必须包含请求/响应示例
- 参数说明必须完整
- 错误状态码说明
- 认证要求说明

## 🧪 Testing Standards

### Backend Tests
- 使用 pytest-asyncio strict mode
- 单元测试覆盖率 ≥ 80%
- 集成测试覆盖主要业务流程
- Mock 外部依赖和数据库操作

### Frontend Tests
- Jest + React Testing Library
- 组件单元测试
- 用户交互测试
- API 集成测试
- 快照测试用于 UI 一致性

### Spider Tests
- 数据抓取准确性测试
- 错误处理和重试机制测试
- 反反爬机制测试
- 数据清洗和格式化测试

## 🚀 Performance & Security

### Database Optimization
- 使用索引优化查询
- 分页处理大数据集
- Redis 缓存热点数据
- 连接池管理

### Frontend Optimization
- Next.js Image 组件
- 代码分割和懒加载
- React.memo 和 useMemo
- 虚拟滚动大列表

### Security Best Practices
- 输入验证和 SQL 注入防护
- JWT token 安全管理
- CORS 和 CSRF 防护
- 敏感数据加密

## 📊 Logging & Monitoring

### Logging Strategy
- 使用结构化日志格式
- 记录关键业务操作
- 错误日志包含堆栈信息
- 避免记录敏感信息

### Monitoring Points
- API 响应时间
- 数据库查询性能
- 爬虫任务状态
- 系统资源使用

## 🔄 Git Workflow

### Commit Message Format
```
<type>(<scope>): <description>

Types: feat, fix, docs, style, refactor, test, chore, perf
Scopes: backend, frontend, spider, docs, ci
```

### Branch Naming
- `feature/功能描述`
- `bugfix/bug描述`
- `hotfix/紧急修复`
- `docs/文档更新`

## 🎯 Key Principles

1. **类型安全第一**: TypeScript strict mode + Pydantic 严格验证
2. **性能优化**: 异步优先，合理缓存，避免 N+1 查询
3. **用户体验**: 响应式设计，加载状态，错误处理
4. **代码质量**: 单元测试覆盖，代码审查，自动化检查
5. **安全意识**: 输入验证，权限控制，数据加密
6. **文档完整**: API 文档，组件文档，架构说明

## 🚀 Quick Start Commands

### Backend Development
```bash
cd fastapi_backend
make install      # 安装依赖
make test         # 运行测试
make lint         # 代码检查
make dev          # 启动开发服务器
```

### Frontend Development
```bash
cd nextjs-frontend
pnpm install      # 安装依赖
pnpm dev          # 启动开发服务器
pnpm test         # 运行测试
pnpm build        # 构建生产版本
```

### Spider Operations
```bash
cd Spider_XHS
python main.py    # 运行爬虫
python -m pytest # 运行测试
```

> 遵循以上规则确保代码质量与团队协作效率。详细示例参考 `mdc-examples/` 目录。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JunJD) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
