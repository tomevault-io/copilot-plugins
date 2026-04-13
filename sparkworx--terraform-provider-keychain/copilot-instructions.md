## terraform-provider-keychain

> A Terraform provider for managing items in macOS Keychains. This provider allows infrastructure-as-code management of secrets, certificates, keys, and identities stored in any user-accessible macOS Keychain.

# Terraform Provider: macOS Keychain

## Project Overview

A Terraform provider for managing items in macOS Keychains. This provider allows infrastructure-as-code management of secrets, certificates, keys, and identities stored in any user-accessible macOS Keychain.

## Goals

- Full read/write access to any named macOS Keychain accessible by the logged-in user
- Support for all keychain item classes supported by macOS Security.framework
- No keychain creation functionality (keychains must pre-exist)
- Secure handling of sensitive data with proper Terraform state considerations

## Technology Stack

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Language | Go 1.21+ | Required for Terraform Plugin Framework |
| Terraform SDK | terraform-plugin-framework | Modern, recommended SDK for new providers |
| macOS Integration | cgo + Security.framework | Direct access to all Keychain APIs |

## Architecture

```
terraform-provider-keychain/
├── CLAUDE.md
├── Brewfile                    # Development dependencies (Homebrew)
├── Makefile                    # Cross-architecture build targets
├── .golangci.yml               # Linter config with security rules
├── .gitleaks.toml              # Secret detection config
├── .pre-commit-config.yaml     # Pre-commit hook definitions
├── main.go                     # Provider entry point
├── go.mod
├── go.sum
├── dist/                       # Build output (gitignored)
│   ├── terraform-provider-keychain_v0.1.0_darwin_arm64
│   └── terraform-provider-keychain_v0.1.0_darwin_amd64
├── .github/
│   └── workflows/
│       ├── release.yml         # CI/CD for cross-arch builds
│       └── security.yml        # Security scanning workflow
├── internal/
│   ├── provider/
│   │   └── provider.go         # Provider configuration
│   ├── keychain/               # macOS Keychain cgo bindings
│   │   ├── keychain.go         # Go interface
│   │   ├── keychain_darwin.go  # cgo implementation
│   │   ├── types.go            # Keychain item types
│   │   └── errors.go           # Error handling
│   ├── resources/
│   │   ├── generic_password.go
│   │   ├── internet_password.go
│   │   ├── certificate.go
│   │   ├── key.go
│   │   └── identity.go
│   └── datasources/
│       ├── generic_password.go
│       ├── internet_password.go
│       ├── certificate.go
│       ├── key.go
│       └── identity.go
├── examples/
│   └── ...
└── docs/
    └── ...
```

## macOS Keychain Item Types

The provider must support all `SecItemClass` types:

| Item Class | Terraform Resource | Description |
|------------|-------------------|-------------|
| `kSecClassGenericPassword` | `keychain_generic_password` | Application passwords, API keys, tokens |
| `kSecClassInternetPassword` | `keychain_internet_password` | Web/network credentials with protocol, server, port |
| `kSecClassCertificate` | `keychain_certificate` | X.509 certificates |
| `kSecClassKey` | `keychain_key` | Cryptographic keys (symmetric, asymmetric) |
| `kSecClassIdentity` | `keychain_identity` | Certificate + private key pairs |

## Provider Configuration

```hcl
provider "keychain" {
  # Path to the keychain file (optional, defaults to login keychain)
  keychain_path = "/Users/username/Library/Keychains/MyKeychain.keychain-db"
  
  # Keychain password (optional, for locked keychains)
  # Recommend using environment variable KEYCHAIN_PASSWORD
  password = var.keychain_password
}
```

### Provider Schema

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `keychain_path` | string | No | Full path to keychain file. Defaults to default keychain. |
| `password` | string (sensitive) | No | Password to unlock keychain. Can use `KEYCHAIN_PASSWORD` env var. |

## Resource Schemas

### keychain_generic_password

```hcl
resource "keychain_generic_password" "example" {
  service      = "my-application"
  account      = "api-user"
  password     = var.api_password
  label        = "My Application API Key"
  description  = "API key for production"
  
  access_group = "TEAMID.com.example.app"  # Optional
}
```

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `service` | string | Yes | Service name (kSecAttrService) |
| `account` | string | Yes | Account name (kSecAttrAccount) |
| `password` | string (sensitive) | Yes | The secret data |
| `label` | string | No | Human-readable label |
| `description` | string | No | Item description |
| `access_group` | string | No | Keychain access group |

### keychain_internet_password

