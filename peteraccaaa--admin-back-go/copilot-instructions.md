## admin-back-go

> 这是 `E:\admin_go` 的 Go 主后端。

# AGENTS.md for admin_back_go

## 当前定位

这是 `E:\admin_go` 的 Go 主后端。

主架构：

```text
Gin modular monolith
route -> handler -> service -> repository -> model
```

## 必读

开始任何 Go 后端任务前，先读：

```text
E:\admin_go\AGENTS.md
E:\admin_go\docs\architecture\04-go-backend-framework.md
E:\admin_go\docs\architecture\05-development-quality-rules.md
docs\architecture.md
internal\module\README.md
internal\platform\README.md
```

## 禁止

```text
禁止写成 Java 风格
禁止无意义 interface
禁止 ServiceImpl
禁止 Manager/Factory 滥用
禁止 handler 直接查 DB/Redis
禁止 service 依赖 gin.Context
禁止在 RBAC 契约未固定前实现权限业务
禁止写兜底字段、猜字段、吞未知 DTO
禁止新接口继续全 POST 动作 URL
```

## 当前阶段

当前已经进入 Auth + RBAC core 的最小迁移期。只允许迁移登录态、Users/init、菜单/权限、角色、操作日志这些核心闭环；业务模块继续按阶段迁移。

---
> Source: [PeterACCaaa/admin_back_go](https://github.com/PeterACCaaa/admin_back_go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
