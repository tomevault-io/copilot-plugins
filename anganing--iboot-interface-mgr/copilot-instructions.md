## iboot-interface-mgr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

ApiWeave 是一个配置驱动的接口集成平台，包含三个子项目：

- **apiweave-server** — Spring Boot 3.5 / Java 21 后端，提供接口注册、转换、路由和调用的核心能力
- **apiweave-web** — 基于 Vben Admin Pro (Vue 3 + Vite + Ant Design Vue Next) 的 Monorepo 管理前端
- **soap-test-server** — 用于测试 SOAP 连接器的独立 Spring Boot Web Service

## 常用命令

### 后端 (apiweave-server)

```bash
# 启动（默认 local profile，端口 5320，context-path /api）
cd apiweave-server && mvn spring-boot:run

# 运行测试
cd apiweave-server && mvn test

# 打包
cd apiweave-server && mvn package -DskipTests
```

### 前端 (apiweave-web)

```bash
cd apiweave-web

# 安装依赖（需要 pnpm >= 11）
pnpm install

# 启动开发服务器（默认代理到 localhost:5320/api）
pnpm dev:apiweave

# 构建
pnpm build:apiweave

# 类型检查
pnpm check:type

# Lint
pnpm lint

# 格式化
pnpm format

# 单元测试
pnpm test:unit

# E2E 测试
pnpm test:e2e
```

### SOAP 测试服务器

```bash
cd soap-test-server && mvn spring-boot:run
```

## 架构

### 后端分层

```
com.iboot.apiweave
├── admin/       — 管理后台：App/API/Grant/API-Key CRUD，BeanSearcher 动态查询
├── auth/        — Sa-Token 认证：admin 登录 + API-Key 鉴权
├── common/      — 统一响应 ApiResponse、BusinessException、ErrorCode 枚举
├── config/      — Web 拦截器（SaInterceptor 权限控制）、MyBatis-Plus 配置
├── invoke/      — 核心调用链：InvocationService 编排 → ConnectorExecutor 执行
├── system/      — /system/info、/system/echo 公开端点
└── transform/   — 请求/响应转换：PASSTHROUGH / MAPPING / TEMPLATE / JAVASCRIPT
```

### 请求处理流程

```
调用方 → /open/{appCode}/{apiCode}
  → PublicInvocationController（API-Key 校验 + 授权检查）
    → InvocationService.invoke()
      1. requestTransform（按配置转换请求载荷）
      2. ConnectorExecutor（HTTP / JDBC / SOAP 其中之一）
      3. responseTransform（按配置转换响应载荷）
      4. 写入调用日志（按 logPolicy 控制）
    → 返回响应
```

### Connector 接口

三个连接器都实现 `ConnectorExecutor` 接口（`String type()` + `ConnectorResult execute(JsonNode config, JsonNode body)`）：

| Connector | type() | 用途 |
|-----------|--------|------|
| `HttpConnector` | `"HTTP"` | 支持 GET/POST/PUT/PATCH/DELETE，BASIC/BEARER 认证，重试，query params 透传 |
| `JdbcConnector` | `"JDBC"` | 直接执行 SQL，支持 QUERY/UPDATE/INSERT/DELETE，列名风格转换 |
| `SoapConnector` | `"SOAP"` | Spring WebServiceTemplate，SOAP Action，可选 XML→JSON 解析 |

### Transform 模式

| 模式 | 说明 |
|------|------|
| `PASSTHROUGH` | 原样透传 |
| `MAPPING` | JSONPath 字段映射，支持 required/defaultValue |
| `TEMPLATE` | `${$jsonpath}` 模板替换 |
| `JAVASCRIPT` | GraalJS 沙箱执行（超时 30s，虚拟线程隔离） |

### 公开入口

| 端点 | 用途 |
|------|------|
| `POST /open/{appCode}/{apiCode}` | JSON 入站调用 |
| `POST /open/{appCode}/{apiCode}` (Content-Type: text/xml) | SOAP 入站调用 |
| `GET /open/{appCode}/{apiCode}?wsdl` | SOAP WSDL 自动生成 |

### API 入站协议

- **HTTP_JSON** — 标准 JSON body 入站
- **SOAP_11 / SOAP_12** — SOAP 信封入站，通过 XPath 映射提取字段，支持 BASE64、XML_TO_JSON 等处理步骤

### 鉴权机制

- **Admin 登录**：Sa-Token（Bearer token，默认 30 分钟过期），`/admin/**` 路径强制要求 admin 角色
- **API-Key**：调用方在 Header 或 query param 中传 `apikey`，由 `SaApiKeyDataLoaderImpl` 校验，无需 session token
- **公开路由**：`/auth/login`、`/open/**`、`/system/**`、`/actuator/**` 不经过 Admin 拦截器

### 数据库

- **主库**：PostgreSQL（jdbc:postgresql://127.0.0.1:5432/apiweave），通过 Flyway 管理迁移
- **连接器目标**：PostgreSQL / MySQL / SQL Server / Oracle（JDBC Connector 支持）
- **核心表**：`aw_app`、`aw_api`、`aw_api_grant`、`aw_api_key`、`aw_user`、`aw_call_log`
- **重要**：早期版本有 `aw_api_version` 表，但 V5–V9 迁移已逐步将连接器配置直接写入 `aw_api`，不再使用版本分离模型

### 前端结构

```
apiweave-web/
├── apps/
│   ├── web-antdv-next/     — 主应用（Ant Design Vue Next 变体）
│   ├── web-antd/           — Ant Design Vue 变体
│   ├── web-ele/            — Element Plus 变体
│   ├── web-naive/          — Naive UI 变体
│   ├── web-tdesign/        — TDesign 变体
│   └── backend-mock/       — Nitro 本地 mock 服务器
├── packages/               — 共享包（@vben/* 命名空间）
│   ├── @core/              — 核心基础（base, composables, preferences, ui-kit）
│   ├── effects/            — 功能模块（access, hooks, layouts, plugins, request）
│   ├── stores/             — Pinia 状态管理
│   ├── utils/              — 工具函数
│   └── types/              — TypeScript 类型
└── internal/               — 内部工具（lint 配置、tsconfig、vite-config、tailwind-config）
```

### Vite 开发代理

前端开发服务器将 `/api` 请求代理到 `http://localhost:5320/api`（见 `apps/web-antdv-next/vite.config.ts`）。

### 响应格式

所有 API 响应遵循统一格式：
```json
{ "code": 0, "message": "success", "data": {...} }
```
分页响应在 `data` 中额外包含 `total`、`page`、`size` 字段。

---
> Source: [anganing/iboot-interface-mgr](https://github.com/anganing/iboot-interface-mgr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
