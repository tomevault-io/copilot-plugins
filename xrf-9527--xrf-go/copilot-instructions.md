## xrf-go

> Project memory file for XRF-Go v1.0.0-RC4 - provides specific development guidance and conventions for Claude Code.

# CLAUDE.md

Project memory file for XRF-Go v1.0.0-RC4 - provides specific development guidance and conventions for Claude Code.

## Project Overview

XRF-Go is a production-ready Xray installation and configuration tool with 100% DESIGN.md alignment. Core design: "High efficiency, ultra-fast, extremely easy to use" with multi-configuration concurrent operation achieving ~0.12ms protocol addition speed.

## 权限处理机制 (Permission Handling)

### 智能权限检测 (基于Docker模式最佳实践)
- **安装脚本**: 支持root用户直接运行和普通用户sudo权限运行
- **Go程序**: 要求root权限执行系统级操作 (符合系统工具标准)
- **权限适配**: 动态检测用户身份，自动设置命令前缀 ($SUDO_CMD)
- **错误处理**: 提供清晰的权限获取指导和解决方案

### 权限要求原因
- 绑定特权端口 (80, 443)
- 写入系统目录 (/usr/local/bin, /etc/xray)
- 管理systemd服务 (启动、停止、重载)
- 配置系统优化 (BBR, 文件描述符限制)

## Code Architecture Requirements

### Module Structure (Enforce Strictly)
- `cmd/xrf/` - Single main CLI entry point
- `pkg/config/` - Use ConfigManager for all configuration operations
- `pkg/system/` - System detection via detector.go, service management via service.go
- `pkg/tls/` - ACME operations via acme.go, Caddy management via caddy.go
- `pkg/api/` - Xray gRPC client integration for runtime operations
- `pkg/utils/` - All utilities must use existing modules (crypto, validator, logger, etc.)

### Configuration File Conventions (Must Follow)
- Use `/etc/xray/confs/` directory only
- File naming: numeric prefix (00-09: base, 10-19: inbound, 20-29: outbound, 90-99: routing)
- Each protocol = one dedicated JSON file
- Always validate with `xray test -confdir` before writing
- Implement rollback mechanism for failed operations

## Development Commands (Use These Exact Commands)

### Building
- Production build: `go build -ldflags="-s -w" -o xrf cmd/xrf/main.go`
- Cross-compile Linux AMD64: `GOOS=linux GOARCH=amd64 go build -o xrf-linux-amd64 cmd/xrf/main.go`
- Cross-compile Linux ARM64: `GOOS=linux GOARCH=arm64 go build -o xrf-linux-arm64 cmd/xrf/main.go`
- 支持的架构: amd64 (x86_64) 和 arm64 (aarch64) 服务器架构

### Testing
- Run all tests: `go test ./...`
- TLS module tests: `go test -cover ./pkg/tls/`
- Configuration validation: `xray test -confdir /etc/xray/confs`

### Code Quality (Run Before Every Commit)
- Format: `go fmt ./...`
- Vet: `go vet ./...`
- Lint: `golangci-lint run`
- Unit Tests: `go test ./...`
- Security: `go mod audit`

### Git and GitHub Workflow (PRIORITY: Use gh CLI)
- **GitHub Operations**: ALWAYS use `gh` CLI for pull requests, issues, releases
- **Pre-Commit Checklist**: Run in exact order before ANY commit:
  ```bash
  go fmt ./...           # Format code
  go vet ./...          # Static analysis
  golangci-lint run     # Advanced linting
  go test ./...         # Run unit tests
  ```
- **Pull Requests**: `gh pr create`, `gh pr list`, `gh pr merge`
- **Issues**: `gh issue create`, `gh issue list`, `gh issue close`
- All quality checks must pass with 0 errors/warnings before proceeding

## Protocol Implementation Rules

