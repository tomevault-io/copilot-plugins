## comprehensive-microservices-webapp

> 本项目是一个基于 **ASP.NET Core** 的微服务架构 Web 应用，将传统单体架构（Monolithic）拆分为多个独立的微服务。

# Comprehensive-Microservices-WebApp

## 项目概述

本项目是一个基于 **ASP.NET Core** 的微服务架构 Web 应用，将传统单体架构（Monolithic）拆分为多个独立的微服务。
每个微服务拥有独立的业务逻辑、数据访问层和数据库。

---

## 整体架构

### 单体架构（重构前参考）

```
UI
└── Services / Business Logic
    ├── Products
    ├── Users
    ├── Orders
    └── Payments
└── Data Access
└── Database（共享单一数据库）
```

### 微服务架构（当前目标）

```
Comprehensive-Microservices-WebApp/
├── Products.Microservice/        # 商品微服务
├── Users.Microservice/           # 用户微服务
├── Orders.Microservice/          # 订单微服务
├── Payments.Microservice/        # 支付微服务
└── (Gateway / Shared 待扩展)
```

---

## 各微服务结构（Clean Architecture）

每个微服务均采用 **Clean Architecture** 四层结构：

```
<ServiceName>.Microservice/
├── UI/                  # Controllers, Program.cs, Swagger
├── Core/                # DTOs, Service Interfaces, Service Implementations
├── Domain/              # Entities, Repository Interfaces
└── Infrastructure/      # DbContext, Repository Implementations, Migrations
```

### 微服务一览

| 微服务 | 职责 | 独立数据库 |
|---|---|---|
| Products.Microservice | 商品的增删改查 | Products Database |
| Users.Microservice | 用户注册、认证、管理 | Users Database |
| Orders.Microservice | 订单创建与管理 | Orders Database |
| Payments.Microservice | 支付处理与记录 | Payments Database |

---

## 技术栈

- **框架**：ASP.NET Core (.NET 8+)
- **架构模式**：Clean Architecture + Microservices
- **数据库**：每个微服务独立数据库（SQL Server / PostgreSQL，待定）
- **ORM**：Entity Framework Core
- **API 文档**：Swagger / OpenAPI
- **容器化**：Docker（待集成）
- **API 网关**：待集成（如 Ocelot / YARP）

---

## 开发规范

- 每个微服务独立运行，互不直接引用
- 层与层之间依赖方向：UI → Core → Domain ← Infrastructure
- 跨服务通信方式（待定）：HTTP / Message Queue（如 RabbitMQ）
- 每个微服务独立配置 `Program.cs`、`appsettings.json`

---

## 当前进度

- [ ] Products.Microservice 基础结构搭建
- [ ] Users.Microservice 基础结构搭建
- [ ] Orders.Microservice 基础结构搭建
- [ ] Payments.Microservice 基础结构搭建
- [ ] Docker Compose 统一编排
- [ ] API Gateway 集成
- [ ] 服务间通信实现

---

## 备注

- 本项目作为学习用途，参考课程：ASP.NET Core Microservices（Web Academy by Harsha Vardhan）
- 遵循 Clean Architecture 原则，保持各层职责清晰

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rick-liyue-huang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rick-liyue-huang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
