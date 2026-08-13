## environment-config

> 编辑环境/部署配置时参考——ENV_FILE、变量、prod 不提交


# 环境配置（按需参考）

- **位置**：`deployment/environments/env.{example|local|dev|stg|prod}`；prod 不提交 Git。
- **加载**：Backend 用 `ENV_FILE` 指定；默认查找 `.env.local` → `../deployment/environments/env.local` → `.env`。例：`ENV_FILE=../deployment/environments/env.local python -m uvicorn app.main:app --reload`。
- **常用变量**：`DATABASE_URL` / `DATABASE_URL_SYNC`、`REDIS_URL`、`CELERY_BROKER_URL`、`SECRET_KEY`、`HASHIDS_SALT`、`ENVIRONMENT`、`ALLOWED_ORIGINS`。Local：PostgreSQL 5433、Redis 6379；Dev 用 Docker 脚本。
- **安全**：env.prod、frontend/env.prod、docker-compose.prod.yml 不提交；`chmod 600` 敏感文件；`git check-ignore -v deployment/environments/env.prod` 校验。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
