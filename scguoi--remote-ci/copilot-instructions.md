## remote-ci

> make install-tools      # 安装所有语言工具

# 开发环境设置指南

## 一键设置

使用以下命令一键设置完整的开发环境：
```bash
make dev-setup
```

这个命令会自动：
- 安装所有代码格式化和检查工具
- 配置Git钩子
- 设置分支管理策略

## 工具安装

### 手动安装工具
```bash
make install-tools      # 安装所有语言工具
make install-tools-go   # 仅安装Go工具
make install-tools-ts   # 仅安装TypeScript工具
make install-tools-java # 仅安装Java工具
make install-tools-python # 仅安装Python工具
```

### 检查工具状态
```bash
make check-tools        # 检查所有工具
make check-tools-go     # 检查Go工具
make check-tools-ts     # 检查TypeScript工具
make check-tools-java   # 检查Java工具
make check-tools-python # 检查Python工具
```

## 环境要求

### Go环境
- Go 1.24.4 或更高版本
- 工具：goimports, gofumpt, golines, gocyclo, staticcheck, golangci-lint

### TypeScript环境
- Node.js
- npm
- 工具：prettier, eslint, typescript

### Java环境
- Java JDK
- Maven

### Python环境
- Python 3
- 工具：black, flake8, mypy

## 验证设置

运行以下命令验证环境设置：
```bash
make project-status    # 显示检测到的项目状态
make info             # 显示项目信息
make help             # 显示所有可用命令
```

## 故障排除

如果工具安装失败：
1. 检查网络连接
2. 确保有足够的权限
3. 运行 `make check-tools` 查看具体错误
4. 手动安装缺失的工具
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scguoi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
