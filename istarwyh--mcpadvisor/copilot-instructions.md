## mcpadvisor

> MCPAdvisor is a TypeScript-based CLI tool for discovering and recommending MCP (Model Context Protocol) servers. It helps users find the right MCP server for their needs and provides installation guidance.

# Claude Instructions for MCPAdvisor

## Project Overview
MCPAdvisor is a TypeScript-based CLI tool for discovering and recommending MCP (Model Context Protocol) servers. It helps users find the right MCP server for their needs and provides installation guidance.

## Development Commands

### Build & Test
- `pnpm run build` - Compile TypeScript and make executable
- `pnpm run test` - Run tests with Vitest
- `pnpm run test:watch` - Run tests in watch mode
- `pnpm run test:coverage` - Run tests with coverage report
- `pnpm run test:jest` - Run Jest tests (alternative test runner)
- `pnpm run test:e2e` - Run end-to-end tests with Playwright
- `pnpm run test:meilisearch:e2e` - Smart Meilisearch E2E testing (auto-starts services)

### Code Quality
- `pnpm run lint` - Run ESLint
- `pnpm run lint:fix` - Run ESLint with auto-fix
- `pnpm run format` - Format code with Prettier
- `pnpm run format:check` - Check code formatting
- `pnpm run check` - Run both lint and format check

### Dependencies
- `pnpm run deps:update` - Update all dependencies to latest
- `pnpm run deps:check` - Check for outdated dependencies
- `pnpm run deps:clean` - Clean and reinstall dependencies

## Architecture

### Core Services
- **Search Service** (`src/services/searchService.ts`) - Main search orchestration
- **Vector Engines** (`src/services/database/`) - Vector database implementations
  - Memory-based vector engine
  - Meilisearch integration
  - OceanBase integration
  - Nacos integration
- **Installation Service** (`src/services/installation/`) - Installation guide generation
- **Server Service** (`src/services/server/`) - MCP server management

### Search Providers
- **Offline Search** - Memory-based search with local data
- **Meilisearch** - Full-text search with vector capabilities
- **Nacos** - Service discovery integration
- **Compass Search** - External search provider

### Key Directories
- `src/services/` - Core business logic
- `src/types/` - TypeScript type definitions
- `src/utils/` - Utility functions
- `src/tests/` - Test files (Vitest and Jest)
- `docs/` - Documentation files
- `config/` - Configuration files

## Code Style & Best Practices

