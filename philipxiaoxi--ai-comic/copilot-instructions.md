## ai-comic

> 用户通过 AI 生成连载漫画，支持角色库管理、分镜脚本生成、漫画图片生成。

# AGENTS.md

AI 漫画创作平台 - 前后端分离架构。

## 项目概述

用户通过 AI 生成连载漫画，支持角色库管理、分镜脚本生成、漫画图片生成。

## 开发命令

```bash
# 后端开发 (server 目录)
cd server && npm run dev      # 启动开发服务器 (端口 7001)
cd server && npm test         # 运行测试

# 前端开发 (web 目录)
cd web && npm run dev         # 启动开发服务器 (端口 3000，代理到后端 7001)
cd web && npm run build       # 构建生产版本
```

## 架构

### 后端 (server/)
- **框架**: Egg.js
- **数据库**: SQLite (better-sqlite3)，初始化脚本在 `database/init.js`
- **认证**: JWT + HttpOnly Cookie，中间件在 `app/middleware/jwt.js`

### 前端 (web/)
- **框架**: Vue 3 + Vite
- **UI**: Vuetify (Material Design)
- **状态管理**: Pinia
- **路由**: Vue Router

## 环境变量

生产环境必须设置:
- `EGG_KEYS`: 应用密钥
- `JWT_SECRET`: JWT 签名密钥

## 开发约束

**前端开发必须使用 `frontend-design` 技能**：所有前端页面和组件开发时，必须调用 `/frontend-design` 技能确保设计质量。

---
> Source: [philipxiaoxi/ai-comic](https://github.com/philipxiaoxi/ai-comic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