```hcl
resource "keychain_internet_password" "example" {
  server       = "api.example.com"
  account      = "deploy-user"
  protocol     = "https"
  port         = 443
  path         = "/v1"
  password     = var.deploy_password
  
  authentication_type = "httpBasic"
}
```

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `server` | string | Yes | Server hostname |
| `account` | string | Yes | Account/username |
| `protocol` | string | Yes | Protocol (http, https, ftp, ssh, etc.) |
| `port` | number | No | Port number |
| `path` | string | No | URL path |
| `password` | string (sensitive) | Yes | The secret data |
| `authentication_type` | string | No | Auth type (httpBasic, httpDigest, etc.) |
| `security_domain` | string | No | Security domain |

### keychain_certificate

```hcl
resource "keychain_certificate" "example" {
  label       = "My CA Certificate"
  certificate = file("ca-cert.pem")  # PEM or DER format
  
  # Read-only attributes populated after creation
  # subject, issuer, serial_number, not_before, not_after
}
```

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `certificate` | string | Yes | Certificate data (PEM or DER, base64 encoded) |
| `label` | string | No | Human-readable label |

Computed attributes: `subject`, `issuer`, `serial_number`, `not_before`, `not_after`, `fingerprint_sha1`, `fingerprint_sha256`

### keychain_key

```hcl
resource "keychain_key" "example" {
  label           = "My Signing Key"
  key_data        = var.private_key_pem
  key_class       = "private"      # private, public, symmetric
  key_type        = "rsa"          # rsa, ec, aes
  key_size_bits   = 2048
  
  extractable     = false
  permanent       = true
}
```

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `key_data` | string (sensitive) | Yes | Key data (PEM or raw, base64 encoded) |
| `label` | string | No | Human-readable label |
| `key_class` | string | Yes | `private`, `public`, or `symmetric` |
| `key_type` | string | Yes | `rsa`, `ec`, `aes`, etc. |
| `key_size_bits` | number | No | Key size in bits |
| `extractable` | bool | No | Whether key can be exported (default: true) |
| `permanent` | bool | No | Store permanently (default: true) |
| `application_tag` | string | No | Application-specific tag |

### keychain_identity

```hcl
resource "keychain_identity" "example" {
  label       = "Code Signing Identity"
  pkcs12_data = filebase64("identity.p12")
  password    = var.p12_password
}
```

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `pkcs12_data` | string (sensitive) | Yes | PKCS#12 bundle (base64 encoded) |
| `password` | string (sensitive) | Yes | Password for PKCS#12 |
| `label` | string | No | Human-readable label |

Computed attributes: Inherits certificate computed attributes plus `key_type`, `key_size_bits`

## Data Sources

All resources have corresponding data sources for reading existing keychain items:

```hcl
data "keychain_generic_password" "existing" {
  service = "existing-service"
  account = "existing-account"
}

output "password" {
  value     = data.keychain_generic_password.existing.password
  sensitive = true
}
```

## cgo Implementation Notes

### Required Frameworks

```go
// #cgo CFLAGS: -x objective-c
// #cgo LDFLAGS: -framework Security -framework CoreFoundation
```

### Key Security.framework Functions

| Function | Purpose |
|----------|---------|
| `SecKeychainOpen` | Open a specific keychain by path |
| `SecKeychainUnlock` | Unlock keychain with password |
| `SecItemAdd` | Add new keychain item |
| `SecItemUpdate` | Update existing item |
| `SecItemDelete` | Delete item |
| `SecItemCopyMatching` | Query/search items |

### Error Handling

Map OSStatus codes to meaningful Go errors:

| OSStatus | Constant | Go Error |
|----------|----------|----------|
| -25291 | `errSecDuplicateItem` | `ErrItemAlreadyExists` |
| -25300 | `errSecItemNotFound` | `ErrItemNotFound` |
| -25293 | `errSecAuthFailed` | `ErrAuthenticationFailed` |
| -25308 | `errSecInteractionNotAllowed` | `ErrInteractionNotAllowed` |
| -25294 | `errSecNoSuchKeychain` | `ErrKeychainNotFound` |

## Security Considerations

### Terraform State

⚠️ **WARNING**: Keychain passwords/secrets will be stored in Terraform state.

Recommendations for users:
- Use encrypted remote state backends (S3 + KMS, Terraform Cloud, etc.)
- Never commit state files to version control
- Consider using `sensitive = true` on all secret outputs
- Use environment variables for provider password configuration

### Keychain Access

- Provider runs with the permissions of the executing user
- Access to specific keychain items may prompt for user authorization
- Consider Keychain Access Control Lists (ACLs) for programmatic access
- For CI/CD, may need to configure `security set-key-partition-list`

### Code Signing (for distribution)

