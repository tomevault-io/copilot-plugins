## go

> Go 安全，扩展通用规则

# Go 安全

> 本文件以 Go 特定内容扩展通用安全规则。

## 密钥管理

```go
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY 未配置")
}
```

## 安全扫描

- 使用 **gosec** 进行静态安全分析：
  ```bash
  gosec ./...
  ```

## Context 与超时

始终使用 `context.Context` 进行超时控制：

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```

---
> Source: [nongjun/feishu-cursor-claw](https://github.com/nongjun/feishu-cursor-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
