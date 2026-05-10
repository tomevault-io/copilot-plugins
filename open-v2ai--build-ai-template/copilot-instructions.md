## quick-reference

> Build AI Template 快速参考指南和常用命令


# Build AI Template - 快速参考指南

## 项目快速导航

- **后端入口**: `api/app/main.py`
- **前端入口**: `web/app/[locale]/page.tsx`
- **聊天组件**: `web/components/chat-content-page.tsx`
- **LLM 工具**: `api/app/agents/llm.py`

### 常用目录

```bash
api/app/
├── models/       # SQLModel 数据模型
├── schemas/      # Pydantic 验证模式
├── routers/v1/   # API v1 路由
├── crud/         # 数据库 CRUD 操作
└── services/     # 业务逻辑服务

web/
├── app/[locale]/ # 页面路由
├── components/   # React 组件
└── app/messages/ # 国际化翻译
```

## 开发工作流程

1. **后端优先**: 在 `api/app/` 中添加模型、路由、服务。
2. **数据库迁移**: 运行 `alembic revision --autogenerate` 和 `alembic upgrade head`。
3. **前端实现**: 在 `web/components/` 中创建组件。
4. **国际化**: 添加翻译到 `web/app/messages/`。
5. **测试验证**: 确保功能正常。

## 常用开发命令

### 后端 (`/api`)

```bash
# 安装/同步依赖
uv sync

# 运行开发服务器
uv run python -m app.main

# 创建数据库迁移
alembic revision --autogenerate -m "描述"

# 执行迁移
alembic upgrade head
```

### 前端 (`/web`)

```bash
# 安装依赖
pnpm install

# 运行开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 添加 Shadcn UI 组件
pnpm dlx shadcn-ui@latest add [component-name]
```

### 部署 (`/deploy` 或 `/deploy-test`)

```bash
# 启动服务
make up

# 停止服务
make down

# 查看日志
make logs

# 完全重建
make rebuild
```

## 常见问题快速排查

- **数据库迁移失败**: 检查 `DATABASE_URL` 配置是否正确。
- **API 认证失败**: 确认 `.env` 文件中的 `SECRET_KEY` 和 JWT 配置正确。
- **前端翻译缺失**: 确保 `web/app/messages/` 中的 `zh.json` 和 `en.json` 包含对应的翻译键。
- **AI 调用失败**: 检查 `.env` 文件中的 `AGENT_API_KEY` 是否配置正确。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