If distributing the provider binary:
- Sign with Developer ID for Gatekeeper approval
- Notarize the binary with Apple
- Hardened runtime required for notarization

## Security Scanning

This project handles sensitive keychain data. Security scanning is **mandatory** before commits and releases.

### Security Tools Overview

| Tool | Purpose | Catches |
|------|---------|---------|
| `gosec` | Static analysis for Go | Hardcoded creds, weak crypto, unsafe functions, SQL injection |
| `govulncheck` | Dependency scanner | Known CVEs in Go modules |
| `gitleaks` | Git history scanner | Accidentally committed secrets, API keys, passwords |
| `trivy` | Comprehensive scanner | Vulnerabilities, misconfigs, embedded secrets |
| `golangci-lint` | Linter aggregator | Can include gosec + security-focused linters |

### Running Security Scans

```bash
# Run all security checks
make security

# Individual tools
make gosec          # Code security issues
make govulncheck    # Dependency vulnerabilities
make gitleaks       # Secrets in git history
make trivy          # Comprehensive scan
```

### golangci-lint Security Configuration

Create `.golangci.yml` to enable security-focused linters:

```yaml
run:
  timeout: 5m

linters:
  enable:
    # Security
    - gosec          # Security issues
    - gocritic       # Code quality and security
    - bodyclose      # HTTP response body not closed
    - noctx          # HTTP requests without context
    - exhaustive     # Enum switch completeness
    
    # Code Quality
    - staticcheck
    - govet
    - errcheck
    - ineffassign
    - unused
    - misspell

linters-settings:
  gosec:
    excludes:
      # Customize if needed, but be conservative
      # - G104  # Unhandled errors (covered by errcheck)
    config:
      global:
        audit: true  # Enable all rules including audit rules
  
  gocritic:
    enabled-tags:
      - diagnostic
      - security
      - performance

issues:
  exclude-use-default: false
  max-issues-per-linter: 0
  max-same-issues: 0
```

### Gitleaks Configuration

Create `.gitleaks.toml` to customize secret detection:

```toml
[allowlist]
description = "Allowlist for terraform-provider-keychain"

# Test fixtures with fake secrets
paths = [
  '''testdata/.*''',
  '''.*_test\.go''',
]

# Example placeholder patterns (not real secrets)
regexes = [
  '''EXAMPLE_.*''',
  '''PLACEHOLDER_.*''',
  '''test-secret-\d+''',
]

# Specific commits to ignore (use sparingly)
# commits = ["abc123..."]
```

### Pre-commit Hooks

Create `.pre-commit-config.yaml` for automated security checks:

```yaml
repos:
  - repo: https://github.com/securego/gosec
    rev: v2.18.2
    hooks:
      - id: gosec
        args: ['-exclude-dir=testdata', './...']

  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.1
    hooks:
      - id: gitleaks

  - repo: https://github.com/golangci/golangci-lint
    rev: v1.55.2
    hooks:
      - id: golangci-lint

  - repo: local
    hooks:
      - id: govulncheck
        name: govulncheck
        entry: govulncheck ./...
        language: system
        pass_filenames: false
```

Install hooks:
```bash
pre-commit install
pre-commit run --all-files  # Initial run
```

### CI/CD Security Integration

Add security scanning to GitHub Actions:

```yaml
  security:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for gitleaks
      
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      
      - name: Install security tools
        run: |
          brew install gosec govulncheck gitleaks trivy
      
      - name: Run gosec
        run: gosec -exclude-dir=testdata ./...
      
      - name: Run govulncheck
        run: govulncheck ./...
      
      - name: Run gitleaks
        run: gitleaks detect --source . --verbose
      
      - name: Run trivy
        run: trivy fs --security-checks vuln,secret,config .
```

### Security Best Practices for This Provider

Since this provider handles keychain secrets:

1. **Never log secret values** - Use `tflog.Trace` with `SecretAttribute` masking
2. **Mark sensitive fields** - Use `Sensitive: true` in all password/key schemas
3. **Zero memory after use** - Clear byte slices containing secrets
4. **Avoid string conversions** - Strings are immutable; use `[]byte` for secrets
5. **No secrets in errors** - Sanitize error messages before returning

Example secure handling:

```go
// Good: Clear sensitive data after use
func (k *Keychain) GetPassword(service, account string) ([]byte, error) {
    password, err := k.secItemCopyMatching(...)
    if err != nil {
        return nil, fmt.Errorf("failed to retrieve item: %w", err)  // No secret in error
    }
    return password, nil  // Caller responsible for zeroing
}

// Good: Zero memory when done
defer func() {
    for i := range password {
        password[i] = 0
    }
}()

// Bad: Secret in error message
return nil, fmt.Errorf("password '%s' is invalid", password)

// Bad: Logging secrets
tflog.Debug(ctx, "Retrieved password", map[string]any{"password": password})
```

