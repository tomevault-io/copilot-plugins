## cloudguard-monitor

> 轻量级云服务器监控套件，为个人开发者和小型团队提供开箱即用的服务器监控方案。

# CloudGuard Monitor

## 项目概述
轻量级云服务器监控套件，为个人开发者和小型团队提供开箱即用的服务器监控方案。
Agent-Server 合一架构，单进程部署在被监控服务器上，通过 REST API + WebSocket 对外提供数据。

## 仓库结构
- `server/` — Go 服务端（Agent + API），module: `github.com/trekmax/cloudguard-monitor`
- `cli/` — Go CLI 客户端（待开发）
- `android/` — Android App（待开发）
- `docs/` — 项目文档（PRD、技术评估、开发计划）

## 代码规范
- 错误处理：使用 `fmt.Errorf` + `%w` 包装
- 日志：使用 `slog`，必须携带结构化字段
- 测试：表驱动测试，覆盖率 ≥80%
- 包命名：小写单词，不使用下划线
- Conventional Commits 提交规范

## 常用命令
```bash
cd server
make build    # 编译到 bin/cloudguard
make test     # 运行测试（-race -v）
make lint     # fmt + vet
make run      # 编译并运行
make coverage # 生成覆盖率报告
```

## API 规范
- 统一响应格式: `{code, message, data, timestamp}`
- 认证: `Authorization: Bearer <token>`
- 版本: `/api/v1/`

## 当前进度
### M1: 基础架构
- [x] Sprint 1: 项目初始化（结构、配置、日志、Makefile）
- [x] Sprint 2: 基础采集器（CPU、内存、调度器）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TrekMax) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
