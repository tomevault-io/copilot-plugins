## testing-and-quality

> - **Jest**: TypeScript testing with [jest.config.js](mdc:jest.config.js)

# Testing & Quality Assurance

## 🧪 Testing Infrastructure

### Test Framework
- **Jest**: TypeScript testing with [jest.config.js](mdc:jest.config.js)
- **Coverage**: 90%+ target with HTML and LCOV reports
- **Setup**: [tests/setup.ts](mdc:tests/setup.ts) for test environment configuration
- **Structure**: Mirror source structure in tests/ directory

### Test Configuration
The [jest.config.js](mdc:jest.config.js) provides:
- TypeScript support via `ts-jest`
- Module path mapping for clean imports
- Coverage reporting with multiple formats
- Test environment setup and teardown

## 📁 Test Structure

### Directory Organization
```
tests/
├── setup.ts                     # Test environment setup
├── core/                        # Core functionality tests
│   └── utils/
│       └── config.test.ts       # ConfigManager tests
├── modules/                     # Module-specific tests
│   ├── smartlead/              # SmartLead module tests
│   └── instantly/              # Instantly module tests
└── integration/                 # Integration tests
```

### Example Test Structure
See [tests/core/utils/config.test.ts](mdc:tests/core/utils/config.test.ts) for comprehensive test examples:
```typescript
import { ConfigManager } from '../../../src/core/utils/config';

describe('ConfigManager', () => {
  let configManager: ConfigManager;

  beforeEach(() => {
    configManager = ConfigManager.getInstance();
  });

  afterEach(() => {
    // Cleanup test files
  });

  describe('loadConfig', () => {
    it('should return empty object when no config file exists', () => {
      const config = configManager.loadConfig();
      expect(config).toEqual({});
    });
  });
});
```

## 🔧 Testing Commands

### Running Tests
```bash
npm test                 # Run all tests once
npm run test:watch       # Watch mode for development
npm run test:coverage    # Generate coverage reports
```

### Test Development
```bash
npm run dev              # Development mode with auto-recompilation
npm run type-check       # TypeScript validation
npm run lint             # Code quality checks
```

## ✅ Code Quality Standards

### ESLint Configuration
Configured in [package.json](mdc:package.json) with strict TypeScript rules:
```json
{
  "eslintConfig": {
    "parser": "@typescript-eslint/parser",
    "plugins": ["@typescript-eslint"],
    "extends": [
      "eslint:recommended",
      "@typescript-eslint/recommended"
    ],
    "rules": {
      "@typescript-eslint/no-explicit-any": "warn",
      "@typescript-eslint/no-unused-vars": "error"
    }
  }
}
```

### Prettier Configuration
Code formatting rules in [package.json](mdc:package.json):
```json
{
  "prettier": {
    "semi": true,
    "trailingComma": "es5",
    "singleQuote": true,
    "printWidth": 80,
    "tabWidth": 2
  }
}
```

### Quality Commands
```bash
npm run lint             # ESLint checking
npm run lint:fix         # Auto-fix ESLint issues
npm run format           # Prettier formatting
npm run type-check       # TypeScript validation
```

## 🧪 Writing Tests

### Unit Test Guidelines
1. **Test Single Responsibility**: Each test should test one specific behavior
2. **Use Descriptive Names**: Test names should clearly describe what they test
3. **Setup/Teardown**: Use beforeEach/afterEach for test isolation
4. **Mock External Dependencies**: Mock API calls, file system, etc.

### Test Categories

#### 1. Unit Tests
Test individual functions and classes:
```typescript
describe('ThemeManager', () => {
  it('should apply correct colors for SmartLead theme', () => {
    const theme = new ThemeManager('smartlead');
    expect(theme.primary('text')).toContain('#2563eb');
  });
});
```

#### 2. Integration Tests
Test module interactions:
```typescript
describe('Module Integration', () => {
  it('should load and initialize modules correctly', async () => {
    const selector = new ModuleSelector();
    const module = await selector.loadModule('smartlead');
    expect(module).toBeDefined();
    expect(module.name).toBe('smartlead');
  });
});
```

#### 3. CLI Tests
Test command-line interface:
```typescript
describe('CLI Commands', () => {
  it('should show version correctly', async () => {
    const output = await execCLI(['--version']);
    expect(output).toMatch(/\d+\.\d+\.\d+/);
  });
});
```

### Test Utilities

#### Environment Setup
The [tests/setup.ts](mdc:tests/setup.ts) handles:
- Test environment variables
- Test configuration directory setup
- Console method mocking
- Global test timeout configuration

#### Mock Data
Create reusable mock data for tests:
```typescript
export const mockCampaign = {
  id: 123,
  name: 'Test Campaign',
  status: 'ACTIVE',
  created_at: '2024-01-01T00:00:00Z'
};

export const mockConfig = {
  apiKey: 'test-api-key',
  activeModule: 'smartlead' as ModuleName
};
```

## 📊 Coverage Requirements

### Coverage Targets
- **Functions**: 90%+ coverage
- **Lines**: 90%+ coverage  
- **Branches**: 85%+ coverage
- **Statements**: 90%+ coverage

### Coverage Reports
```bash
npm run test:coverage    # Generate coverage reports
open coverage/index.html # View HTML coverage report
```

### Coverage Configuration
Coverage thresholds in [jest.config.js](mdc:jest.config.js):
```javascript
coverageThreshold: {
  global: {
    branches: 85,
    functions: 90,
    lines: 90,
    statements: 90
  }
}
```

## 🚨 Testing Best Practices

### 1. Test File Organization
- **Mirror Structure**: Tests should mirror the source structure
- **Descriptive Names**: Use `.test.ts` suffix for test files
- **Group Related Tests**: Use describe blocks for logical grouping

### 2. Test Data Management
- **Isolated Data**: Each test should use its own data
- **Cleanup**: Always clean up test files and state
- **Deterministic**: Tests should be predictable and repeatable

### 3. Error Testing
- **Happy Path**: Test normal operation
- **Error Cases**: Test error conditions and edge cases
- **Validation**: Test input validation and error messages

### 4. Async Testing
- **Proper Async**: Use async/await for asynchronous operations
- **Timeout Handling**: Set appropriate timeouts for async tests
- **Promise Resolution**: Ensure promises are properly resolved/rejected

## 🔄 Continuous Integration

### Pre-commit Checks
Before committing code, ensure:
```bash
npm run type-check       # TypeScript validation
npm run lint             # ESLint compliance
npm run format           # Code formatting
npm test                 # All tests pass
npm run build            # Successful compilation
```

### CI/CD Pipeline
Recommended GitHub Actions workflow:
```yaml
name: Test Suite
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run type-check
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

## 🎯 Quality Metrics

### Code Quality Indicators
- ✅ Zero TypeScript compilation errors
- ✅ Zero ESLint warnings/errors
- ✅ 90%+ test coverage
- ✅ All tests passing
- ✅ Successful build

### Performance Metrics
- ✅ Fast test execution (< 30 seconds)
- ✅ Quick development feedback (< 5 seconds)
- ✅ Efficient build process (< 60 seconds)
- ✅ Small bundle size

### Maintainability Metrics
- ✅ High test coverage
- ✅ Clear test structure
- ✅ Comprehensive documentation
- ✅ Consistent code style

---
> Source: [LeadMagic/cold-email-cli](https://github.com/LeadMagic/cold-email-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