## Prerequisites

### Xcode Command Line Tools

Required for cgo cross-compilation to both ARM64 and AMD64:

```bash
xcode-select --install
```

### Homebrew Dependencies

Install all development dependencies using the included Brewfile:

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install all dependencies
brew bundle

# Verify installations
go version          # Should be 1.21+
terraform version   # For acceptance tests
golangci-lint --version
```

### Brewfile Contents

| Formula | Purpose | Required |
|---------|---------|----------|
| `go` | Go 1.21+ for terraform-plugin-framework | ✅ Yes |
| `terraform` | Running acceptance tests | ✅ Yes |
| `golangci-lint` | Code linting (includes staticcheck, gosimple) | ✅ Yes |
| `gosec` | Security issue detection in Go code | ✅ Yes |
| `govulncheck` | Dependency vulnerability scanning | ✅ Yes |
| `gitleaks` | Detect hardcoded secrets in git history | ✅ Yes |
| `trivy` | Comprehensive vulnerability scanner | ✅ Yes |
| `goreleaser` | Automated release builds | Optional |
| `jq` | JSON processing in scripts | Optional |
| `pre-commit` | Git hooks for security & quality checks | Optional |

### Verify cgo Cross-Compilation

Confirm your environment can build for both architectures:

```bash
# Test ARM64 build
CGO_ENABLED=1 GOOS=darwin GOARCH=arm64 go build -o /dev/null ./...

# Test AMD64 build
CGO_ENABLED=1 GOOS=darwin GOARCH=amd64 go build -o /dev/null ./...
```

## Build Instructions

### Cross-Architecture Builds

Both ARM64 (Apple Silicon) and AMD64 (Intel) binaries can be built from a single macOS machine with Xcode installed. Xcode's clang supports both targets natively.

```bash
# Build for Apple Silicon (ARM64)
CGO_ENABLED=1 GOOS=darwin GOARCH=arm64 go build -o dist/terraform-provider-keychain_darwin_arm64

# Build for Intel (AMD64)
CGO_ENABLED=1 GOOS=darwin GOARCH=amd64 go build -o dist/terraform-provider-keychain_darwin_amd64

# Build both architectures
make build-all
```

### Makefile Targets

```makefile
VERSION ?= 0.1.0
BINARY_NAME = terraform-provider-keychain

.PHONY: deps build-all build-arm64 build-amd64 install clean lint test security gosec govulncheck gitleaks trivy

# Install dependencies via Homebrew
deps:
	brew bundle
	@echo "Verifying Xcode Command Line Tools..."
	@xcode-select -p > /dev/null 2>&1 || (echo "Run: xcode-select --install" && exit 1)

build-all: build-arm64 build-amd64

build-arm64:
	CGO_ENABLED=1 GOOS=darwin GOARCH=arm64 go build -o dist/$(BINARY_NAME)_v$(VERSION)_darwin_arm64

build-amd64:
	CGO_ENABLED=1 GOOS=darwin GOARCH=amd64 go build -o dist/$(BINARY_NAME)_v$(VERSION)_darwin_amd64

install: build-all
	mkdir -p ~/.terraform.d/plugins/localhost/local/keychain/$(VERSION)/darwin_arm64
	mkdir -p ~/.terraform.d/plugins/localhost/local/keychain/$(VERSION)/darwin_amd64
	cp dist/$(BINARY_NAME)_v$(VERSION)_darwin_arm64 ~/.terraform.d/plugins/localhost/local/keychain/$(VERSION)/darwin_arm64/$(BINARY_NAME)
	cp dist/$(BINARY_NAME)_v$(VERSION)_darwin_amd64 ~/.terraform.d/plugins/localhost/local/keychain/$(VERSION)/darwin_amd64/$(BINARY_NAME)

lint:
	golangci-lint run

test:
	go test ./...

test-acc:
	TF_ACC=1 go test ./... -v

# Security scanning targets
security: gosec govulncheck gitleaks trivy
	@echo "✅ All security checks passed"

gosec:
	@echo "🔍 Running gosec (Go security checker)..."
	gosec -exclude-dir=testdata -fmt=text ./...

govulncheck:
	@echo "🔍 Running govulncheck (dependency vulnerabilities)..."
	govulncheck ./...

gitleaks:
	@echo "🔍 Running gitleaks (secret detection)..."
	gitleaks detect --source . --verbose

