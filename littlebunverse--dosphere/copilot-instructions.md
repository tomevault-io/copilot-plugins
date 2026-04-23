## dosphere

> > 说明：本文件为**项目级**规则，会与全局 `~/.codex/AGENTS.md` **叠加**生效；如有冲突，优先遵循更具体、更贴近当前目录/代码的规则。

# Codex 项目级提示词（web-project）



> 说明：本文件为**项目级**规则，会与全局 `~/.codex/AGENTS.md` **叠加**生效；如有冲突，优先遵循更具体、更贴近当前目录/代码的规则。



## 前端（React）组件与动效规范（强制）

- 实现功能时，UI 组件/区块优先使用 `shadcn-studio-mcp` 查询并复用；找不到再使用 `kibo-ui-mcp`  寻找相近组件。
- 动画/动效方案优先参考 `https://ssgoi.dev/zh/docs/getting-started/introduction`，在满足效果的前提下尽量减少新增依赖与包体积。

---

# Repository Guidelines

## 项目结构与模块组织
- 根目录包含两个子项目：`web/`（React 前端）与 `server/`（go 后端）。


## 构建、测试与开发命令
- 前端：`cd "web"` → `pnpm install`、`pnpm dev`（本地开发）、`pnpm build`（生产构建）、`pnpm lint`（ESLint）。
- 后端：`cd "server"` → `go run ./cmd/server`、`go test ./...`、`go vet ./...`。

## 编码风格与命名规范
- TypeScript/TSX 与 Java 统一 4 空格缩进，遵循现有文件风格。
- 前端组件用 PascalCase（如 `TaskList`），Hook 以 `use` 开头；后端类以职责命名（如 `ListController`、`TodoListService`）。
- 关键业务逻辑添加中文注释，避免遗留无用代码与 TODO。

## 测试指南
- 后端以 Go 单元测试为主，测试文件以 `*_test.go` 结尾；新增测试优先放在 `server/` 对应包目录。
- 单测执行时间建议控制在 60s 内；前端当前未配置测试框架，若新增需同步补充脚本与文档。

## 提交与 Pull Request 指南
- Git 历史提交信息为简短英文描述，建议继续使用清晰的动词短语（如 `Add list validation`）。
- PR 需说明变更范围、关联问题、运行过的测试命令；UI 改动请附截图，涉及数据库变更需说明迁移步骤。

## 配置与安全提示
- 前后端通过 `/api` 代理交互，调试时确保后端服务已启动。
- 授权请求使用 `Authorization: Bearer <token>`，敏感信息不要提交到仓库。

---
> Source: [LittleBunVerse/Dosphere](https://github.com/LittleBunVerse/Dosphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