### Supported Protocols (Use Exact Aliases)
- `vr` → VLESS-REALITY with xtls-rprx-vision flow
- `vw` → VLESS-WebSocket-TLS
- `vmess` → VMess-WebSocket-TLS
- `hu` → VLESS-HTTPUpgrade
- `tw` → Trojan-WebSocket-TLS
- `ss` → Shadowsocks
- `ss2022` → Shadowsocks-2022

### Configuration Templates
- All templates embedded in `pkg/config/templates.go`
- Use Go string constants only
- Include performance optimizations: `tcpKeepAliveIdle: 300`, `tcpUserTimeout: 10000`
- Apply BBR congestion control settings automatically

### REALITY Protocol Parameter Rules (CRITICAL - DO NOT CONFUSE)
- **`dest`** = 目标地址 (如: www.microsoft.com:443) - NEVER rename to `sni`
- **`serverNames`** = SNI服务器名列表 (如: ["www.microsoft.com"]) - 服务端使用数组
- **`serverName`** = 单个SNI服务器名 (如: "www.microsoft.com") - 客户端使用字符串
- **CLI参数**: `--dest` 用于指定目标地址，不是SNI参数
- **配置结构**: `dest`和`serverNames`是两个独立功能的参数，都必须正确配置

## CLI Command Implementation

### Core Operations (Must Implement Error Handling)
- `xrf install` - Use pkg/system/installer.go with GitHub API integration
- `xrf add [protocol]` - Target 10-50ms with full safety checks, use ConfigManager.AddProtocol
- `xrf list` - Display protocols with color coding via utils/colors.go
- `xrf remove [tag]` - Implement with automatic backup and rollback
- `xrf info [tag]` - Show detailed config via structured output
- `xrf change [tag] [key] [value]` - Validate before change, rollback on failure

### Service Management (Use systemd Integration)
- All service commands must use pkg/system/service.go
- `xrf reload` - Send USR1 signal to Xray process
- `xrf check-port` - Use pkg/utils/network.go port validation

### TLS Automation (ACME Integration Required)
- Use pkg/tls/acme.go for Let's Encrypt operations
- Use pkg/tls/caddy.go for reverse proxy setup
- Implement 30-day auto-renewal mechanism

### Utility Commands
- `xrf generate [type]` - Use pkg/utils/crypto.go functions
- `xrf ip` - Use pkg/utils/http.go GetPublicIP function
- `xrf logs` - Integrate systemd journal reading

## Performance Requirements (Must Meet)

### Speed Targets (Updated Based on Real Testing)
- Protocol addition: 10-50ms (production with backup+validation)
- Unit tests: <10ms (skip validation)
- TLS protocols: +50-100ms (certificate generation overhead)
- Memory usage: <20MB
- Binary size: <10MB
- Configuration ops: >40/sec (realistic with full safety checks)

### Error Handling Strategy
- Use pkg/utils/errors.go for consistent error types
- Implement createAutoBackup() before any config change
- Call restoreFromBackup() on operation failure
- Validate with `xray test -confdir` after every change
- Provide specific fix suggestions in error messages

### Security Implementation
- systemd hardening: NoNewPrivileges=true, ProtectSystem=strict
- Port conflict detection via pkg/utils/network.go
- Certificate validation via pkg/utils/crypto.go
- BBR setup via system optimization commands

## Coding Standards (Enforce Strictly)

### Module Usage Rules
- Use existing ConfigManager methods, never direct file operations
- Use pkg/utils/logger.go for all logging with color support
- Use pkg/utils/validator.go for all input validation
- Use pkg/utils/crypto.go for UUID, password, key generation
- Use pkg/system/detector.go for OS detection before installation
- Use pkg/config/testutils.go for test certificate generation in test mode

### Error Handling Pattern
```go
// Always implement this pattern
if err := createAutoBackup(); err != nil {
    return fmt.Errorf("backup failed: %v", err)
}
if err := operation(); err != nil {
    restoreFromBackup()
    return fmt.Errorf("operation failed: %v", err)
}
if err := validateConfigAfterChange(); err != nil {
    restoreFromBackup()
    return fmt.Errorf("validation failed: %v", err)
}
```

