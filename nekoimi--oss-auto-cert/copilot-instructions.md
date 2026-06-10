## oss-auto-cert

> 本文档为 AI 编码助手提供 oss-auto-cert 项目的编码规范指南。

# AGENTS.md - oss-auto-cert 编码规范

本文档为 AI 编码助手提供 oss-auto-cert 项目的编码规范指南。

## 项目概述

- **语言**: Go 1.23.0
- **模块**: `github.com/nekoimi/oss-auto-cert`
- **用途**: 基于 Let's Encrypt 实现阿里云 OSS/CDN SSL 证书自动管理

## 构建命令

```bash
# 构建项目
go build -o oss-auto-cert main.go

# 生产环境构建（静态链接）
go build --ldflags "-extldflags -static -s -w" -o oss-auto-cert main.go

# 安装依赖
go mod tidy && go mod download

# 运行应用
./oss-auto-cert -config=/path/to/config.yaml -log-level=info
```

## 测试命令

```bash
# 运行所有测试
go test ./...

# 详细模式运行测试
go test -v ./...

# 运行特定包的测试
go test -v ./pkg/utils/
go test -v ./internal/alioss/

# 运行单个测试函数
go test -v -run TestDateIsExpire ./pkg/utils/
go test -v -run TestBackupIfExists ./pkg/utils/

# 测试覆盖率
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

## 代码风格规范

### 导入规范

- 导入分组：标准库 → 第三方库 → 项目内部包
- 使用 goimports 格式化
- 示例：
```go
import (
    "context"
    "os"
    
    "github.com/charmbracelet/log"
    "github.com/aliyun/aliyun-oss-go-sdk/oss"
    
    "github.com/nekoimi/oss-auto-cert/internal/config"
    "github.com/nekoimi/oss-auto-cert/pkg/utils"
)
```

### 格式化规范

- 使用 `gofmt` 格式化所有 Go 文件
- 使用 tab 缩进（Go 标准）
- 运行 `go vet` 检查问题

### 命名规范

- **包名**: 小写，单字，无下划线
- **导出标识符**: PascalCase（如 `NewAutoCert`, `Config`）
- **非导出标识符**: camelCase（如 `loadOptions`, `expiredEarly`）
- **常量**: MixedCase（如 `DefaultExpiredEarly`）
- **接口**: PascalCase，-er 后缀（如 `Provider`）
- **缩写**: 全大写（如 `OSS`, `CDN`）

### 类型和结构体

- 使用 struct tags 进行 YAML/JSON 序列化
- 导出类型需添加文档注释
- 示例：
```go
// Config 保存应用配置
type Config struct {
    Path    string   `yaml:"-"`        // 配置文件路径
    Webhook string   `yaml:"webhook"`  // 通知 Webhook URL
    Buckets []Bucket `yaml:"buckets"`  // OSS Bucket 配置
}
```

### 错误处理

- 返回错误而不是记录后继续执行
- 使用 `fmt.Errorf()` 添加上下文
- 使用 charmbracelet/log 记录错误
- 显式处理错误，避免使用 `_` 忽略
- 示例：
```go
if err != nil {
    log.Errorf("加载配置失败: %s", err.Error())
    return nil, err
}
```

### 日志规范

- 使用 `github.com/charmbracelet/log` 记录日志
- 日志级别: Debug, Info, Warn, Error, Fatal
- 使用结构化日志: `log.Debugf()`, `log.Infof()`
- 日志和注释可使用中文（项目约定）

### 上下文使用

- 将 `context.Context` 作为函数第一个参数
- 使用 context 处理取消和超时
- 示例：
```go
func (c *AutoCert) Run(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
        // 处理逻辑
    }
}
```

### 并发规范

- 简单标志使用 `atomic.Bool`
- goroutine 间使用 channel 通信
- 始终处理 context 取消信号
- 示例：
```go
type Service struct {
    running atomic.Bool
    msgCh   chan string
}
```

### 包组织结构

- `main.go`: 入口点
- `internal/`: 内部实现
  - `config/`: 配置管理
  - `cert/`: 证书自动化逻辑
  - `alioss/`: 阿里云 OSS/CDN/CAS 服务
  - `acme/`: ACME/Let's Encrypt 集成
  - `types/`: 共享类型定义
- `pkg/`: 公共包
  - `utils/`: 工具函数
  - `webhook/`: Webhook 通知

## 测试规范

- 测试文件: `*_test.go`
- 使用标准 `testing` 包
- 测试函数命名: `TestFunctionName`
- 使用 `t.Log()` 记录测试日志
- 显式检查错误
- 示例：
```go
func TestDateIsExpire(t *testing.T) {
    result := DateIsExpire("2024-10-13", time.Hour*24*30)
    t.Logf("result: %v", result)
}
```

## Docker 命令

```bash
# 构建镜像
docker build -t oss-auto-cert:latest .

# 运行容器
docker run -d --rm \
  -v $PWD/config.yaml:/etc/oss-auto-cert/config.yaml \
  -e OSS_ACCESS_KEY_ID=xxx \
  -e OSS_ACCESS_KEY_SECRET=xxx \
  oss-auto-cert:latest

# Docker Compose
docker-compose up -d
```

## 环境变量

- `OSS_ACCESS_KEY_ID`: 阿里云 Access Key ID（必填）
- `OSS_ACCESS_KEY_SECRET`: 阿里云 Access Key Secret（必填）
- `ACME_EMAIL`: 证书注册邮箱
- `ACME_DATA_DIR`: 证书存储目录
- `ACME_EXPIRED_EARLY`: 提前续期天数
- `DEBUG`: 调试模式，设为 "true" 开启

## 安全规范

- 切勿记录或暴露凭证
- 敏感数据使用环境变量
- 验证所有外部输入
- 优雅处理错误

## CI/CD

GitHub Actions 工作流位于 `.github/workflows/release-docker.yml`：
- 版本标签（v*）触发
- 构建多架构镜像（linux/amd64, linux/arm64）
- 发布到 Docker Hub、GHCR 和阿里云镜像仓库

## 其他说明

- 项目对用户可见的消息和注释使用中文
- 遵循代码库现有模式
- 保持函数专注且精简
- 导出函数需添加文档注释
- 使用依赖注入提高可测试性

---
> Source: [nekoimi/oss-auto-cert](https://github.com/nekoimi/oss-auto-cert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
