## ai-moive-studio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

AICG内容分发平台是一个AI驱动的长文本到视频自动转换系统，包含完整的前后端架构、用户管理系统、文件存储和异步任务处理功能。

## 开发环境设置

### 基础设施启动
```bash
./scripts/start.sh
```

### 后端开发
```bash
cd backend
uv sync                          # 安装依赖
alembic upgrade head             # 数据库迁移
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000  # 开发服务器
```

### 前端开发
```bash
cd frontend
npm install                     # 安装依赖
npm run dev                     # 开发服务器（默认端口3000，会自动检测占用）
```

### 服务地址
- API文档: http://localhost:8000/docs
- MinIO控制台: http://localhost:9001 (minioadmin/minioadmin)
- 前端应用: http://localhost:3001 或 http://localhost:3000

## 项目架构

### 后端架构 (FastAPI + SQLAlchemy + Alembic)

**核心模块结构:**
- `src/core/`: 核心配置、数据库、安全、日志
- `src/models/`: SQLAlchemy数据模型（基于BaseModel的时间戳混入）
- `src/api/v1/`: API路由，按功能模块组织
- `src/services/`: 业务逻辑层，包括头像上传等服务
- `migrations/`: Alembic数据库迁移文件

**重要设计模式:**
- 使用异步SQLAlchemy (AsyncSession)
- 模型继承BaseModel（UUID主键 + 时间戳混入）
- 配置使用Pydantic Settings，支持环境变量覆盖
- JWT认证 + Bearer Token授权
- 统一的错误处理和响应格式

### 前端架构 (Vue 3 + Element Plus + Pinia)

**核心结构:**
- `src/router/`: 路由配置，包含认证守卫
- `src/stores/`: Pinia状态管理（auth.js处理认证）
- `src/services/`: API服务层，统一HTTP请求处理
- `src/views/`: 页面组件
- `src/utils/`: 工具函数（包含日期时区处理）

**关键特性:**
- 响应式拦截器处理401错误，区分登录失败和token过期
- 用户时区感知的日期显示
- 现代化UI设计系统，中文界面
- 文件上传进度反馈

### 数据库和存储

**PostgreSQL:**
- 所有时间戳以UTC格式存储
- 用户模型包含时区偏好字段
- 支持JSON字段存储用户偏好设置

**MinIO对象存储:**
- `aicg-files`: 通用文件存储桶
- `aicg-avatars`: 用户头像存储桶
- 支持图片压缩和格式验证

## 常用开发命令

### 后端
```bash
# 开发服务器
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# 数据库操作
alembic revision --autogenerate -m "描述"  # 创建迁移
alembic upgrade head                       # 应用迁移
alembic downgrade -1                       # 回滚迁移

# 测试
pytest                                    # 运行所有测试
pytest tests/unit/                        # 单元测试
pytest tests/integration/                 # 集成测试
pytest -m "not slow"                     # 排除慢速测试

# 代码质量
black src/                               # 代码格式化
isort src/                               # 导入排序
mypy src/                                # 类型检查
flake8 src/                              # 代码风格检查
```

### 前端
```bash
# 开发
npm run dev                              # 开发服务器

# 构建
npm run build                            # 生产构建
npm run preview                          # 预览构建结果

# 测试
npm run test:e2e                         # E2E测试
npm run test:e2e:ui                      # E2E测试UI
npm run test:e2e:debug                   # E2E调试模式

# 代码质量
npm run lint                             # ESLint检查和修复
npm run format                           # Prettier格式化
```

## 关键配置

### 环境变量 (.env)
系统支持从.env文件加载配置，主要配置项：
- `DATABASE_URL`: PostgreSQL连接字符串
- `REDIS_URL`: Redis连接字符串
- `JWT_SECRET_KEY`: JWT密钥
- `MINIO_ENDPOINT/ACCESS_KEY/SECRET_KEY`: MinIO配置

### 时间和时区处理
- 数据库存储UTC时间，使用`datetime.now(timezone.utc)`
- 前端根据用户时区偏好动态显示本地时间
- 用户模型包含`timezone`和`language`字段

### 认证和授权
- JWT Token有效期30分钟
- API拦截器自动处理token过期跳转
- 区分登录接口和其他接口的401错误处理

## 文件上传和服务

### 头像上传服务 (`src/services/avatar.py`)
- 支持JPG/PNG/WebP格式，最大5MB
- 自动压缩到200x200像素
- MinIO存储，返回可访问URL

### API错误处理
前端API拦截器 (`frontend/src/services/api.js`)：
- 401错误：区分登录失败和token过期
- 422错误：表单验证错误详细显示
- 网络错误：统一提示和重试机制

## 测试策略

- 后端：pytest + pytest-asyncio，支持单元、集成、E2E测试
- 前端：Playwright E2E测试，包含认证流程和响应式测试
- 测试标记：`unit`、`integration`、`e2e`、`slow`

---
> Source: [869413421/ai-moive-studio](https://github.com/869413421/ai-moive-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
