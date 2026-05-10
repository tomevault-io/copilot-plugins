## deployment-devops

> 部署和运维规范，包含 Docker、CI/CD 和监控最佳实践


# 部署和运维规范

## 技术栈

- **容器化**: Docker + Docker Compose
- **CI/CD**: GitHub Actions
- **反向代理**: Nginx
- **数据库**: PostgreSQL 15+ + Redis 7+
- **监控**: 日志文件 + 健康检查

## 项目部署结构

项目包含独立的生产环境 (`deploy/`) 和测试环境 (`deploy-test/`) 配置。

```bash
deploy/
├── docker-compose.yaml     # 生产环境编排
├── nginx/default.conf      # Nginx 配置
├── volumes/                # 数据卷挂载 (db, redis)
├── .env.example            # 环境变量模板
├── makefile                # 部署命令
└── README.md               # 部署说明
```

## Docker 配置

使用多阶段构建 (`multi-stage builds`) 优化镜像大小和安全性。

### 后端 Dockerfile (`api/Dockerfile`)

- **builder 阶段**: 安装 `uv` 并下载所有 Python 依赖。
- **production 阶段**:
  - 使用 `python:3.12-slim` 基础镜像。
  - 创建非 root 用户 `appuser` 并使用。
  - 从 `builder` 阶段复制虚拟环境。
  - 添加 `HEALTHCHECK` 指令。
  - 启动命令: `uvicorn app.main:app`

### 前端 Dockerfile (`web/Dockerfile`)

- **builder 阶段**: 使用 `pnpm` 安装依赖并构建 Next.js 应用 (`pnpm build`)。
- **production 阶段**:
  - 使用 `node:18-alpine` 基础镜像。
  - 创建非 root 用户 `nextjs` 并使用。
  - 复制 `.next/standalone` 和 `.next/static` 以支持独立运行。
  - 添加 `HEALTHCHECK` 指令。
  - 启动命令: `node server.js`

## 部署命令

通过 `Makefile` 简化部署操作，所有命令在 `deploy/` 或 `deploy-test/` 目录下执行。

```makefile
# 核心命令
up:       ## 启动所有服务 (docker-compose up -d)
down:     ## 停止所有服务 (docker-compose down)
build:    ## (重新)构建 Docker 镜像
restart:  ## 重启所有服务
rebuild:  ## 完全重建 (down -> clean -> build -> up)

# 运维命令
logs:     ## 查看所有服务日志
status:   ## 查看服务状态
health:   ## 检查服务健康状态 (curl)
migrate:  ## 执行数据库迁移 (alembic upgrade head)
backup:   ## 备份数据库 (pg_dump)
restore:  ## 恢复数据库 (psql)
update:   ## 更新代码并重建服务 (git pull + rebuild)

# 调试命令
shell-api:      ## 进入 API 容器
shell-postgres: ## 进入数据库容器
```

## 环境变量配置

在部署前，必须将 `.env.example` 复制为 `.env` 并填写所有必要的值。

**关键配置项**:

- `ENV`: `production` 或 `test`
- `DATABASE_URL`: PostgreSQL 连接字符串
- `REDIS_URL`: Redis 连接字符串
- `SECRET_KEY`: 用于 JWT 签名的密钥
- `MAIL_*`: 邮件服务配置
- `AGENT_*`: AI 服务 (如 OpenAI) 的 API Key 和配置

## 监控和日志

### 健康检查

- **API**: `/api/v1/system/health`
  - **基础检查**: 返回服务状态和版本。
  - **详细检查**: `/health/detailed`，检查数据库、Redis 连接和系统资源（CPU、内存）。如果资源使用率过高或连接失败，返回 `503 Service Unavailable`。
- **Web**: 前端应用也应提供一个简单的健康检查端点。

### 日志

- 所有服务通过 Docker Compose 将日志输出到标准输出 (`stdout`)。
- 使用 `make logs` 或 `docker-compose logs -f <service_name>` 查看实时日志。
- 在生产环境中，应配置日志聚合系统（如 ELK, Grafana Loki）来收集和分析日志。

## 部署清单和最佳实践

### 部署前检查清单

- [ ] 环境变量 `.env` 已正确配置。
- [ ] SSL 证书已配置在 Nginx 中。
- [ ] 数据库迁移脚本已测试并准备就绪。
- [ ] 数据库备份策略已配置。
- [ ] 已完成安全扫描，无高危漏洞。

### 运维最佳实践

- **定期备份**: 每日自动备份数据库。
- **监控告警**: 设置资源使用率和服务健康状态告警。
- **安全更新**: 定期更新基础镜像和应用依赖。
- **灾难恢复**: 定期测试数据库的备份恢复流程。
- **文档同步**: 保持部署文档与实际流程一致。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
