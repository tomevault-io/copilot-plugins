## tapd-ai-cli

> 本文件为 AI 编码助手在此仓库中工作时提供指导。

# AGENTS.md

本文件为 AI 编码助手在此仓库中工作时提供指导。

## 项目简介

tapd-ai-cli 是一个 Go CLI 工具，通过 TAPD（腾讯敏捷产品研发平台）Open API 与 TAPD 平台交互。目标用户是 **AI Agent**（Claude Code、GPT-4o 等），而非人类用户。所有输出均针对最小 token 消耗进行优化。

## 技术栈

- Go 1.24+，CLI 框架：`spf13/cobra`
- SDK：`github.com/studyzy/tapd-sdk-go`（独立仓库，仅依赖标准库）
- HTML→Markdown：`github.com/JohannesKaufmann/html-to-markdown/v2`（CLI 层，SDK 返回原始 HTML）
- Markdown→HTML：`github.com/gomarkdown/markdown`（create/update 时将用户输入转为 TAPD 所需的 HTML）
- 认证：`TAPD_ACCESS_TOKEN`（Bearer，推荐）或 `TAPD_API_USER`/`TAPD_API_PASSWORD`（Basic Auth）
- 许可证：Apache 2.0

## 构建与开发命令

```bash
# 构建（版本号从 git tag 注入）
make build          # go build -ldflags "-X ...Version=$(git describe)" -o tapd ./cmd/tapd/
make build-cross    # 交叉编译，需设 GOOS/GOARCH

# 安装到 $GOPATH/bin
make install

# 测试
make test           # go test -race ./...
go test ./internal/cmd -run TestStoryList   # 运行单个测试
make test-integration                       # go test ./... -v -run "TestIntegration" -count=1

# 覆盖率（目标 >= 60%）
make coverage       # test-coverage + go tool cover -func

# 代码格式化与检查
make fmt            # gofmt -w . && goimports -w .
make lint           # go vet ./... && goimports -l .

# 安装开发工具
make tools          # go install goimports

# 清理
make clean
```

## 架构

### 目录结构

```
cmd/tapd/           # 入口 main.go
internal/
  cmd/              # Cobra 命令定义（每个资源一个文件），消费 SDK
    root.go         # 根命令、全局标志、客户端初始化、通用 helpers
    story.go        # 需求 CRUD 命令（典型命令模板）
    bug.go          # 缺陷管理
    task.go         # 任务管理
    launch.go       # 发布评审
    help.go         # --help 输出紧凑参考卡生成逻辑
    markdown.go     # htmlToMarkdown / markdownToHTML 双向转换
    id_expand.go    # 短 ID → TAPD 长 ID 自动展开
    url.go          # 从 TAPD URL 解析条目类型并查询详情
    ...
  config/           # 凭据管理（CLI flags > env > ./.tapd.json > ~/.tapd.json）
  output/           # JSON/Markdown 输出格式化、退出码常量
```

SDK 独立仓库：[github.com/studyzy/tapd-sdk-go](https://github.com/studyzy/tapd-sdk-go)，本地联调可用 `go.work`（已 gitignore）。

### 命令树

```
tapd
├── auth      login
├── workspace list | switch <id> | info
├── story     list | show <id> | create | update <id> | count | todo
├── task      list | show <id> | create | update <id> | count | todo
├── bug       list | show <id> | create | update <id> | count | todo
├── wiki      list | show <id> | create | update
├── iteration list | create | update | count
├── comment   list | add | update | count
├── tcase     list | create | batch-create
├── timesheet list | add | update
├── launch    list | count | create | update <id> | templates | fields | logs
├── workflow  transitions | status-map | last-steps
├── relation  bugs | create
├── url       <tapd-url>
├── skill     init
└── ...       release, attachment, category, custom-field, label, module, user, report 等

全局标志：--workspace-id <id>, --json, --pretty, --no-comments
```

### 核心设计模式

#### 1. 命令执行流程

每条命令遵循统一模式：参数解析 → SDK 方法调用 → HTML→Markdown 转换（如需）→ 格式化输出。

#### 2. 输出格式双模式

- **list 命令**：始终输出紧凑 JSON（`model.ListResponse` 包装，含 items/total/page/has_more）
- **show 命令**：默认输出 YAML frontmatter + Markdown body（`output.PrintMarkdown`），`--json`/`--pretty` 时输出 JSON
- **create/update 命令**：输出 `model.SuccessResponse`（success/id/url）
- **错误**：JSON 输出到 stderr（`output.PrintError`），附退出码 0-4

#### 3. 泛型 list 辅助函数（`listWithFilters[T]`）

所有 `list` 命令通过 `listWithFilters[T]` 支持 `--filter` 标志，将 request struct 的 `ToParams()` 结果与 filter 参数合并，通过 SDK 的 `DoGet` 直接发请求，`ParseList` 解析响应。

#### 4. 短 ID 自动展开（`expandShortID`）

`show`/`update` 命令支持 ≤9 位短号，自动左补零并拼接 "11" + workspaceID 前缀。

#### 5. 描述字段输入（`readDescription`）

create/update 命令支持三种描述输入：`--description` > `--file` > stdin 管道。输入为 Markdown，自动转 HTML 后提交 API。

#### 6. 客户端初始化（`PersistentPreRunE`）

根命令的 `PersistentPreRunE` 统一加载配置、初始化 `apiClient`，`auth login` 和 `skill init` 命令跳过。

### 添加新命令的模式

参考 `story.go` 作为模板：
1. 创建 `internal/cmd/{resource}.go`
2. 定义父命令 `var xxxCmd = &cobra.Command{...}`
3. 定义子命令 `xxxListCmd`/`xxxShowCmd`/`xxxCreateCmd`/`xxxUpdateCmd`
4. 在 `init()` 中绑定 flags、`AddCommand` 到父命令，父命令 `AddCommand` 到 `rootCmd`
5. list 命令使用 `listWithFilters[T]`，show 命令使用 `printDetail`，create/update 使用 `printSuccessResponse`

## 代码规范

- 代码注释和文档使用**中文**；错误消息和日志使用**英文**
- 每个导出的函数、结构体、接口必须有中文文档注释
- 使用 `gofmt`/`goimports` 格式化
- 不使用 `panic` 或 `goto`
- 函数不超过 80 行，文件不超过 800 行，嵌套不超过 4 层
- 错误作为最后一个返回值，必须处理或显式忽略
- `switch` 语句必须有 `default` 分支

## 测试要求

- 命令层测试参数解析和输出格式（使用 `cobra` 的 `ExecuteC` 执行命令）
- 测试文件 `xxx_test.go`，函数 `TestXxx`
- 集成测试以 `TestIntegration` 前缀命名，通过 `make test-integration` 单独运行
- 覆盖率目标 >= 60%

## 参考文档

- 需求规格：`docs/requirement.md`
- CLI 命令契约：`specs/001-mvp-tapd-cli/contracts/cli-commands.md`
- SDK 提取规范：`specs/002-extract-tapd-sdk/spec.md`
- SDK API 合约：`specs/002-extract-tapd-sdk/contracts/sdk-api.md`

---
> Source: [studyzy/tapd-ai-cli](https://github.com/studyzy/tapd-ai-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
