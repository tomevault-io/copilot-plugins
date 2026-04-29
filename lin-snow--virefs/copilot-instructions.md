## virefs

> 本文件为 AI 编码代理（Cursor、Copilot、Cline 等）提供项目上下文。人类开发者请阅读 [README.md](README.md) 和 [CONTRIBUTING.md](CONTRIBUTING.md)。

# AGENTS.md — AI Agent 指南

本文件为 AI 编码代理（Cursor、Copilot、Cline 等）提供项目上下文。人类开发者请阅读 [README.md](README.md) 和 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 项目概述

VireFS 是一个 Go 文件系统抽象库，将本地文件系统和 S3 兼容对象存储统一到 `virefs.FS` 接口之下。包名为 `virefs`，模块路径为 `github.com/lin-snow/VireFS`。

## 代码结构

```
virefs (根包)
├── fs.go            # 核心接口：FS、Copier、Presigner、AccessInfo、OpError、哨兵错误
├── key.go           # CleanKey：key 规范化与路径穿越防护
├── localfs.go       # LocalFS 实现（本地磁盘后端）
├── objectfs.go      # ObjectFS 实现（S3/MinIO/R2 后端）
├── s3config.go      # S3Config、Provider、NewS3Client、NewObjectFSFromConfig
├── mount.go         # MountTable：多后端按前缀路由
├── schema.go        # Schema：声明式 key 路由（按扩展名/自定义函数）
├── migrate.go       # Migrate：批量迁移（Walk + Copy + 冲突策略）
├── hook.go          # WithHooks：操作拦截装饰器
├── walk.go          # Walk：递归遍历
├── *_test.go        # 对应的测试文件
└── plugin/
    └── zip/         # zip.FS 插件（只读 FS，基于 zip 归档）
        ├── zipfs.go
        ├── pack.go
        ├── unpack.go
        └── zip_test.go
```

## 核心设计模式

### 接口层次

- `FS` 是所有后端必须实现的最小接口（Get/Put/Delete/List/Stat/Access/Exists）
- `Copier` 和 `Presigner` 是可选能力接口，通过类型断言使用
- `KeyFunc` 和 `AccessFunc` 是函数类型，用于扩展 key 变换和 URL 生成
- `Middleware` 是 `func(FS) FS`，通过 `Chain` 组合多层中间件
- `BaseFS` 是嵌入辅助结构体，编写中间件时只需覆盖需要拦截的方法

### key 处理流水线

所有操作的 key 都经过：`CleanKey`（规范化）→ `KeyFunc`（可选变换）→ 拼接前缀 → 存储操作。`CleanKey` 会拒绝 `..` 路径穿越。

### 错误处理

- 使用 `fs.go` 中的五个哨兵错误：`ErrNotFound`、`ErrInvalidKey`、`ErrAlreadyExist`、`ErrNotSupported`、`ErrPermission`
- 后端实现必须将错误包装为 `*OpError{Op, Key, Err}`
- 始终使用 `errors.Is()` 检查哨兵错误

### 选项模式

- `LocalOption` / `ObjectOption` 配置后端实例（functional options 模式）
- `PutOption` 配置写入操作，通过 `BuildPutConfig` 聚合

## 编码规范

- 使用 `gofmt` 格式化，`go vet` 静态检查
- 导出的类型和函数必须有 GoDoc 注释
- 测试使用表驱动风格（table-driven tests）
- ObjectFS 通过接口 `S3API` / `PresignAPI` 解耦 AWS SDK，测试中用 mock 实现
- LocalFS 测试使用 `t.TempDir()` 创建临时目录

## 添加新后端

1. 在根包中创建新文件（如 `newbackend.go`）
2. 实现 `FS` 接口的全部七个方法
3. key 处理必须调用 `CleanKey` 并应用 `KeyFunc`（如支持）
4. 错误必须包装为 `*OpError`
5. 如支持高效复制，额外实现 `Copier`
6. 创建对应的 `*_test.go`

## 添加新插件

插件放在 `plugin/<name>/` 子目录中，使用独立的包名，通过 `import virefs "github.com/lin-snow/VireFS"` 引用核心类型。参考 `plugin/zip/` 的结构。

## 测试

```bash
go test -race ./...          # 运行全部测试（含竞态检测）
go test -race -run TestXxx   # 运行单个测试
go test -race ./plugin/zip/  # 运行插件测试
```

ObjectFS 测试使用 mock S3 客户端，无需真实 S3 服务。

## CI/CD

- Push 到 main 或 PR 会触发 `.github/workflows/ci.yml`（格式检查、vet、竞态测试）
- 推送 `v*` tag 会触发 `.github/workflows/release.yml`（测试门禁 + 自动创建 GitHub Release）

## 注意事项

- 仅依赖 AWS SDK（aws-sdk-go-v2 核心、s3、config、credentials），不要引入其他外部依赖，除非有充分理由
- `CleanKey` 是安全边界，不要绕过它
- LocalFS 的 `WithAtomicWrite()` 使用 temp + rename 模式，修改时注意跨文件系统兼容性
- MountTable 的 prefix 必须是单段路径（不含 `/`）
- Schema 路由按声明顺序匹配，第一个命中生效

---
> Source: [lin-snow/VireFS](https://github.com/lin-snow/VireFS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
