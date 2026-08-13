## database-management

> 编辑 models/迁移/DB 脚本时参考——Alembic 流程、本地 vs 服务器命令


# 数据库管理（按需参考）

- **改表**：禁止直接执行 DDL。改 `backend/app/models/` → `alembic.sh local autogen "描述"` 或 `db-docker.sh dev autogen "描述"` → `upgrade`。迁移文件在 `backend/migrations/versions/`，提交后不得改，只可新增迁移。
- **本地**：有 venv，用 `./scripts/db/alembic.sh local current|history|upgrade|downgrade|autogen "描述"`；先 `cd backend && source .venv/bin/activate`。
- **服务器 (dev/stg/prod)**：无 Python，用 Docker：`./deployment/scripts/db-docker.sh {dev|stg|prod} current|upgrade|autogen "描述"`。环境文件 `deployment/environments/env.{local|dev|stg|prod}`。
- **查库**：本地调试优先用 MCP postgres-supplynexus-local（只读）；写操作与迁移用 Alembic/脚本。
- **安全**：生产迁移前备份；先在 staging 测；可回滚。不同环境用不同 DB 与配置。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
