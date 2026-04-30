## xhs-comment

> 后端服务基于 FastAPI 构建，采用模块化API架构。详细的架构说明参见 [xhs_backend/api/ARCHITECTURE.md](mdc:xhs_backend/api/ARCHITECTURE.md)。

# 后端 API 端点与规范

后端服务基于 FastAPI 构建，采用模块化API架构。详细的架构说明参见 [xhs_backend/api/ARCHITECTURE.md](mdc:xhs_backend/api/ARCHITECTURE.md)。

## API 文档

- **权威API参考**: FastAPI自动生成OpenAPI交互式文档 (通常位于 `/docs`) 和 ReDoc 文档 (通常位于 `/redoc`)。这些是查找最新API端点、请求/响应模型和参数的权威来源。
- **高级指南**: [xhs_backend/README.md](mdc:xhs_backend/README.md) 可能包含关于API使用的高级指南和概述。

## API 架构概览

### v1 模块化API结构

#### 用户管理 (`/api/v1/user/`)
- **用户认证**: [xhs_backend/api/v1/user/auth/](mdc:xhs_backend/api/v1/user/auth)
  - 登录管理: [xhs_backend/api/v1/user/auth/login.py](mdc:xhs_backend/api/v1/user/auth/login.py)
  - SSO单点登录: [xhs_backend/api/v1/user/auth/sso.py](mdc:xhs_backend/api/v1/user/auth/sso.py)
  - 令牌管理: [xhs_backend/api/v1/user/auth/token.py](mdc:xhs_backend/api/v1/user/auth/token.py)
- **用户配置**: [xhs_backend/api/v1/user/profile.py](mdc:xhs_backend/api/v1/user/profile.py)
- **用户笔记**: [xhs_backend/api/v1/user/notes.py](mdc:xhs_backend/api/v1/user/notes.py)

#### 内容管理 (`/api/v1/content/`)
- **评论管理**: [xhs_backend/api/v1/content/comments.py](mdc:xhs_backend/api/v1/content/comments.py)
- **笔记管理**: [xhs_backend/api/v1/content/notes.py](mdc:xhs_backend/api/v1/content/notes.py)

#### 系统管理 (`/api/v1/system/`)
- **抓取规则**: [xhs_backend/api/v1/system/capture_rules.py](mdc:xhs_backend/api/v1/system/capture_rules.py)
- **网络数据**: [xhs_backend/api/v1/system/network_data.py](mdc:xhs_backend/api/v1/system/network_data.py)

#### 通知管理 (`/api/v1/notification/`)
- **通知处理**: [xhs_backend/api/v1/notification/notifications.py](mdc:xhs_backend/api/v1/notification/notifications.py)

#### 传统端点 (`/api/endpoints/`)
- **用户笔记**: [xhs_backend/api/endpoints/user_notes.py](mdc:xhs_backend/api/endpoints/user_notes.py) - 用户笔记相关的传统API端点

## 主要API端点

### 抓取规则管理 (`/api/v1/system/capture-rules`)
- `GET /capture-rules` - 获取启用的规则（插件使用，无需认证）
- `GET /capture-rules/all` - 获取所有规则（管理使用，需要认证）
- `POST /capture-rules` - 创建新规则
- `PUT /capture-rules/{name}` - 更新规则
- `DELETE /capture-rules/{name}` - 删除规则

### 网络数据处理 (`/api/v1/system/network-data`)
- `POST /network-data` - 接收网络数据（插件使用，无需认证）
- `GET /network-data` - 查询网络数据（管理使用，需要认证）
- `GET /network-data/stats` - 获取统计信息
- `POST /network-data/batch-process` - 批量处理

### 系统监控 (`/api/v1/system/`)
- `GET /status` - 系统状态信息
- `GET /health` - 健康检查（无需认证）
- `GET /version` - 版本信息
- `GET /database-stats` - 数据库统计
- `GET /metrics` - Prometheus兼容指标

### 评论管理 (`/api/v1/content/comments`)
- `GET /comments` - 获取评论列表（支持过滤和分页）
- `GET /comments/{comment_id}` - 获取评论详情
- `PUT /comments/{comment_id}` - 更新评论（例如状态）
- `DELETE /comments/{comment_id}` - 删除评论
- `POST /comments/batch/status` - 批量状态更新

### 笔记管理 (`/api/v1/content/notes`)
- `GET /notes` - 获取笔记列表
- `GET /notes/{note_id}` - 获取笔记详情
- `POST /notes` - 创建笔记
- `PUT /notes/{note_id}` - 更新笔记

### 通知管理 (`/api/v1/notification/notifications`)
- `GET /notifications` - 获取通知列表
- `GET /notifications/{notification_id}` - 获取通知详情
- `POST /notifications` - 创建通知
- `PUT /notifications/{notification_id}` - 更新通知状态

### 认证相关 (`/api/v1/user/auth/`)
- `POST /auth/login` - 用户登录
- `POST /auth/sso/initiate` - 发起SSO登录
- `GET /auth/sso/callback` - SSO回调处理
- `POST /auth/token/refresh` - 刷新访问令牌
- `GET /users/me` - 获取当前用户信息（需认证）
- `PUT /users/me` - 更新用户配置

## 插件专用端点

### 无认证端点（便于插件快速访问）
- `GET /api/v1/system/capture-rules` - 获取抓取规则
- `POST /api/v1/system/network-data` - 提交网络数据
- `GET /api/v1/system/health` - 健康检查

### 插件数据提交格式
插件向后端提交数据的标准格式：

```json
{
  "url": "请求URL",
  "method": "HTTP方法",
  "headers": {},
  "requestData": {},
  "responseData": {},
  "timestamp": "ISO时间戳",
  "source": "插件标识"
}
```

## 认证与授权

- **Bearer Token认证**: 大部分API端点需要通过 `Authorization: Bearer <JWT_TOKEN>` 进行认证
- **无认证端点**: 插件专用端点（抓取规则获取、数据提交、健康检查）无需认证，便于插件快速访问
- **SSO集成**: 支持Keycloak单点登录，具体配置见 [xhs_backend/KEYCLOAK_SETUP.md](mdc:xhs_backend/KEYCLOAK_SETUP.md)
- **权限控制**: 通过依赖注入 [xhs_backend/api/deps.py](mdc:xhs_backend/api/deps.py) 实现细粒度权限控制

## 向后兼容性

- **兼容层**: [xhs_backend/api/v1/compatibility.py](mdc:xhs_backend/api/v1/compatibility.py) 确保旧API路径仍然可用
- **路由重定向**: 原有API路径自动重定向到新的模块化端点
- **渐进式迁移**: 新功能使用v1模块化端点，现有功能保持兼容

## 数据模型

所有请求/响应模型定义在 [xhs_backend/api/models/](mdc:xhs_backend/api/models) 目录下：
- **认证模型**: [xhs_backend/api/models/auth.py](mdc:xhs_backend/api/models/auth.py)
- **通用模型**: [xhs_backend/api/models/common.py](mdc:xhs_backend/api/models/common.py)
- **内容模型**: [xhs_backend/api/models/content.py](mdc:xhs_backend/api/models/content.py)
- **网络模型**: [xhs_backend/api/models/network.py](mdc:xhs_backend/api/models/network.py)
- **通知模型**: [xhs_backend/api/models/notification.py](mdc:xhs_backend/api/models/notification.py)
- **用户模型**: [xhs_backend/api/models/user.py](mdc:xhs_backend/api/models/user.py)

详细的数据模型定义和响应格式请参考OpenAPI文档。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yunkst) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
