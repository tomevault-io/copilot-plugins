## yw-mall

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- **Backend**: Go 1.26, go-zero 1.10.1, goctl 1.10.1, proto3/gRPC
- **Frontend**: uni-app (Vue 3 + TypeScript + Vite), Wot Design Uni, Pinia, H5 target
- **Infra**: MySQL (via ProxySQL :6033), Redis, Kafka, etcd, DTM
- **Tools**: `protoc` at `/home/carter/workspace/go/bin/protoc` (not in default PATH — use full path)

## Starting and Stopping Services

```bash
# Start everything (infra check → DDL → seed → 15 services)
./start.sh start

# Stop Go services only (infra containers stay up)
./start.sh stop

# Check running services
./start.sh status

# Re-run DB/seed bootstrap without restarting services
./start.sh bootstrap

# Drop all mall_* databases and reset (clean slate)
./start.sh nuke
```

After startup, verify seed data:
```bash
bash scripts/check-seed.sh   # exits 0 if all checks pass
```

API gateway listens at `http://localhost:18888`.

## Building and Testing Individual Services

Each service is its own Go module. Always `cd` into the service directory first:

```bash
cd mall-{name}-rpc && go build ./...
cd mall-api && go build ./...
```

To run a single service manually:
```bash
cd mall-user-rpc && go run user.go
```

No automated test suite — validation is done via `start.sh` + `scripts/check-seed.sh` + manual E2E from `docs/e2e-frontend-backend-checklist.md`.

## Frontend

```bash
cd mall-frontend
pnpm dev:h5          # dev server at http://localhost:5173 (proxies /api → :18888)
pnpm run build:h5    # production H5 build, must exit 0 with no TS errors
```

Always run pnpm from within `mall-frontend/` — running from the repo root creates a stray root-level `package.json`.

### Frontend Architecture

```
mall-frontend/src/
  api/
    request.ts       # base wrapper: uni.request + JWT header + 401 redirect
    shop.ts          # 8 functions (getShopDetail, listShops, followShop, …)
    product.ts       # listProducts, getProductDetail, searchProducts
  stores/
    user.ts          # token + userId, persisted to localStorage
    cart.ts          # count only (increment/decrement)
  types/api.ts       # all API response types (ShopItem, ProductDetailResp, …)
  styles/tokens.scss # design tokens — globally injected by vite, never @import manually
  pages/
    index/index.vue  # home: search + recommended shops + featured products
    shop/list.vue    # paginated shop list with infinite scroll
    shop/detail.vue  # banner, follow toggle, shop products grid
    product/list.vue # keyword or shopId query param → 2-col grid + infinite scroll
    product/detail.vue # price displayed as ¥(cents/100).toFixed(2), sticky add-to-cart
    login/index.vue  # placeholder (wd-empty)
```

**uni-app navigation:** use `uni.navigateTo`, `uni.reLaunch`, `uni.showToast` — there is no vue-router. Page params are read from `getCurrentPages()` options, not `$route`.

**Wot Design Uni** components are auto-imported via easycom — no manual imports needed. Do **not** add `build.rollupOptions.external` for wot-design-uni or any Vue packages; the build passes without it and adding it breaks production bundling.

## Architecture

### Service Map

15 services, all registered in `start.sh`:

| Service | Port | DB |
|---|---|---|
| mall-api (HTTP gateway) | 18888 | — |
| mall-user-rpc | 19001 | mall_user |
| mall-product-rpc | 9002 | mall_product |
| mall-order-rpc | 9003 | mall_order |
| mall-cart-rpc | 9004 | mall_cart |
| mall-payment-rpc | 9005 | mall_payment |
| mall-activity-rpc | 9010 | mall_activity |
| mall-workflow-rpc | 9012 | mall_workflow |
| mall-reward-rpc | 9013 | mall_reward |
| mall-risk-rpc | 9014 | mall_risk |
| mall-review-rpc | 9015 | mall_review |
| mall-logistics-rpc | 9016 | mall_logistics |
| mall-shop-rpc | 9017 | mall_shop |
| mall-rule-rpc | 9011 | mall_rule |
| mall-activity-async-worker | — | — |

