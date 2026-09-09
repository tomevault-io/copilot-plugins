## go-project-structure

> Go Project Structure Rules

## Description
This rule ensures that Go projects follow a consistent and idiomatic structure, making the codebase easier to navigate, maintain, and scale.

## Rule
When organizing a Go project:

### Directory Structure
1. Use the standard Go project layout:
   - `/cmd` - Main applications for this project
   - `/internal` - Private application and library code
   - `/pkg` - Library code that's ok to use by external applications
   - `/api` - API definitions, protocol buffers, OpenAPI/Swagger specs
   - `/web` - Web application specific components
   - `/configs` - Configuration file templates or default configs
   - `/scripts` - Scripts to perform various build, install, analysis, etc. operations
   - `/test` - Additional external test apps and test data

2. Place main application entry points in `/cmd/<appname>` directories
3. Keep internal code that shouldn't be imported by other projects in `/internal`
4. Place shared libraries that can be imported by other projects in `/pkg`

### Package Organization
1. Organize packages by functionality, not by type
2. Keep package names short, clear, and descriptive
3. Avoid package name collisions with standard library
4. Use singular form for package names (e.g., `store`, not `stores`)
5. Avoid deeply nested package hierarchies

### Dependency Management
1. Use Go modules for dependency management
2. Pin dependencies to specific versions in go.mod
3. Regularly update and audit dependencies
4. Minimize the number of external dependencies
5. Prefer standard library solutions when available

### Configuration
1. Use environment variables for configuration
2. Support configuration files as an alternative
3. Provide sensible defaults for all configuration options
4. Validate configuration at startup

### Testing
1. Place tests in the same package as the code they test
2. Use `_test.go` suffix for test files
3. Place integration tests in a separate `integration` package
4. Store test fixtures in `testdata` directories

## Implementation
- The Cursor IDE will enforce this rule by:
  - Suggesting appropriate locations for new files
  - Highlighting structural issues
  - Providing refactoring options to improve project organization

## Benefits
- Consistent, navigable project structure
- Clear separation of concerns
- Better code reusability
- Easier onboarding for new team members
- Alignment with Go community standards

## Examples

### ✅ Correct Project Structure:

```
myproject/
├── cmd/
│   └── myapp/
│       └── main.go
├── internal/
│   ├── auth/
│   │   └── auth.go
│   └── database/
│       └── database.go
├── pkg/
│   └── validator/
│       └── validator.go
├── api/
│   └── openapi.yaml
├── configs/
│   └── config.yaml
├── scripts/
│   └── setup.sh
└── go.mod
```

### ❌ Incorrect Project Structure:

```
myproject/
├── main.go
├── auth.go
├── database.go
├── validator.go
└── go.mod
``` 

---
> Source: [dstotijn/mcp-cbs-cijfers-open-data](https://github.com/dstotijn/mcp-cbs-cijfers-open-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