### Testing Requirements
- Write unit tests for new functions
- Test with real Xray binary validation
- Benchmark performance-critical functions
- Test rollback mechanisms

### Testing Best Practices (Based on System Module Implementation)
- **Test Environment Detection**: Use `config.IsTestEnvironment()` to auto-detect test mode (unified approach)
- **Certificate Management**: Generate test certificates dynamically using `GetTestCertificate()`
- **Test Isolation**: Each test uses independent temporary directories
- **Performance Targets**: Set realistic goals based on actual operations:
  - Unit tests (pure logic): <10ms
  - Integration tests (system calls): 100-400ms
  - TLS protocols (cert generation): +50-100ms extra
  - Concurrent operations: 500-1000ms (multiple parallel operations)
- **Error Handling in Tests**: Always check return values with proper Go practices:
  ```go
  if err := configMgr.Initialize(); err != nil {
      t.Fatalf("Initialize failed: %v", err)  // For test setup
  }
  if err := someOperation(); err != nil {
      b.Errorf("Operation failed: %v", err)   // For benchmark non-fatal
  }
  ```
- **Test Architecture**: Implement 3-layer testing (IMPLEMENTED):
  1. Unit tests (`*_unit_test.go`) - Pure logic, skip I/O, <10ms
  2. Integration tests (`*_integration_test.go`) - Full flow with validation, 100-400ms
  3. Main tests (`*_test.go`) - Mixed functional tests, realistic timeouts
- **Parallel Testing**: All tests use `t.Parallel()` for concurrent execution
- **Version Consistency**: Always fetch latest Xray version (no hardcoded fallback)
- **Environment Variables**: 
  - `config.IsTestEnvironment()` detects go test automatically
  - `XRF_TEST_MODE=1` - Legacy support (deprecated)

## Dependencies and Constraints

### Runtime Requirements
- Xray binary: Auto-download from GitHub releases via pkg/system/installer.go
- System: Ubuntu/CentOS/Debian with systemd support (仅支持 amd64/arm64 架构)
- Network: Port availability for protocols (use smart allocation)
- Permissions: 支持root用户直接运行或普通用户sudo权限运行

### Build Requirements
- Go 1.19+ required
- No external dependencies beyond standard library
- Embed all templates in binary (no external files)

## Current Development Focus

### Completed (Production Ready)
- ✅ All 25+ CLI commands implemented
- ✅ DESIGN.md 100% alignment achieved
- ✅ Performance targets realistic and achievable (10-50ms with full safety)
- ✅ Full system management infrastructure
- ✅ TLS automation with ACME + Caddy integration

### Quality Improvement Priorities
- ✅ TLS test certificate auto-generation implemented
- ✅ Test architecture restructured (unit/integration separation) 
- ✅ Performance targets adjusted to realistic values
- ✅ Go lint compliance achieved (errcheck, gofmt)
- ✅ Root用户权限处理矛盾修复 - 采用Docker模式智能权限检测
- ✅ 架构支持精简 - 专注主流服务器架构 (amd64/arm64)
- Error message improvements
- Input validation enhancement
- Documentation updates

### Code Quality Compliance (Enforce Before Commit)
- **Pre-Commit Pipeline**: Execute commands in exact order:
  1. `go fmt ./...` (must not modify any files)
  2. `go vet ./...` (must pass with 0 warnings)
  3. `golangci-lint run` (must pass with 0 issues)
  4. `go test ./...` (must pass all tests)
- **GitHub Operations**: Use `gh` CLI for all GitHub tasks (issues, PRs, releases)
- **Error Handling**: All error return values must be checked (errcheck)
- **Test Standards**: Follow Go testing best practices:
  - Use `t.Fatal()` for setup failures that prevent test execution
  - Use `t.Error()` followed by `continue` for non-fatal test failures
  - Use `b.Error()` for non-fatal benchmark failures
  - Always provide context in error messages
- **Mod Tidy**: `go mod tidy` must not make changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xrf-9527) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