### RPC Service Layout

Each `mall-{name}-rpc/` follows the same structure:
```
mall-{name}-rpc/
  {name}.go              # main entry
  go.mod                 # own module with replace directives to siblings
  etc/{name}.yaml        # config (DB DSN, etcd, cross-service RPC addresses)
  {name}/                # generated gRPC stubs (from protoc)
  {name}client/          # generated client wrapper (used by consumers)
  internal/
    config/config.go     # typed config struct
    svc/servicecontext.go # wires DB, Redis, RPC clients
    logic/               # one file per RPC method
    server/              # generated gRPC server wiring
  sql/{name}.sql         # DDL (applied by start.sh bootstrap)
  cmd/seed/main.go       # optional: seed program (run via start.sh bootstrap)
```

### Cross-Module Dependencies

Each service's `go.mod` uses `replace` directives to reference siblings by path:
```
replace mall-user-rpc => ../mall-user-rpc
replace mall-common => ../mall-common
```

When adding a cross-service RPC dependency to a service:
1. Add `require` + `replace` to `go.mod`
2. Add `XxxRpc zrpc.RpcClientConf` to `internal/config/config.go`
3. Add the etcd block to `etc/{name}.yaml`
4. Initialize in `internal/svc/servicecontext.go` with `zrpc.MustNewClient`

### mall-common

Shared code used across all services (no main, no `go run`):
- `errorx/` — typed error codes (HTTP status + biz code mapping)
- `proto/` — all `.proto` source files, organized by domain
- `minioutil/` — MinIO upload helper
- `result/` — standard HTTP response wrapper

### mall-api (HTTP Gateway)

HTTP entry point. Calls downstream RPCs via gRPC. Follows go-zero REST pattern:
- `internal/types/types.go` — request/response structs for all HTTP endpoints
- `internal/handler/routes.go` — route registration (JWT middleware applied per group)
- `internal/handler/*handler.go` — thin handler, calls logic
- `internal/logic/*logic.go` — logic layer, calls RPCs, builds response

Adding a new endpoint requires updating all four files plus `etc/mall-api.yaml` and `internal/svc/servicecontext.go`.

### Proto → Code Generation

Proto files live in `mall-common/proto/{service}/{service}.proto`. After editing:

```bash
export PATH=/home/carter/workspace/go/bin:$PATH
cd mall-{name}-rpc
protoc --go_out=. --go-grpc_out=. \
  --proto_path=. --proto_path=../mall-common/proto \
  ../mall-common/proto/{name}/{name}.proto
```

Generated stubs go into `mall-{name}-rpc/{name}/`. Client wrappers in `{name}client/` must be updated manually to expose new methods.

### Service Discovery

All RPCs register with etcd. Key pattern: `yw-mall/{service}-rpc` (e.g., `yw-mall/user-rpc`, `yw-mall/shop-rpc`). The `etc/*.yaml` config block:
```yaml
XxxRpc:
  Etcd:
    Hosts:
      - etcd1:2379
    Key: yw-mall/xxx-rpc
```

### go-zero Patterns to Know

- **Generated models** only expose columns from the DDL — use raw `sqlx.QueryRowsCtx` / `QueryRowCtx` for joins or extra columns.
- **CachedConn** caches by primary key — negative cache entries can block re-inserts; `start.sh` flushes `cache:*` on startup.
- **Logic files** are the place for business logic; handlers are thin wrappers that decode request → call logic → encode response.
- The `mall.api` DSL file drives `goctl api go` for handler/logic scaffolding.

### Database Access

ProxySQL on `:6033` routes to MySQL masters/slaves. DSN pattern:
```
proxysql:proxysql123@tcp(127.0.0.1:6033)/mall_{name}?charset=utf8mb4&parseTime=true&loc=Local
```

---
> Source: [yw-mall/yw-mall](https://github.com/yw-mall/yw-mall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
