## security

> 安全开发规范


# 安全开发规范

## 认证和授权

### JWT 认证

- 使用 `jose` 库进行 JWT 处理
- Access Token 用于 API 认证
- Refresh Token 用于刷新访问令牌
- Token 包含用户 ID 和权限信息

### 密码安全

- 使用 `bcryptjs` 进行密码哈希
- 密码强度验证
- 定期更新密码策略

## 数据验证

### 输入验证

- 使用 Zod 进行类型安全验证
- 所有用户输入都要验证
- 防止 SQL 注入和 XSS 攻击

### 文件上传

- 验证文件类型和大小
- 扫描恶意文件
- 限制上传路径

## CORS 配置

- 开发环境允许所有源
- 生产环境限制允许的源
- 正确配置 CORS 头

## 环境变量

- 敏感信息存储在环境变量中
- 使用 `.env.local` 进行本地开发
- 生产环境使用安全的密钥管理

## 错误处理

- 不在错误信息中暴露敏感信息
- 记录详细错误日志用于调试
- 返回用户友好的错误信息

## 安全头

- 设置适当的安全头
- 使用 HTTPS 传输
- 防止点击劫持和 XSS

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
