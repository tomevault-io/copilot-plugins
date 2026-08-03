## agentstore

> 平台自营的 AI Agent 市场,基于多租户 SaaS 底座。用户发现 Agent、对话、消耗积分、购买积分包;运营方在 admin 后台管理 Agent、套餐、积分、计费、品牌、健康、日志、上线就绪。

# AgentStore — Claude Code 启动须知

> 每次启动只需读本文件。

## 项目是什么

平台自营的 AI Agent 市场,基于多租户 SaaS 底座。用户发现 Agent、对话、消耗积分、购买积分包;运营方在 admin 后台管理 Agent、套餐、积分、计费、品牌、健康、日志、上线就绪。

- 后端:Go 1.25 + gorilla/mux + MongoDB,入口 `backend/cmd/server`
- CLI/MCP:`backend/cmd/agentstore`(初始化、只读 MCP)
- 前端:React 19 + TypeScript + Vite 7 + Tailwind 4,目录 `frontend/src`
- 计费:Stripe;LLM:OpenAI 兼容 provider

## V1 已完成

- 多租户 + RBAC(owner/admin/user)、JWT(access+refresh)+ `X-Tenant-ID`
- 认证全套(密码、OAuth、magic link、MFA/TOTP)
- Agent 市场(24 个精品 agent)+ 对话(SSE 流式)+ 多模态(语音/图片/文档)
- 积分系统(订阅积分 + 购买积分包,对话按 Agent 成本扣费,余额不足拦截)
- Stripe 计费(checkout、订阅、portal、税、退款、争议、优惠码)
- admin 后台(用户/租户/套餐/计费/品牌/健康/日志/API key/webhook/遥测/配置/上线就绪)
- 一键部署(docker-compose)、网页安装向导、GHCR 镜像发布、三语文档(中/英/日)

## 启动 / 测试 / 构建命令

```bash
# 本地起服务(详见 docs/DEVELOPMENT.md)
set -a && source .env && set +a
cd backend && go run ./cmd/server          # 后端 :4290
cd frontend && npm install && npm run dev   # 前端 :4280
cd backend && go run ./cmd/agentstore setup # 初始化新库

# 改完代码必须验证
cd backend && go build ./... && go vet ./... && go test ./...
cd frontend && npx tsc --noEmit && npm run lint && npm test -- --run
```

## 核心目录

- `backend/internal/api/handlers/` — HTTP handler(auth/admin/tenant/billing/chat/agent…)
- `backend/internal/models/` — MongoDB 模型结构体
- `backend/internal/db/` — 连接、索引、JSON Schema(`schema.go`)
- `backend/internal/credits/` `…/stripe/` `…/llm/` — 积分、计费、LLM(高风险)
- `backend/internal/middleware/` — 认证、租户解析、RBAC、计费拦截
- `backend/internal/bootstrap/` — 首次初始化逻辑(CLI 和 HTTP 共用)
- `frontend/src/pages/{app,admin,auth,public}/` — 各角色页面
- `frontend/src/api/client.ts` — API 客户端 + token 刷新
- `frontend/src/utils/attachments.ts`、`frontend/src/hooks/useSpeechRecognition.ts` — 多模态/语音

## 必读文档

- `docs/ARCHITECTURE.md` — 技术栈、结构、数据流
- `docs/BUSINESS_RULES.md` — 业务规则与不可误改逻辑
- `docs/DECISIONS.md` — 已定型的技术/产品决策
- `docs/DEVELOPMENT.md` — 完整开发/验证流程
- `docs/DEPLOYMENT.md` — 部署

## 开发禁区

1. **不改业务代码做归档/清理**;清理先审计、只删证明无用的。
2. **高风险路径**改动需谨慎并跑测试:`backend/internal/credits/`、`…/stripe/`、`…/llm/`、`api/handlers/chat.go`、`api/handlers/billing.go`、`api/handlers/admin_launch_readiness.go`、`middleware/tenant.go`、`frontend/src/pages/app/ChatPage.tsx`、`frontend/src/api/client.ts`、`.env.example`、`Dockerfile`、`.github/workflows/`。
3. **模型结构体改动必须同步两处校验**:`internal/models/` 的 `validate` tag 与 `internal/db/schema.go` 的 MongoDB JSON Schema 保持等价,然后 `cd backend && go test ./internal/validation/...`。新增接受用户/API 写入的集合:加 tag、加 schema 函数并入 `AllSchemas()`、加 `validate_test.go` 测试。
4. **系统事件用 `syslog.Logger`**(severity:critical/high/medium/low/debug),勿在 handler/前端塞临时调试输出。
5. **不要 `git add` / `git commit`,除非明确要求。**
6. **依赖项目部署必须用 `Dockerfile.saas` + `fly.saas.toml`**,绝不裸跑 `fly deploy`(否则 auth 路由缺失、登录静默失败)。本仓库本体用根目录 `Dockerfile`。

## 每次开发完成后要更新的文档

- 改了架构/模块/数据流 → `docs/ARCHITECTURE.md`
- 改了业务规则/权限/计费/订单 → `docs/BUSINESS_RULES.md`
- 做了不该反复推翻的决策 → `docs/DECISIONS.md`

---
> Source: [Givemifi/AgentStore](https://github.com/Givemifi/AgentStore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
