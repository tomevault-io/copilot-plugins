## go-dev

> Go 开发环境规范


# Go 开发环境

## 0x01 版本管理

- 系统安装了 gvm（Go Version Manager），当项目所需 Go 版本与系统默认版本不一致时，使用 `gvm` 切换到目标版本
- 用法：`source ~/.gvm/scripts/gvm && gvm use <version>`，版本号从项目 `go.mod` 第一行读取
- **禁止因版本不匹配就跳过编译或测试**，必须先尝试用 gvm 切换

---
> Source: [ZhuoZhuoCrayon/ai-workspace](https://github.com/ZhuoZhuoCrayon/ai-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
