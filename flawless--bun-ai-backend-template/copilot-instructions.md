## bun-ai-backend-template

> Corporate backend microservices development environment with strict quality enforcement and measurable guarantees.

# Development Standards

Corporate backend microservices development environment with strict quality enforcement and measurable guarantees.

## 🔧 Development Hooks

### Type Checking

```bash
bun run type-check
```

- Runs TypeScript compiler without emitting files
- Uses strict mode with comprehensive type checking
- Configured for Bun-specific optimizations

### Linting

```bash
bun run lint      # Check code quality
bun run lint:fix  # Auto-fix issues
```

- ESLint with flat config (v9+)
- TypeScript ESLint rules with type-aware linting
- Security plugin for vulnerability detection

### Formatting

```bash
bun run format        # Format all files
bun run format:check  # Check formatting (CI/CD)
```

- Prettier with modern configuration
- 100 character line width
- Single quotes, trailing commas
- Consistent indentation (2 spaces)

### Testing

```bash
bun run test              # Run all tests
bun run test:unit         # Unit tests only
bun run test:integration  # Integration tests only
bun run test:contract     # Contract tests only
bun run test:e2e          # End-to-end tests only
bun run test:coverage     # Run with coverage report
bun run test:watch        # Watch mode
```

#### Testing Philosophy

This project follows the **Testing Trophy** approach advocated by Kent C. Dodds:

> "The modern approach emphasizes integration testing over unit tests, following Kent C. Dodds' 'Testing Trophy' philosophy: 'Write tests. Not too many. Mostly integration.' For TypeScript API projects, this means allocating 45-50% of tests to integration coverage, 25-30% to unit tests, and 10-15% each to contract and E2E tests."

**Test Examples:**

- **Integration Tests (45-50%)**: `tests/integration/` - Multi-component interaction examples
- **Unit Tests (25-30%)**: `tests/unit/` - Individual function behavior examples
- **Contract Tests (10-15%)**: `tests/contract/` - API contract compliance examples
- **E2E Tests (10-15%)**: `tests/e2e/` - End-to-end workflow examples

**Note:** Template includes minimal test examples demonstrating each Testing Trophy level. Expand with project-specific tests following the distribution ratios.

### Pre-commit Hooks

**⚠️ SECURITY NOTICE:**
Bypassing pre-commit hooks using `--no-verify` or any other method is **STRICTLY PROHIBITED** and will be tracked, monitored, and result in disciplinary action. All commits must pass automated quality gates.

## 🏗️ Architecture & Stack

### Core Technologies

- **Runtime:** Bun (ESNext, fast bundler, native test runner)
- **Language:** TypeScript (strict mode, latest features)
- **Framework:** Elysia (fast, end-to-end type safety, Bun-optimized)
- **HTTP Server:** Built-in Bun HTTP server

### Development Tools

- **Package Manager:** Bun
- **Linter:** ESLint v9 (flat config)
- **Formatter:** Prettier
- **Test Runner:** Bun Test (native, with coverage)
- **Git Hooks:** Husky v9
- **Type Checker:** TypeScript 5.6+

## 📁 Project Structure

```
ts-backend-template/
├── src/
│   └── index.ts           # Main application entry point
├── tests/
│   ├── unit/              # Unit test examples (25-30%)
│   ├── integration/       # Integration test examples (45-50%)
│   ├── contract/          # Contract test examples (10-15%)
│   ├── e2e/              # E2E test examples (10-15%)
│   └── test-utils.ts     # Minimal test utilities
├── .claude/              # Claude Code hooks
├── .husky/               # Git hooks
├── package.json          # Project dependencies and scripts
├── tsconfig.json         # TypeScript configuration
├── eslint.config.js      # ESLint flat configuration
├── .prettierrc           # Prettier configuration
├── bunfig.toml          # Bun configuration
└── README.md            # Project documentation
```

## 🚀 Available Commands

### Development

- `bun run start:dev` - Start development server with hot reload
- `bun run start` - Start production server
- `bun run build` - Build for production

### Code Quality

- `bun run type-check` - TypeScript validation
- `bun run lint` - Code quality and security checks
- `bun run lint:fix` - Auto-fix code issues
- `bun run format` - Code formatting
- `bun run format:check` - Format validation

### Testing

- `bun run test` - Run all tests
- `bun run test:[unit|integration|contract|e2e]` - Run specific test type
- `bun run test:coverage` - Coverage report
- `bun run test:watch` - Watch mode

### Maintenance

- `bun run clean` - Clean build artifacts
- `bun run prepare` - Setup Husky hooks

## 🔧 Configuration Notes

### Bun Optimizations

- Native TypeScript execution
- Fast bundling and dependency resolution
- Built-in test runner with coverage
- Module resolution: "bundler" mode
- Target: ESNext for maximum performance

### TypeScript Strict Mode

- All strict flags enabled
- `noUncheckedIndexedAccess` for safer array access
- `exactOptionalPropertyTypes` for precise optionals
- Comprehensive error checking and type safety

### ESLint Security

- `eslint-plugin-security` for vulnerability detection
- Type-aware rules for TypeScript
- Import/export validation
- Consistent code patterns

## 📊 Health Check Standards

All services must implement standardized health and readiness endpoints:

### Health Endpoint

```typescript
GET /health
{
  "status": "ok" | "degraded" | "unhealthy",
  "timestamp": "ISO8601",
  "uptime": number,
  "environment": string,
  "version": string
}
```

### Readiness Endpoint

```typescript
GET /ready
{
  "status": "ready" | "not_ready",
  "timestamp": "ISO8601",
  "latency": number,
  "dependencies": {
    "database": {
      "status": "healthy" | "unhealthy",
      "latency": number,
      "last_checked": "ISO8601",
      "endpoint": string,
      "error"?: string
    },
    "email": {
      "status": "healthy" | "unhealthy",
      "latency": number,
      "last_checked": "ISO8601",
      "endpoint": string,
      "error"?: string
    }
  }
}
```

## 🏆 Quality Standards

**Enforced Standards:**

- **80% minimum test coverage** - Branch and line coverage enforced
- **Zero TypeScript errors** - Strict mode compilation required
- **Zero security violations** - ESLint security plugin enforced
- **Consistent formatting** - Prettier compliance required
- **Testing Trophy distribution** - 45-50% integration, 25-30% unit, 10-15% contract/e2e

**Quality Gates:**

- Automated validation prevents non-compliant code
- All commits must meet strict quality standards
- Coverage threshold enforcement (80% minimum)
- Security scanning integrated into development workflow

## 🔧 Development Environment

**Required Tools:**

- Bun runtime (latest stable)
- TypeScript 5.6+ with strict configuration
- ESLint v9 with flat config
- Prettier with corporate formatting rules
- Husky v9 for git hooks

**Service Standards:**

- Structured logging with correlation IDs
- Health checks with standardized responses
- Readiness checks with dependency monitoring
- Configuration validation at startup
- Graceful shutdown procedures
- Performance monitoring integration

## 📚 Documentation Structure

This project uses a three-tier documentation approach:

- **@concepts/README.md** - High-level conceptual documentation for domain models, system principles, and business workflows
- **@adr/README.md** - Architecture Decision Records documenting technical choices with context and rationale
- **@tasks/README.md** - Implementation-focused task documentation with step-by-step guidance and phases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Flawless)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Flawless)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
