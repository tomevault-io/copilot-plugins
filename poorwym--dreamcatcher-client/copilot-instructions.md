## dreamcatcher-client

> DreamCatcher是一个拍照辅助工具的后端API，提供用户认证、拍摄计划管理和LLM聊天功能。

# DreamCatcher API 文档

## 概述

DreamCatcher是一个拍照辅助工具的后端API，提供用户认证、拍摄计划管理和LLM聊天功能。

**基础URL**: `http://localhost:8000/api/v1`

## 认证方式

API使用Bearer Token认证。在请求头中包含：
```
Authorization: Bearer <your_token>
```

## API端点

### 认证相关 (`/auth`)

#### 1. 用户注册
```http
POST /auth/register
```

**请求体**:
```json
{
  "user_name": "用户名",
  "email": "user@example.com",
  "password": "password123"
}
```

**响应**:
```json
{
  "user_id": "uuid",
  "user_name": "用户名",
  "email": "user@example.com",
  "message": "注册成功",
  "success": true
}
```

**说明**:
- user_name: 1-50个字符
- password: 6-100个字符

#### 2. 用户登录
```http
POST /auth/login
```

**请求体**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**响应**:
```json
{
    "user": {
        "user_id": "89f0f3a0-4c1e-4a41-bb8e-a786dd0828b4",
        "user_name": "Alice Smith",
        "email": "alice@example.com"
    },
    "token": {
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI4OWYwZjNhMC00YzFlLTRhNDEtYmI4ZS1hNzg2ZGQwODI4YjQiLCJlbWFpbCI6ImFsaWNlQGV4YW1wbGUuY29tIiwiZXhwIjoxNzQ5MTk3MTEzfQ.IGzi1rGNDOw6oibqk-QEDaSqwL5Vfn4Vwryt0bYflVc",
        "token_type": "bearer",
        "expires_in": 1800
    },
    "message": "登录成功"
}
```

#### 3. 获取当前用户信息
```http
GET /auth/me
```

**需要认证**: ✅

**响应**:
```json
{
  "user_id": "uuid",
  "user_name": "用户名",
  "email": "user@example.com"
}
```

#### 4. 获取当前用户详细信息
```http
GET /auth/me/detail
```

**需要认证**: ✅

**响应**:
```json
{
  "user_id": "uuid",
  "user_name": "用户名",
  "email": "user@example.com",
  "created_at": "2023-01-01T00:00:00Z",
  "updated_at": "2023-01-01T00:00:00Z"
}
```

#### 5. 更新当前用户信息
```http
PUT /auth/me
```

**需要认证**: ✅

**请求体**:
```json
{
  "user_name": "新用户名",
  "email": "new@example.com",
  "password": "newpassword123"
}
```

**响应**:
```json
{
  "user_id": "uuid",
  "user_name": "新用户名",
  "email": "new@example.com"
}
```

#### 6. 修改密码
```http
POST /auth/change-password
```

**需要认证**: ✅

**请求体**:
```json
{
  "old_password": "oldpassword",
  "new_password": "newpassword123"
}
```

**响应**:
```json
{
  "message": "密码修改成功",
  "success": true
}
```

#### 7. 根据ID获取用户信息
```http
GET /auth/user/{user_id}
```

**需要认证**: ✅

**路径参数**:
- `user_id`: 用户UUID

**响应**:
```json
{
  "user_id": "uuid",
  "user_name": "用户名",
  "email": "user@example.com"
}
```

#### 8. 验证令牌
```http
POST /auth/verify-token
```

**需要认证**: ✅

**响应**:
```json
{
  "user_id": "uuid",
  "user_name": "用户名",
  "email": "user@example.com"
}
```

### LLM聊天 (`/llm`)

#### 1. LLM聊天
```http
POST /llm/chat
```

**需要认证**: ✅

**请求体**:
```json
{
  "query": "用户的问题或请求"
}
```

**响应**:
```json
{
  "response": "LLM的回复内容",
  "success": true,
  "message": "请求处理成功"
}
```

**支持的功能**:
- 查询拍摄计划
- 创建新的拍摄计划
- 获取地点经纬度
- 查询天气信息
- 获取当前时间

#### 2. 检查LLM服务状态
```http
GET /llm/health
```

**响应**:
```json
{
  "status": "healthy",
  "service": "LLM Chat Service",
  "message": "LLM服务运行正常"
}
```

### 拍摄计划管理 (`/plans`)

#### 1. 获取指定拍摄计划
```http
GET /plans/{plan_id}
```

**需要认证**: ✅

**路径参数**:
- `plan_id`: 计划UUID

**响应**:
```json
{
        "name": "Sunset Time-lapse",
        "description": "Capture sunset over the city skyline",
        "start_time": "2025-06-11T02:30:00+08:00",
        "camera": {
            "focal_length": 35.0,
            "position": [
                120.1536,
                30.2875,
                100.0
            ],
            "rotation": [
                0.0,
                0.0,
                0.0,
                1.0
            ]
        },
        "tileset_url": "https://example.com/tileset.json",
        "user_id": "89f0f3a0-4c1e-4a41-bb8e-a786dd0828b4",
        "id": "b000da98-a72c-48a3-81ec-a78d67f67204",
        "created_at": "2025-06-05T21:44:31.003196+08:00",
        "updated_at": "2025-06-05T21:44:31.003196+08:00"
    }
```

