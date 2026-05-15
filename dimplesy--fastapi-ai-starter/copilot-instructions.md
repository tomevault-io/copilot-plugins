## fastapi-ai-starter

> 这是一个基于 fastapi 的后端 API 开发框架，使用的技术栈有 sqlmodel + alembic + fastapi-pagination + celery + pydantic + supervisord

# fastapi-ai-starter 项目开发规范

这是一个基于 fastapi 的后端 API 开发框架，使用的技术栈有 sqlmodel + alembic + fastapi-pagination + celery + pydantic + supervisord

## 输出格式
- 总是使用中文

## 常用命令

### 包管理 (UV)
```bash
uv sync                    # 安装依赖
uv run <command>           # 使用虚拟环境运行命令
```

### 开发服务器
```bash
uv run main.py             # 启动 FastAPI 应用 (端口 8000)
uv run celery -A celery_tasks worker --loglevel=INFO  # Celery Worker
uv run celery -A celery_tasks beat --loglevel=INFO    # Celery Beat (定时任务)
supervisord -c supervisord.conf  # 使用 Supervisor 管理所有进程
```

### Docker
```bash
docker-compose up -d       # 启动所有服务
docker-compose up -d pgvector redis  # 仅启动数据库和 Redis
docker-compose down        # 停止服务
```

### 代码质量
```bash
uv run ruff check .        # 代码检查
uv run ruff format .       # 代码格式化
uv run mypy .              # 类型检查
pre-commit run --all-files # 运行所有预提交钩子
```

### 数据库迁移
```bash
./scripts/migration.sh "描述"   # 生成新迁移文件
./scripts/migrate.sh            # 应用迁移
alembic upgrade head            # 应用迁移 (直接)
alembic downgrade -1            # 回滚一个版本
```

## 架构概览

### 服务层架构
项目采用**服务工厂模式**管理服务生命周期和依赖：

- `app/services/base.py` - 服务基类 (Service ABC)
- `app/services/schema.py` - ServiceType 枚举定义所有服务类型
- `app/services/manager.py` - ServiceManager 单例，管理服务注册和创建
- `app/services/factory.py` - ServiceFactory 基类，自动推断服务依赖
- `app/services/deps.py` - 依赖注入函数 (get_db_service, get_settings_service, session_scope)

**添加新服务的步骤**：
1. 在 `app/services/<name>/` 创建服务模块
2. 定义 `service.py` (继承 Service) 和 `factory.py` (继承 ServiceFactory)
3. 在 `ServiceType` 枚举中添加新服务类型
4. ServiceManager 会自动发现并注册工厂

### 数据库层
- 使用 SQLModel (Pydantic + SQLAlchemy) 作为 ORM
- 异步引擎 (`AsyncEngine`) + `async_sessionmaker`
- PostgreSQL + pgvector 扩展支持向量搜索
- 数据库迁移通过 Alembic 自动执行 (应用启动时)

**模型目录结构**：
```
app/services/database/models/<model_name>/
├── model.py    # SQLModel 模型定义
└── crud.py     # CRUD 操作
```

### API 路由
- 路由按版本组织：`app/api/v1/`
- 主路由注册在 `app/api/router.py`
- 所有 API 前缀为 `/api/v1/`
- 使用 `fastapi-pagination` 支持分页

### Celery 任务
- Celery 应用定义在 `celery_tasks/celery.py`
- 配置文件：`celery_tasks/celeryconfig.py`
- 任务放在 `celery_tasks/workers/` 目录
- 新任务需在 `celeryconfig.py` 的 `include` 列表中注册

### 配置管理
- `app/services/settings/base.py` - Settings 类 (Pydantic Settings)
- 环境变量从 `.env` 文件加载
- 必需变量：`ENVIRONMENT`, `DATABASE_URL`, `JWT_SECRET`

## 项目约定

### 数据库 Session 使用
```python
from app.api.util import DbSession

# 在 API 端点中使用
async def endpoint(session: DbSession):
    # session 自动提交/回滚
```

### 异步代码
- 全项目使用异步 (async/await)
- 数据库操作全部异步
- Celery 任务是同步函数

### 代码风格
- Python 3.12+
- Ruff 配置：line-length 120, target py312
- 类型注解必需，使用 MyPy 检查

---
> Source: [DimplesY/fastapi-ai-starter](https://github.com/DimplesY/fastapi-ai-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