trivy:
	@echo "🔍 Running trivy (comprehensive scan)..."
	trivy fs --security-checks vuln,secret,config --severity HIGH,CRITICAL .

# Pre-release check: all quality and security
check: lint test security
	@echo "✅ All checks passed - ready for release"

clean:
	rm -rf dist/
```

### Local Installation

```bash
# Install both architectures for local development
make install

# Or manually for a specific architecture
mkdir -p ~/.terraform.d/plugins/localhost/local/keychain/0.1.0/darwin_arm64
cp dist/terraform-provider-keychain_v0.1.0_darwin_arm64 \
   ~/.terraform.d/plugins/localhost/local/keychain/0.1.0/darwin_arm64/terraform-provider-keychain
```

### GitHub Actions CI/CD

macOS runners can build both architectures without needing separate ARM and Intel runners:

```yaml
name: Build and Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      
      - name: Build ARM64
        run: CGO_ENABLED=1 GOOS=darwin GOARCH=arm64 go build -o dist/terraform-provider-keychain_darwin_arm64
      
      - name: Build AMD64
        run: CGO_ENABLED=1 GOOS=darwin GOARCH=amd64 go build -o dist/terraform-provider-keychain_darwin_amd64
      
      - name: Create checksums
        run: |
          cd dist
          shasum -a 256 * > terraform-provider-keychain_SHA256SUMS
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: binaries
          path: dist/
```

### Why Separate Binaries (Not Universal)

The Terraform Registry and plugin protocol expect architecture-specific binaries:

| Approach | Terraform Compatible | Notes |
|----------|---------------------|-------|
| Separate binaries | ✅ Yes | Required format for registry publishing |
| Universal (fat) binary | ❌ No | Not recognized by Terraform |

Binary naming convention for releases:
```
terraform-provider-keychain_v0.1.0_darwin_amd64
terraform-provider-keychain_v0.1.0_darwin_arm64
```

### Run Acceptance Tests

```bash
# Requires macOS with test keychain
TF_ACC=1 go test ./... -v
```

## Testing Strategy

### Unit Tests
- Mock Security.framework calls where possible
- Test Go type conversions and error handling

### Integration Tests
- Create temporary keychain for testing
- Test all CRUD operations for each item type
- Run only on macOS (use build tags)

### Acceptance Tests
- Full Terraform lifecycle tests
- Use `resource.Test` from terraform-plugin-testing
- Cleanup test keychains after runs

```go
// +build darwin

func TestAccGenericPassword_basic(t *testing.T) {
    resource.Test(t, resource.TestCase{
        // ...
    })
}
```

## Development Commands

```bash
# First-time setup: install all dependencies
make deps

# Build both architectures
make build-all

# Build specific architecture
make build-arm64
make build-amd64

# Install locally (both architectures)
make install

# Clean build artifacts
make clean

# Run linter
make lint

# Run unit tests
make test

# Run acceptance tests (macOS only)
make test-acc

# Security scanning
make security      # Run ALL security checks
make gosec         # Go code security issues
make govulncheck   # Dependency vulnerabilities
make gitleaks      # Secrets in git history
make trivy         # Comprehensive vulnerability scan

# Pre-release validation (lint + test + security)
make check

# Generate documentation
go generate ./...

# Format code
go fmt ./...

# Set up pre-commit hooks (recommended)
pre-commit install
```

## Dependencies

```go
require (
    github.com/hashicorp/terraform-plugin-framework v1.5.0
    github.com/hashicorp/terraform-plugin-go v0.21.0
    github.com/hashicorp/terraform-plugin-testing v1.6.0
    github.com/hashicorp/terraform-plugin-log v0.9.0
)
```

## References

- [Terraform Plugin Framework Documentation](https://developer.hashicorp.com/terraform/plugin/framework)
- [Apple Security Framework Reference](https://developer.apple.com/documentation/security)
- [Keychain Services API](https://developer.apple.com/documentation/security/keychain_services)
- [SecItem Functions](https://developer.apple.com/documentation/security/keychain_services/keychain_items)

## Status

🚀 **Project Status: Active Development**

- [x] Provider skeleton with plugin framework
- [x] cgo bindings for Security.framework
- [x] Generic password resource/datasource
- [x] Internet password resource/datasource
- [x] Certificate resource/datasource
- [x] Key resource/datasource
- [x] Identity resource/datasource
- [x] Documentation generation
- [x] Acceptance test suite
- [x] CI/CD pipeline documented (GitHub Actions - macOS runners, cross-arch builds)
- [x] Security scanning documented (gosec, govulncheck, gitleaks, trivy)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sparkworx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sparkworx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