#### 2. 获取拍摄计划列表
```http
GET /plans/
```

**需要认证**: ✅

**查询参数**:
- `skip`: 跳过的记录数（默认0）
- `limit`: 限制返回数量（默认100）

**响应**:
```json
[
  {
        "name": "Sunset Time-lapse",
        "description": "Capture sunset over the city skyline",
        "start_time": "2025-06-11T02:30:00+08:00",
        "camera": {
            "focal_length": 35.0,
            "position": [
                120.1536,
                30.2875,
                100.0
            ],
            "rotation": [
                0.0,
                0.0,
                0.0,
                1.0
            ]
        },
        "tileset_url": "https://example.com/tileset.json",
        "user_id": "89f0f3a0-4c1e-4a41-bb8e-a786dd0828b4",
        "id": "b000da98-a72c-48a3-81ec-a78d67f67204",
        "created_at": "2025-06-05T21:44:31.003196+08:00",
        "updated_at": "2025-06-05T21:44:31.003196+08:00"
    },
    {
        ...
    }
    ...
]
```

#### 3. 创建拍摄计划
```http
POST /plans/
```

**需要认证**: ✅

**请求体**:
```json
{
  "name": "Sunset Time-lapse",
  "description": "Capture sunset over the city skyline",
  "start_time": "2025-06-10T18:30:00Z",
  "camera": {
    "focal_length": 35.0,
    "position": [
      120.1536,
      30.2875,
      100.0
    ],
    "rotation": [
      0.0,
      0.0,
      0.0,
      1.0
    ]
  },
  "tileset_url": "https://example.com/tileset.json",
  "user_id": "89f0f3a0-4c1e-4a41-bb8e-a786dd0828b4"
}
```

**响应**:
```json
{
    "name": "Sunset Time-lapse",
    "description": "Capture sunset over the city skyline",
    "start_time": "2025-06-11T02:30:00+08:00",
    "camera": {
        "focal_length": 35.0,
        "position": [
            120.1536,
            30.2875,
            100.0
        ],
        "rotation": [
            0.0,
            0.0,
            0.0,
            1.0
        ]
    },
    "tileset_url": "https://example.com/tileset.json",
    "user_id": "89f0f3a0-4c1e-4a41-bb8e-a786dd0828b4",
    "id": "d0cbcba5-90ff-4cf0-8bf3-467018f9ec7b",
    "created_at": "2025-06-06T16:19:24.050622+08:00",
    "updated_at": "2025-06-06T16:19:24.050622+08:00"
}
```

#### 4. 更新拍摄计划
```http
PATCH /plans/{plan_id}
```

**需要认证**: ✅

**路径参数**:
- `plan_id`: 计划UUID

**请求体**:
要修改的字段
```json
{
  "description": "Updated description for sunset time-lapse",
  "tileset_url": "https://cdn.example.com/new_tileset.json"
}
```

**响应**:
```json
{
    "name": "Sunset Time-lapse",
    "description": "Updated description for sunset time-lapse",
    "start_time": "2025-06-11T02:30:00+08:00",
    "camera": {
        "focal_length": 35.0,
        "position": [
            120.1536,
            30.2875,
            100.0
        ],
        "rotation": [
            0.0,
            0.0,
            0.0,
            1.0
        ]
    },
    "tileset_url": "https://cdn.example.com/new_tileset.json",
    "user_id": "89f0f3a0-4c1e-4a41-bb8e-a786dd0828b4",
    "id": "b000da98-a72c-48a3-81ec-a78d67f67204",
    "created_at": "2025-06-05T21:44:31.003196+08:00",
    "updated_at": "2025-06-06T16:21:20.673033+08:00"
}
```

#### 5. 删除拍摄计划
```http
DELETE /plans/{plan_id}
```

**需要认证**: ✅

**路径参数**:
- `plan_id`: 计划UUID

**响应**: HTTP 204 No Content

#### 6. 管理员获取所有计划
TODO

## 错误响应

所有API端点在出错时会返回以下格式的错误响应：

```json
{
  "detail": "错误详细信息",
  "status_code": 400
}
```

### 常见错误代码

- `400 Bad Request`: 请求参数无效
- `401 Unauthorized`: 未提供有效的认证令牌
- `403 Forbidden`: 权限不足
- `404 Not Found`: 资源不存在
- `422 Unprocessable Entity`: 请求格式正确但内容无效
- `500 Internal Server Error`: 服务器内部错误
- `503 Service Unavailable`: 服务不可用

## 注意事项

1. 所有时间字段使用ISO 8601格式（UTC时间）
2. UUID字段使用标准UUID格式
3. 用户只能访问自己创建的拍摄计划
4. 密码要求6-100个字符
5. 用户名要求1-50个字符
6. 邮箱地址必须是有效格式

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/poorwym)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/poorwym)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
