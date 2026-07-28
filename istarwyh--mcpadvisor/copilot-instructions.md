## mcpadvisor

> - **Build**: `pnpm run build` (compiles TypeScript to build/)

# AGENT.md - MCP Advisor Development Guide

## Build & Test Commands
- **Build**: `pnpm run build` (compiles TypeScript to build/)
- **Test**: `pnpm test` (runs vitest), `pnpm run test:watch` (watch mode)
- **Single test**: `pnpm test -- filename.test.ts` or `pnpm test -- --grep "test name"`
- **E2E Test**: `pnpm run test:e2e` (Playwright E2E tests)
- **Meilisearch E2E**: `pnpm test:meilisearch:e2e` (smart automated Meilisearch testing)
- **Lint**: `pnpm run lint` (ESLint), `pnpm run lint:fix` (auto-fix)
- **Format**: `pnpm run format` (Prettier), `pnpm run format:check` (check only)
- **Type check**: `tsc --noEmit` or `pnpm run check` (lint + format check)
- **All quality checks**: `pnpm run check && pnpm run test && pnpm run test:e2e`

## Testing Strategy
- **Unit Tests**: Test individual components with vitest (`src/tests/unit/`)
- **Integration Tests**: Test provider interactions (`src/tests/integration/`)
- **E2E Tests**: Test complete workflows with Playwright (`tests/e2e/`)
- **Smart Automation**: `./scripts/run-meilisearch-e2e.sh` auto-starts all services
- **Quality Gate**: All tests must pass before merging
- **Documentation**: See [Quick Start Guide](docs/GETTING_STARTED.md) and [Technical Reference](docs/TECHNICAL_REFERENCE.md) for comprehensive guides

## Architecture
- **Main**: ESM TypeScript project with Node.js MCP server
- **Core**: Search service with multiple providers (Meilisearch, GetMCP, Compass, Nacos, Offline)
- **Vector Search**: Hybrid search combining text matching + semantic vectors
- **Transport**: Stdio (CLI), SSE (web), REST API support
- **Database**: MySQL2, Meilisearch for vector search, Nacos service discovery
- **Key Directories**: `src/services/` (core logic), `src/types/` (TypeScript types), `src/utils/` (utilities)

## Code Style & Conventions
- **Import style**: Use .js extensions for TypeScript imports, ES modules only
- **Naming**: camelCase for variables/functions, PascalCase for classes/types
- **Types**: Use explicit TypeScript types, interfaces over types for objects
- **Async**: Prefer async/await over Promises, handle errors with try/catch
- **Testing**: Vitest framework, descriptive test names in Chinese, use vi.mock() for mocking
- **Formatting**: Prettier (single quotes, 2 spaces, trailing commas, 80 char width)
- **Logging**: Use winston logger from `utils/logger.js`, structured logging

## Security Best Practices
- **No hardcoded secrets**: Never commit API keys, passwords, or tokens to code
- **Environment variables**: Use `.env` files for local development, GitHub Secrets for CI/CD
- **Default security**: Avoid weak default passwords/keys, force users to set secure values
- **Error handling**: Scripts must return proper exit codes (0 for success, 1+ for failure)
- **Secrets management**: Use `process.env.VARIABLE_NAME` with required validation

## Testing Best Practices  
- **Environment isolation**: Save/restore environment variables in beforeEach/afterEach
- **Smart waiting**: Use `page.waitForFunction()` instead of fixed `page.waitForTimeout()`
- **Content validation**: Verify result content relevance, not just quantity
- **Cross-platform**: Use relative paths `$(pwd)` instead of hardcoded absolute paths
- **Timeout configuration**: Set appropriate timeouts (CI: 180s, local: 60s)
- **Network requests**: Avoid global fetch mocking in test setup - breaks HTTP clients
- **Integration tests**: Use real network requests, mock only specific external dependencies
- **Service dependencies**: Check external service health before running integration tests

## Meilisearch Local Testing
- **Setup**: Binary installation preferred for dev, Docker for CI
- **Environment**: Use environment variable fallbacks for API keys
- **Debug approach**: Systematic API → Client → Test Environment troubleshooting
- **Test types**: Unit (mocked), Integration (real HTTP), E2E (full stack)

## Commit Message Guidelines

Follow [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Types
- `feat`: New feature
- `fix`: Bug fix  
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or fixing tests
- `build`: Build system or dependency changes
- `ci`: CI configuration changes
- `chore`: Other changes that don't modify src or test files

### Auto-Commit Template
When Claude Code makes automatic commits, always include:
- Clear conventional commit message
- Optional body explaining the change
- Standard footer with Claude Code signature

Example:
```
fix(ci): Add pnpm installation step to GitHub Actions workflow

Fix "pnpm: command not found" error by installing pnpm globally
before running pnpm install in all workflow jobs.

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Common Issues & Solutions
- **Interface naming**: Use `IClassName` for interfaces to avoid class/interface conflicts
- **Path portability**: Replace `/Users/username/...` with `$(pwd)/...` in scripts
- **CI timeouts**: GitHub Actions needs longer timeouts for container health checks
- **Test pollution**: Always clean up environment variables modified during tests
- **Health checks**: Ensure scripts return correct exit codes for monitoring

---
> Source: [istarwyh/mcpadvisor](https://github.com/istarwyh/mcpadvisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
