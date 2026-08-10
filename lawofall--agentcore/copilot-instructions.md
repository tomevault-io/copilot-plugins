## api

> FastAPI 路由、Schema、认证、错误处理开发规范


# API 开发规范

## 路由

- 所有路由注册在 `main.py`，统一 `prefix="/v1"`
- Router 是薄层：参数解析 → 调用 service → 返回结果
- 分页参数：`page: int = Query(1, ge=1)`，`page_size: int = Query(20, ge=1, le=100)`
- 删除操作返回 `{"status": "ok"}`
- RESTful 命名：`GET /list`、`POST /create`、`PUT /{id}` 全量、`PATCH /{id}` 部分、`DELETE /{id}`

## Schema 命名

| 类型 | 命名模式 |
|---|---|
| 列表项 / 详情 | `{Resource}Summary` / `{Resource}Detail` |
| 列表响应 | `{Resource}ListResponse`（`data` + `total`） |
| 创建 / 更新 | `Create{Resource}Request` / `Update{Resource}Request` |

- 主键统一 `id`，外键 `{entity}_id`，更新请求所有字段可选
- 列表响应包含 `data`、`total`、`page`、`page_size`
- 时间字段序列化为 ISO 8601
- **前端 TS 类型从 OpenAPI 生成，禁止手写**（单一真相源 = `schemas.py`）。改 schema 后两步刷新：① 后端 `uv run python scripts/dump_openapi.py` 更新 committed `apps/server/openapi.json` → ② 前端 `pnpm gen:api` 生成 `types/api.generated.ts`（生成产物，已 gitignore + `*.generated.ts` 免 lint；`postinstall` 会自动重生成）；service 文件用 `type X = components["schemas"]["X"]` 取别名引用。
- ✅ 已迁移：所有 service 的 REST 类型均用生成类型别名（`type X = components["schemas"]["X"]`）；新增/改动一律走生成类型，**勿新增手写 REST 类型**。纯 SSE/事件载荷与客户端域模型（camelCase，经 `toX` 映射）不受此约束。例外：端点无 `response_model`（如 `/readyz`、`/version`）时生成类型为无类型字典，保留手写并加注。带默认值的字段在生成类型里是可选（`?`），读取处按需 `?? 兜底`。

## 认证 & 权限

- 注入：`AuthUser`（必须登录）/ `OptionalUser`（可选）→ 见 `agentcore.deps`
- 权限在 **service 层**校验，不在 router 层

## 错误处理

- 使用 `agentcore.common.errors`：`not_found` / `forbidden` / `bad_request`
- 统一响应：`{"error": {"code": N, "message": "...", "detail": "..."}}`

## Service 层

- ID 生成 `str(uuid4())`
- ORM → Pydantic 转换放 service 私有方法
- 写操作先查再改（验证存在性和权限）
- 软删除检查：`if not entity or entity.deleted_at: raise not_found(...)`

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
