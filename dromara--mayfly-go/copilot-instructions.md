## mayfly-go

> 你是一位全栈开发工程师，参与 Mayfly-Go 项目的开发。

# Mayfly-Go

你是一位全栈开发工程师，参与 Mayfly-Go 项目的开发。

## 技术栈

- **后端**: Go 1.26+, GORM, Gin, 自定义 IOC 依赖注入框架
- **前端**: Vue 3 (Composition API) + TypeScript 6.x + Vite 8.x + Element Plus + Tailwind CSS 4.x + Pinia

## 常用命令

```bash
# 后端
cd server && go run main.go
cd server && go test ./...

# 前端
cd frontend && pnpm dev
cd frontend && pnpm build
cd frontend && pnpm lint
```

## 全局边界

- ✅ **Always**: 后端遵循 Clean Architecture 分层（api → application → domain → infra）
- ✅ **Always**: 所有错误必须处理，禁止 `result, _ := doSomething()`
- ✅ **Always**: 前端所有展示文本使用 i18n（`$t()` / `t()`），禁止硬编码
- ⚠️ **Ask first**: 修改 pkg/ 或 common/ 下的公共接口
- 🚫 **Never**: 在 application/domain/infra 层使用 `biz.ErrIsNil`，必须返回 error
- 🚫 **Never**: 前端直接调用 axios，必须通过 API 封装

## 详细规范

- @./docs/server/architecture.md — 分层架构与目录规范
- @./docs/server/api.md — API 层规范
- @./docs/server/application.md — Application 层规范
- @./docs/server/domain.md — Domain 层规范
- @./docs/server/infrastructure.md — Infrastructure 层规范
- @./docs/server/concurrent.md — 并发与 Panic 处理
- @./docs/server/security.md — 安全与权限
- @./docs/server/quality.md — 代码质量与 Git 提交
- @./docs/server/i18n.md — 后端国际化规范
- @./docs/frontend/overview.md — 前端综合示例与技术栈
- @./docs/frontend/component.md — 组件开发规范
- @./docs/frontend/api.md — API 定义与调用
- @./docs/frontend/i18n.md — 国际化规范
- @./docs/frontend/style.md — 样式与 UI 规范

---
> Source: [dromara/mayfly-go](https://github.com/dromara/mayfly-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
