## api-development

> API 路由开发规范


# API 路由开发规范

## 路由结构

API 路由遵循 RESTful 设计：

- `GET /api/admin/users` - 获取用户列表
- `POST /api/admin/users` - 创建用户
- `GET /api/admin/users/[id]` - 获取单个用户
- `PUT /api/admin/users/[id]` - 更新用户
- `DELETE /api/admin/users/[id]` - 删除用户

## 认证中间件

所有受保护的 API 路由都会经过 [src/middleware.ts](mdc:src/middleware.ts) 认证：

- 检查 `Authorization: Bearer <token>` 头
- 验证 JWT token 有效性
- 公共 API 路径无需认证

## 响应格式

统一使用以下响应格式：

```typescript
{
  code: number,      // HTTP 状态码
  message: string,   // 响应消息
  data: any | null   // 响应数据
}
```

## 错误处理

- 使用 `createErrorResponse` 创建错误响应
- 根据错误类型返回相应的 HTTP 状态码
- 记录详细的错误日志用于调试

## 数据库操作

- 使用 Drizzle ORM 进行数据库操作
- 所有数据库操作都要进行错误处理
- 使用事务处理复杂的数据操作

## CORS 配置

- 开发环境允许所有源 (`*`)
- 生产环境限制允许的源
- 支持预检请求 (OPTIONS)

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