### Naming Conventions
- **Classes**: PascalCase (e.g., `OfflineDataLoader`)
- **Functions/Methods**: camelCase (e.g., `loadFallbackData`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_FALLBACK_DATA_PATH`)
- **Variables**: camelCase (e.g., `serverResponses`)
- **Interfaces**: PascalCase with `I` prefix (e.g., `IVectorSearchEngine`)
- **Files**: kebab-case (e.g., `offline-data-loader.ts`)

### Code Formatting
- Use 2 spaces for indentation
- Keep lines under 80 characters when possible
- Use template strings `` `${variable}` `` instead of concatenation
- Use single quotes `'` for strings, double quotes `"` in JSX
- Always use semicolons

### TypeScript Best Practices
- Always define types for function parameters and return values
- Avoid `any` type; use `unknown` or specific types
- Prefer interfaces over type aliases for objects
- Use union types `string | null` instead of optional `string?`
- Use enums for finite value sets
- Use type guards for type narrowing
- Use named exports instead of default exports

```typescript
// Good practice
export interface SearchOptions {
  minSimilarity?: number;
  limit: number;
}

export function search<T>(query: string, options: SearchOptions): Promise<T[]> {
  // implementation
}

// Avoid
export default function search(query: any, options?: any): any {
  // implementation
}
```

### Functional Programming Principles
- Write pure functions without side effects
- Avoid mutating parameters; return new objects
- Use function composition for complex functionality
- Follow single responsibility principle
- Use higher-order functions (`map`, `filter`, `reduce`)
- Avoid deep nesting; use function composition or Promise chains

```typescript
// Good practice
function normalizeVector(vector: number[]): number[] {
  const magnitude = calculateMagnitude(vector);
  return vector.map(value => value / magnitude);
}

// Avoid
function processVector(vector: number[]): number[] {
  // doing multiple things: calculate, normalize, filter, etc.
}
```

### Path Handling
- Use `pathUtils.ts` for all path-related operations
- Ensure compatibility across development, test, and production environments
- Handle `import.meta.url` compatibility properly
- Avoid hardcoded absolute paths; use relative paths
- Use `path.join()` and `path.resolve()` for path separators
- Provide fallback path mechanisms

```typescript
// Good practice
import { getMcpServerListPath } from '../utils/pathUtils.js';
const dataPath = getMcpServerListPath(import.meta.url);

// Avoid
const dataPath = path.resolve(__dirname, '../../../../data/file.json');
```

### Error Handling
- Use specific error types instead of generic `Error`
- Provide useful error messages with context
- Properly propagate errors; don't swallow them
- Use `try/catch` or Promise `.catch()` for async errors
- Log error details for debugging
- Provide graceful fallbacks when possible
- Scripts must return proper exit codes (0 for success, 1+ for failure)

```typescript
// Good practice
async function loadData(): Promise<Data[]> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new HttpError(`Request failed: ${response.status}`, response.status);
    }
    return await response.json();
  } catch (error) {
    logger.error('Failed to load data', { error, url });
    return []; // Graceful fallback
  }
}

// Shell script error handling
# Good practice
if ! curl -f http://localhost:7700/health; then
    echo 'Service not available' >&2
    exit 1  # Proper error exit code
fi
```

### Security Best Practices
- **Never hardcode secrets**: API keys, passwords, tokens must use environment variables
- **Secure defaults**: Avoid weak default passwords; force users to set secure values
- **Environment validation**: Check required environment variables at startup
- **GitHub Secrets**: Use `${{ secrets.SECRET_NAME }}` in CI/CD workflows
- **Documentation security**: Remove any hardcoded keys from docs and examples

```typescript
// Good practice - Environment variable validation
function getRequiredEnv(name: string): string {
  const value = process.env[name];
  if (!value) {
    throw new Error(`Required environment variable ${name} is not set`);
  }
  return value;
}

const apiKey = getRequiredEnv('MEILISEARCH_API_KEY');

// Avoid - Hardcoded secrets
const apiKey = 'abc123def456'; // NEVER do this
```

### Docker & Container Best Practices
- **Required environment variables**: Use `${VAR:?Error message}` to force variable setting
- **Health checks**: Implement proper health check endpoints
- **Resource limits**: Set appropriate memory and CPU limits
- **Security contexts**: Run containers with non-root users when possible

```yaml
# Good practice - Force environment variable
environment:
  MEILI_MASTER_KEY: ${MEILI_MASTER_KEY:?Please set MEILI_MASTER_KEY environment variable}

# Good practice - Health check with proper timeout
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:7700/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

## Testing Guidelines

### Core Testing Principles
- **Predictability**: Test results should be completely predictable
- **Isolation**: Each test should run independently
- **Maintainability**: Test code should be as clean as production code
- **Clarity**: Test names and assertions should clearly express intent

### Test Structure
- Use `describe` to organize related tests, `test`/`it` for specific cases
- Follow Given-When-Then pattern
- Use `beforeEach`/`afterEach` for setup and cleanup
- Use assertions instead of `console.log` for verification

### Test Design
- **Single responsibility**: Each test should verify one functionality
- **Edge cases**: Test boundary values, null values, invalid inputs
- **Error handling**: Explicitly test error conditions and exception handling
- **Performance**: Tests should execute quickly
- **Environment isolation**: Save/restore environment variables in beforeEach/afterEach
- **Content validation**: Verify result content relevance, not just quantity
- **Network requests**: Avoid global mocking that blocks integration tests

### Integration Test Best Practices
- **Real network requests**: Integration tests should use real HTTP requests, not mocks
- **Global fetch mocking**: Avoid `global.fetch = vi.fn()` in setup - it breaks HTTP clients
- **Targeted mocking**: Only mock specific functions in individual test files
- **API key configuration**: Use environment variable fallbacks for test configurations
- **Service availability**: Always check service health before running integration tests

### Common Integration Test Issues
- **Fetch mocking conflicts**: Global fetch mocks prevent HTTP clients from working
- **API key mismatches**: Test configurations must use correct authentication
- **Service availability**: Tests fail if external services (like Meilisearch) aren't running
- **Environment variables**: Test and runtime environments may have different variable names

### Mocking & Stubs
- Use `vi.fn()` to create mock functions
- Mock external dependencies (API calls, file system operations)
- Keep mocks simple; only mock necessary parts
- Verify mock call counts and parameters

### Async Testing
- Always use `async/await` for async operations
- Use smart waiting: `page.waitForFunction()` instead of fixed `page.waitForTimeout()`
- Test both Promise resolution and rejection
- Set appropriate timeouts (CI: 180s, local: 60s for container health checks)

```typescript
// Example: Good test structure with environment isolation
describe('UserService', () => {
  let service: UserService;
  let mockRepository: MockRepository;
  let originalEnvVars: Record<string, string | undefined> = {};

  beforeEach(() => {
    // Save original environment variables
    originalEnvVars = {
      API_KEY: process.env.API_KEY,
      DATABASE_URL: process.env.DATABASE_URL
    };
    
    mockRepository = {
      findUser: vi.fn(),
      saveUser: vi.fn().mockResolvedValue({ id: '1', name: 'Test' })
    };
    service = new UserService(mockRepository);
  });

  afterEach(() => {
    // Restore original environment variables
    Object.entries(originalEnvVars).forEach(([key, value]) => {
      if (value === undefined) {
        delete process.env[key];
      } else {
        process.env[key] = value;
      }
    });
  });

  describe('createUser', () => {
    it('should successfully create user and return user data', async () => {
      // Given
      const userData = { name: 'Test', email: 'test@example.com' };
      
      // When
      const result = await service.createUser(userData);
      
      // Then
      expect(result).toHaveProperty('id');
      expect(mockRepository.saveUser).toHaveBeenCalledWith(expect.objectContaining(userData));
    });

    it('should throw error when name is empty', async () => {
      await expect(service.createUser({ name: '', email: 'test@example.com' }))
        .rejects
        .toThrow('Username cannot be empty');
    });
  });
});

// Example: Smart waiting in E2E tests
describe('Search functionality', () => {
  it('should display search results', async ({ page }) => {
    await page.getByRole('button', { name: 'Search' }).click();
    
    // Good practice - Wait for content to appear
    await page.waitForFunction(() => {
      const content = document.body.textContent || '';
      return content.includes('Results:') || content.includes('No results');
    }, { timeout: 15000 });
    
    // Verify content quality, not just presence
    const results = await page.locator('[data-testid="search-results"]').count();
    expect(results).toBeGreaterThan(0);
  });
});
```

### Test Frameworks
- Use Vitest for new tests (primary test runner)
- Jest is available for legacy tests
- Test files should be in `src/tests/` directory
- Use `__mocks__/` for mocking external dependencies
- Use `fixtures/` for test data

## Commit Message Guidelines

Follow [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
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

### Commit Message Format Requirements

**IMPORTANT**: The project has commitlint pre-commit hooks that enforce commit message standards:

- **Subject line format**: `<type>[optional scope]: <Description>` (note capitalization)
- **Type**: Must be a valid conventional commit type
- **Description**: Must start with capital letter (sentence-case)
- **Length**: Subject line must not exceed 72 characters

**Common mistakes to avoid**:
```bash
# ❌ Wrong - lowercase description
feat: add new search functionality

# ✅ Correct - capitalized description  
feat: Add new search functionality

# ❌ Wrong - invalid type
feature: Add new functionality

# ✅ Correct - valid type
feat: Add new functionality
```

### Pre-commit Hook Troubleshooting

If your commit is rejected by pre-commit hooks:

1. **Type checking failure**:
   ```bash
   pnpm run check
   pnpm run build
   git commit -m "fix: Resolve type errors"
   ```

2. **Linting failure**:
   ```bash
   pnpm run lint:fix
   git add .
   git commit -m "style: Fix linting issues"
   ```

3. **Commit message format error**:
   ```bash
   # Fix the commit message format
   git commit --amend -m "feat: Add proper commit message"
   ```

### Example
```
fix(data): ensure mcp_server_list.json is included in npm package

Add data directory to package.json files field and improve path
resolution logic to support proper data file loading in packaged
environments.

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>

Fixes #123
```

### Auto-Commit Template
When Claude Code makes automatic commits, always include:
- Clear conventional commit message
- Optional body explaining the change  
- Standard footer with Claude Code signature
- Optional issue reference

Template format:
```
<type>[optional scope]: <description>

[optional body explaining the change]

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>

[optional footer like "Fixes #123"]
```

## Important Technical Notes
- This is a TypeScript project using ES modules (`"type": "module"`)
- Uses pnpm as package manager
- Main entry point is `src/index.ts`
- Built files go to `build/` directory
- Husky is configured for git hooks

## Common Issues & Solutions from Code Reviews
- **Interface naming conflicts**: Use `IClassName` for interfaces to avoid class/interface conflicts
- **Hardcoded paths**: Replace absolute paths like `/Users/username/...` with `$(pwd)/...` in scripts
- **CI timeouts**: GitHub Actions containers need longer timeouts (180s vs 60s local)
- **Test pollution**: Always save/restore environment variables in tests
- **Health check logic**: Ensure scripts return correct exit codes for monitoring
- **Security vulnerabilities**: Never commit hardcoded API keys or weak default passwords
- **Path portability**: Use relative paths and proper path utilities for cross-platform compatibility
- **Global fetch mocking**: Avoid global fetch mocks in test setup - breaks integration tests
- **API key configuration**: Use proper environment variable fallbacks in test configurations
- **Service dependencies**: Check external service availability before running integration tests

## Meilisearch Testing & Integration
- **Local setup**: Use binary installation for development, Docker for CI/CD
- **Test environment**: Remove global fetch mocking to allow real HTTP requests
- **API configuration**: Use environment variable fallbacks: `TEST_KEY || MASTER_KEY || default`
- **Health checks**: Always verify service availability before running integration tests
- **Error debugging**: Use systematic approach: API → Client → Test Environment
- **Network isolation**: Integration tests need real network, unit tests can use mocks

## When Working on This Project
1. Always run `pnpm run check` before committing
2. Use `pnpm run test` to ensure tests pass
3. Follow the existing code patterns and architecture
4. Add tests for new functionality
5. Update documentation when adding new features
6. Use `pathUtils.ts` for all path operations
7. Write pure functions when possible
8. Provide proper error handling with context
9. Follow the naming conventions consistently
10. Write meaningful commit messages following Conventional Commits

## Security & Quality Checklist Before Submitting PRs
- [ ] No hardcoded API keys, passwords, or tokens in code or documentation
- [ ] All secrets use environment variables or GitHub Secrets
- [ ] Scripts return proper exit codes (0 for success, 1+ for failure)
- [ ] Tests save/restore environment variables to prevent pollution
- [ ] Use smart waiting mechanisms in E2E tests instead of fixed timeouts
- [ ] Verify test result content quality, not just quantity
- [ ] Use relative paths in scripts for cross-platform compatibility
- [ ] Interface and class names don't conflict (use `IClassName` pattern)
- [ ] CI timeouts are appropriate for container operations (180s recommended)
- [ ] Docker configurations use required environment variables, not weak defaults

## Development Memories
- Always add the necessary files to package.json and project structure
- Use proper path resolution for cross-platform compatibility
- Ensure configuration files and dependencies are included in builds

---
> Source: [istarwyh/mcpadvisor](https://github.com/istarwyh/mcpadvisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
