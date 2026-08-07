## mcpx

> MCPX 是运行在开发环境中的 **MCP Runtime（网关）**，Go module 为 `mcpx`，需 **Go 1.26.1+**（以 `go.mod` 为准）。

# 仓库指南

## 项目结构与模块组织

MCPX 是运行在开发环境中的 **MCP Runtime（网关）**，Go module 为 `mcpx`，需 **Go 1.26.1+**（以 `go.mod` 为准）。

| 路径 | 说明 |
|------|------|
| `cmd/mcpx-server/` | 可执行入口（`main`、子命令如 `oauth-register`） |
| `internal/` | 业务包：`server`（HTTP Gateway / 工具注册）、`remotesession`、`workspace`、`auth`/`oauth`、`changeset`、`terminal`、`source`、`artifact`、`config`、`state` 等 |
| `docs/plans/`、`docs/specs/` | 实现计划与设计规格 |
| `bin/` | 本地构建产物（已 gitignore） |
| `~/.mcpx/` | 运行时数据（配置、SQLite、日志、任务），**不在本仓库** |

本地产品文档 `prd/`、`docs/superpowers/` 被 `.gitignore` 排除，勿提交。

## 构建、测试与开发命令

命令均来自 README 与 `.github/workflows/ci.yml`：

```bash
# 编译
go build -o bin/mcpx-server ./cmd/mcpx-server

# 单测（CI 同款）
go test ./... -count=1

# 竞态检测
go test -race ./... -count=1

# 格式检查（仅 cmd + internal）
test -z "$(gofmt -l ./cmd ./internal)"

# 静态检查
go vet ./...

# 发布构建（CGO 关闭，与 CI/GoReleaser 一致）
CGO_ENABLED=0 go build -o bin/mcpx-server ./cmd/mcpx-server
```

本地运行：`./bin/mcpx-server` 或 `./bin/mcpx-server --workspace /path/to/project`。版本：`./bin/mcpx-server -version`。  
发版：推送 `v*` 标签触发 GoReleaser（见 `.github/workflows/release.yml`、`.goreleaser.yaml`）。

## 编码风格与命名

- 使用 `gofmt`；改动保持与现有包风格一致（小写包名、导出类型 PascalCase、YAML 字段 `snake_case` tag）。
- 平台相关文件用 `_unix.go` / `_windows.go` / `_darwin.go` 后缀（见 `changeset`、`environment`、`cmd`）。
- 优先改 `internal/` 对应包；工具面注册与 HTTP 网关在 `internal/server/`。
- 最小充分改动；不要引入未使用的依赖。

## 测试指南

- 框架：Go 标准库 `testing`。
- 命名：`*_test.go`，与被测包同目录（如 `internal/auth/token_test.go`）。
- 改后端逻辑后应补/跑相关包测试；宣称完成前至少对改动包执行 `go test ./path/to/pkg -count=1`，合并前宜跑 `go test ./... -count=1`（与 CI 一致）。
- 单测默认用于本地验证；是否纳入提交由用户当轮决定。

## 提交与 PR

- 历史风格：Conventional Commits，**subject 中文动词开头**，例如：
  - `feat(cli): 增加 oauth-register 子命令`
  - `fix(oauth): 持久化 DCR 客户端`
  - `docs(readme): 补充接入说明`
  - `chore(repo): …` / `ci(release): …`
- **禁止** agent 自动 `commit` / `push`。若建议提交：先展示变更摘要与完整 commit message，**经用户明确确认后再执行**。
- PR：说明动机与行为变化；关联 issue（如有）；涉及鉴权/网关/配置时写清兼容与风险；CI 须绿（test、gofmt、vet、race、build）。

## 安全与配置（要点）

- 进程配置与密钥在 `~/.mcpx/config.yaml`（及环境变量如 `MCPX_HOME`、`MCPX_LOG_LEVEL`），**勿把真实 token/password/secret 写入仓库或文档示例**。
- 鉴权为进程级（`auth.mode`：open / bearer / oauth / dual）；项目级 `.mcpx.yaml` 只覆盖描述与安全策略等，不覆盖全局凭证。
- 命令与文件策略见 `security.commands` / `security.files`；默认偏安全（命令默认 `confirm`）。
- 审计日志与任务日志在 `~/.mcpx/`，权限敏感，勿打包进 release（GoReleaser 仅发二进制）。

---
> Source: [opentokenz/mcpx](https://github.com/opentokenz/mcpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
