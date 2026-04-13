## plan-pulse-compass

> **🔥 CRITICAL**: ALL development sessions MUST follow the [ULTRA Token Optimization Protocol](./ULTRA_TOKEN_OPTIMIZATION.md) to achieve 95% token reduction.

# Plan Pulse Compass - Claude Development Guidelines

## 🚀 ULTRA Token Optimization - MANDATORY 95% REDUCTION

**🔥 CRITICAL**: ALL development sessions MUST follow the [ULTRA Token Optimization Protocol](./ULTRA_TOKEN_OPTIMIZATION.md) to achieve 95% token reduction.

**See detailed protocol in**: `ULTRA_TOKEN_OPTIMIZATION.md`

### MANUAL-ONLY Testing Protocol

**❌ NEVER run tests via Claude tools**  
**✅ USER runs all tests manually and provides ONLY error summaries**

**Workflow**:

1. Claude provides code changes ONLY
2. **Claude ALWAYS prompts**: "**Please run tests**: `npm run test:core && npm run typecheck && npm run lint && npm run build` and share brief results"
3. User runs tests and shares: "✅ All pass" OR "❌ 3 test failures in utils/skillBasedPlanning.test.ts lines 45,67,89"
4. Claude fixes based on MINIMAL error info provided
5. Repeat until user confirms "✅ All pass"

**Test Commands for User**:

```bash
# MANDATORY before any commit
npm run test:core          # Share: pass/fail + file names only
npm run lint               # Share: error count + file names only
npm run typecheck          # Share: error count + file names only
npm run build              # Share: pass/fail only

# OPTIONAL
npm run test:e2e           # Share: critical errors only
npm run test:integration   # Share: failure summaries only
```

**Token Savings**: 2,000-8,000 tokens per test cycle → 50-200 tokens

### Code Investigation Protocol

**❌ NEVER read entire files**  
**✅ USER provides specific line ranges and error context**

**Workflow**:

1. Claude asks: "Show me lines X-Y in file.ts where the error occurs"
2. User copies ONLY the requested lines
3. Claude provides targeted fix
4. User applies and tests

**Token Savings**: 2,000+ tokens per file → 200-400 tokens

---

## Project Overview

Plan Pulse Compass is a comprehensive planning and project management application built with React 18.3.1, TypeScript 5.5.3, and Vite 6.0.1. The project uses a modern tech stack with shadcn/ui components, Vitest for testing, and comprehensive CI/CD pipelines.

## Core Technologies

- **Frontend**: React 18.3.1 with TypeScript 5.5.3
- **Build Tool**: Vite 6.0.1
- **UI Library**: shadcn/ui with Radix UI primitives
- **Testing**: Vitest with Testing Library
- **Date Handling**: date-fns for date manipulation
- **Calendar**: react-day-picker for calendar functionality
- **State Management**: React Context API
- **Styling**: Tailwind CSS with CSS custom properties

## Test-Driven Development (TDD) Guidelines

### TDD-First Practice - MANDATORY

**ALWAYS** follow the TDD cycle for ALL code changes:

1. **🔴 RED**: Write a failing test first
   - Create test cases that define the expected behavior
   - Ensure tests fail initially (proving they're testing the right thing)
   - Use descriptive test names that explain the requirement

2. **🟢 GREEN**: Write minimal code to make tests pass
   - Implement only what's needed to satisfy the failing test
   - Avoid over-engineering or adding unnecessary features
   - Focus on making the test pass, not on perfect code

3. **🔵 REFACTOR**: Clean up code while keeping tests green
   - Improve code structure, readability, and performance
   - Ensure all tests continue to pass throughout refactoring
   - Remove duplication and improve design

### TDD Implementation Rules

- **Never write production code without a failing test**
- **Write only enough test code to demonstrate a failure**
- **Write only enough production code to make the failing test pass**
- **Tests must be independent and repeatable**
- **Test behavior, not implementation details**

## ULTRA-Optimized Testing Workflow

### USER-ONLY Testing Protocol - MANDATORY

**🚫 Claude NEVER runs test commands**  
**✅ USER runs tests and shares MINIMAL results**

```bash
# MANDATORY sequence - USER runs, shares brief results
npm run test:core      # Share: "✅ pass" or "❌ 3 fails: utils/skill.test.ts:45,67,89"
npm run typecheck      # Share: "✅ pass" or "❌ 2 errors: Teams.tsx:15, utils/math.ts:32"
npm run lint           # Share: "✅ pass" or "❌ 5 warnings: unused imports"
npm run build          # Share: "✅ pass" or "❌ build failed: import error"

# OPTIONAL - only if requested by Claude
npm run test:e2e       # Share: "✅ pass" or "❌ Teams page: Select undefined"
npm run test:integration # Share: brief failure summaries only
```

**Communication Examples**:

- ✅ "All core tests pass, 2 lint warnings about unused imports in Teams.tsx"
- ❌ "3 test failures in skillBasedPlanning.test.ts lines 45,67,89 - compatibility calc errors"
- ❌ "TypeScript error in Teams.tsx:15 - Select component not found"

### Error Reporting Protocol

**ULTRA-Compressed Error Sharing**:

- File name + line numbers only
- Brief error type (import, type, logic)
- NO full stack traces or detailed output
- NO test code copying unless specifically requested

**Example Error Reports**:

```
❌ test:core - 3 fails:
- skillBasedPlanning.test.ts:45 - compatibility calc wrong
- skillBasedPlanning.test.ts:67 - missing mock data
- Teams.test.tsx:89 - component render error

❌ typecheck - 2 errors:
- Teams.tsx:15 - Select import missing
- utils/math.ts:32 - return type mismatch

✅ lint, build - all pass
```

### Test Categories (Reference Only)

- **Core**: Components, utilities, context, business logic
- **Integration**: Context providers, complex workflows
- **E2E**: User journeys, console errors, cross-browser
- **Build**: TypeScript, linting, production build

## OPTIMIZED Development Workflow

### Code-First Approach

1. **Claude provides code changes ONLY**
2. **Claude prompts**: "**Please run tests** and share brief results"
3. **USER copies code, runs tests, reports results**
4. **Claude fixes based on minimal error info**
5. **Repeat until USER confirms "✅ All pass"**

### Pre-Commit Checklist (USER-ONLY)

```bash
# USER runs and reports: "✅ all pass" or "❌ brief error summary"
npm run test:core && npm run typecheck && npm run lint && npm run build

# OPTIONAL - only if Claude requests E2E validation
npx playwright test tests/e2e/page-console-errors.spec.ts
```

**Token Savings**: 5,000+ tokens per test cycle → 50-100 tokens

## Code Investigation Protocol

### File Reading Optimization

**❌ NEVER**: Read entire files (2000+ tokens)
**✅ ALWAYS**: Request specific line ranges (200-400 tokens)

**Workflow**:

1. Claude asks: "Show me lines 45-67 in utils/skillBasedPlanning.ts where the error occurs"
2. USER copies ONLY requested lines
3. Claude provides targeted fix
4. USER applies change and tests

### Error Context Sharing

**❌ NEVER**: Share full stack traces, console output, or test results  
**✅ ALWAYS**: Share minimal error summaries

**Examples**:

- "❌ 3 test fails in skillBasedPlanning.test.ts:45,67,89 - calc errors"
- "❌ Teams.tsx:15 - Select import missing"
- "✅ All tests pass, ready to commit"

### Commit Protocol

**USER-ONLY Checklist**:

- [ ] Run: `npm run test:core && npm run typecheck && npm run lint && npm run build`
- [ ] Report: "✅ all pass" or brief error summary
- [ ] Apply Claude's fixes (if needed)
- [ ] Repeat until "✅ all pass"
- [ ] Commit and push

### Token Usage Targets

**Before Optimization**: 15,000-25,000 tokens per session  
**After Optimization**: 1,000-3,000 tokens per session  
**Savings**: 80-95% reduction

**Key Savings**:

- Test execution: 8,000 tokens → 50 tokens (99% reduction)
- File reading: 2,000 tokens → 300 tokens (85% reduction)
- Error investigation: 5,000 tokens → 500 tokens (90% reduction)
- Context building: 10,000 tokens → 1,000 tokens (90% reduction)

## Code Quality Standards

### TypeScript Interface Consistency

**CRITICAL**: Ensure test mock data ALWAYS matches TypeScript interfaces:

```typescript
// ❌ WRONG - Mock data doesn't match interface
const mockAllocation = {
  id: 'alloc1',
  teamId: 'team1',
  iterationNumber: 1, // ❌ Field doesn't exist in Allocation interface
  runWorkCategoryId: '', // ❌ Field doesn't exist in Allocation interface
};

// ✅ CORRECT - Mock data matches Allocation interface exactly
const mockAllocation: Allocation = {
  id: 'alloc1',
  personId: 'person1', // ✅ Required field included
  teamId: 'team1',
  projectId: 'proj1',
  epicId: 'epic1',
  cycleId: 'iter1',
  percentage: 80,
  type: 'project', // ✅ Required field included
  startDate: '2024-01-01', // ✅ Required field included
  endDate: '2024-01-14', // ✅ Required field included
  notes: '',
};
```

### Component Testing Standards

- **Props Testing**: Test all component props and their edge cases
- **Event Handling**: Test all user interactions and event handlers
- **State Management**: Test component state changes and side effects
- **Error Boundaries**: Test error handling and graceful degradation
- **Accessibility**: Test ARIA attributes and keyboard navigation

### Performance Standards

- **Component Rendering**: <100ms for initial render
- **Test Execution**: Core test suite <30 seconds
- **Bundle Size**: <500KB initial, <2MB total
- **Memory Usage**: <100MB baseline, no memory leaks
- **API Response**: <200ms average response time

## Development Workflow

### Feature Development Process

1. **📋 Create Feature Tests**

   ```bash
   # Create test file first
   touch src/components/__tests__/NewComponent.test.tsx

   # Write comprehensive test cases
   # Tests should cover all requirements and edge cases
   ```

2. **🔴 Write Failing Tests**

   ```typescript
   describe('NewComponent', () => {
     it('should render with required props', () => {
       render(<NewComponent requiredProp="value" />);
       expect(screen.getByText('Expected Text')).toBeInTheDocument();
     });
   });
   ```

3. **🟢 Implement Component**

   ```typescript
   // Implement minimal code to make tests pass
   const NewComponent: React.FC<Props> = ({ requiredProp }) => {
     return <div>Expected Text</div>;
   };
   ```

4. **🔵 Refactor and Optimize**

   ```typescript
   // Improve implementation while keeping tests green
   // Add proper styling, error handling, accessibility
   ```

5. **✅ Validate Complete Suite**
   ```bash
   npm run test:core
   npm run type-check
   npm run lint
   npm run build
   ```

### Bug Fix Process

1. **🔍 Reproduce Bug**: Write a test that demonstrates the bug
2. **🔴 Failing Test**: Ensure test fails due to the bug
3. **🔧 Fix Implementation**: Modify code to fix the bug
4. **🟢 Validate Fix**: Ensure bug test now passes
5. **🧪 Regression Testing**: Run full test suite to prevent regressions
6. **📝 Document**: Update documentation if needed

### Refactoring Process

1. **🛡️ Safety Net**: Ensure comprehensive test coverage exists
2. **🔄 Small Steps**: Make incremental changes
3. **✅ Continuous Validation**: Run tests after each change
4. **📊 Performance Check**: Verify no performance degradation
5. **🧹 Cleanup**: Remove dead code and improve documentation

## CI/CD Pipeline Integration

### Pipeline Status: ✅ RE-ENABLED & OPTIMIZED

The GitHub Actions CI/CD pipeline has been **re-enabled** with significant optimizations for GitHub free tier compliance (2,000 minutes/month limit).

**Resource-Optimized 3-Tier Architecture:**

### Pipeline Structure

1. **⚡ Lightning Validation** (4 minutes, 512MB)
   - Core test suite (`npm run test:core`)
   - ESLint validation (`npm run lint`)
   - TypeScript type checking (`npm run type-check`)
   - Essential quality gates for ALL changes

2. **🔍 Comprehensive Testing** (15 minutes, 512MB)
   - Integration tests (`npm run test:integration`)
   - Critical E2E smoke tests (Chromium only)
   - Console error detection
   - Broader functionality validation

3. **🏆 Quality Assurance** (25 minutes, 768MB) - Main branch only
   - Full coverage tests (`npm run test:coverage`)
   - Security audit (`npm audit --audit-level=high`)
   - Production build verification
   - Comprehensive validation suite

### Pipeline Quality Gates

- **Lightning Validation**: MUST pass for any PR to be merged (MANDATORY)
- **Core Tests**: 100% pass rate required
- **Type Safety**: Zero TypeScript errors allowed
- **Code Quality**: Zero ESLint errors allowed
- **Build Success**: Must build successfully for production
- **Resource Compliance**: Stay within 40% optimized usage of GitHub free tier

### Key Optimizations Applied

- **Memory Reduction**: 35% less memory usage (512-768MB vs 768-1024MB)
- **Selective Browser Installation**: Chromium-only for smoke tests
- **Smart Path Ignoring**: Skip CI for documentation-only changes
- **Aggressive Caching**: Dependencies, browsers, and build outputs
- **Single Worker E2E**: Prevent memory competition and OOM failures

**📋 For detailed optimization guide, see: `CI_OPTIMIZATION_GUIDE.md`**

## Testing Best Practices

### Mock Data Management

```typescript
// ✅ Use factory functions for consistent mock data
const createMockTeam = (overrides: Partial<Team> = {}): Team => ({
  id: 'team-id',
  name: 'Test Team',
  description: 'Test team description',
  type: 'permanent',
  status: 'active',
  divisionId: 'dev',
  capacity: 40,
  targetSkills: [],
  createdDate: '2024-01-01T00:00:00Z',
  lastModified: '2024-01-01T00:00:00Z',
  ...overrides,
});

// ✅ Validate mock data against interfaces
const mockTeam: Team = createMockTeam();
```

### Test Organization

```typescript
describe('ComponentName', () => {
  // Setup
  beforeEach(() => {
    vi.clearAllMocks();
  });

  // Happy path tests
  describe('when rendering with valid props', () => {
    it('should display component correctly', () => {
      // Test implementation
    });
  });

  // Edge case tests
  describe('when handling edge cases', () => {
    it('should handle empty data gracefully', () => {
      // Test implementation
    });
  });

  // Error case tests
  describe('when encountering errors', () => {
    it('should display error message', () => {
      // Test implementation
    });
  });

  // Interaction tests
  describe('when user interacts', () => {
    it('should respond to user actions', () => {
      // Test implementation
    });
  });
});
```

### Component Testing Patterns

```typescript
// ✅ Test component behavior, not implementation
it('should filter items when search term is entered', async () => {
  const user = userEvent.setup();
  render(<SearchableList items={mockItems} />);

  await user.type(screen.getByRole('textbox'), 'filter term');

  expect(screen.queryByText('Filtered Out Item')).not.toBeInTheDocument();
  expect(screen.getByText('Matching Item')).toBeInTheDocument();
});

// ✅ Test accessibility
it('should be accessible via keyboard navigation', async () => {
  const user = userEvent.setup();
  render(<NavigableComponent />);

  await user.tab();
  expect(document.activeElement).toBe(screen.getByRole('button'));
});

// ✅ Test error boundaries
it('should handle component errors gracefully', () => {
  const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});

  render(<ComponentWithError shouldError={true} />);

  expect(screen.getByText('Something went wrong')).toBeInTheDocument();
  consoleSpy.mockRestore();
});
```

## Deployment Standards

### Pre-Deployment Validation

- **Full Test Suite**: 100% pass rate on all core tests
- **Performance Benchmarks**: No degradation in key metrics
- **Security Scan**: Clean security audit results
- **Browser Compatibility**: Tested on all supported browsers
- **Accessibility**: WCAG 2.1 AA compliance verified

### Deployment Pipeline

1. **Feature Branch**: Development and initial testing
2. **Pull Request**: Code review and automated testing
3. **Staging**: Full integration testing and user acceptance
4. **Production**: Monitored rollout with rollback capability

## Documentation Standards

### Code Documentation

```typescript
/**
 * TimelineGanttView displays project timelines in a Gantt chart format
 *
 * @param teams - Array of team data for team view mode
 * @param allocations - Array of resource allocations to display
 * @param iterations - Array of iteration/cycle data for timeline range
 * @param selectedCycleId - ID of the currently selected cycle
 * @param viewMode - Display mode: 'teams' | 'epics' | 'projects'
 * @param onAllocationClick - Callback when allocation bar is clicked
 */
interface TimelineGanttViewProps {
  teams: Team[];
  allocations: Allocation[];
  iterations: Cycle[];
  selectedCycleId: string;
  viewMode?: TimelineViewMode;
  onAllocationClick?: (allocation: Allocation) => void;
}
```

### Test Documentation

```typescript
/**
 * Tests for TimelineGanttView component
 *
 * Covers:
 * - Basic rendering with all view modes
 * - Allocation bar display and interaction
 * - Timeline navigation and zoom controls
 * - Filtering and team selection
 * - Error handling for edge cases
 *
 * Mock data structure matches production TypeScript interfaces
 */
describe('TimelineGanttView', () => {
  // Test implementations
});
```

## Troubleshooting Guide

### Common Issues

1. **Test Failures After Interface Changes**
   - **Cause**: Mock data doesn't match updated TypeScript interfaces
   - **Solution**: Update all mock data to include required fields
   - **Prevention**: Use factory functions and interface validation

2. **Type Errors in Tests**
   - **Cause**: Test setup doesn't match component prop requirements
   - **Solution**: Ensure test props match component interface exactly
   - **Prevention**: Use TypeScript in test files and strict type checking

3. **Flaky Tests**
   - **Cause**: Async operations or timing issues
   - **Solution**: Use proper async/await patterns and user-event setup
   - **Prevention**: Avoid setTimeout, use waitFor and proper event handling

4. **Build Failures in CI**
   - **Cause**: Environment differences or missing dependencies
   - **Solution**: Ensure package-lock.json is committed and up to date
   - **Prevention**: Test locally in clean environment before push

5. **CI Memory Issues (Exit Code 137)**
   - **Cause**: Playwright tests consuming too much memory in constrained CI environment
   - **Solution**: Increase Node.js memory limits, reduce parallel workers, optimize test configurations
   - **Prevention**: Use memory-efficient test configurations, limit test scope in CI

### Performance Issues

1. **Slow Test Execution**
   - **Cause**: Inefficient test setup or too many DOM operations
   - **Solution**: Optimize test setup, use screen queries efficiently
   - **Prevention**: Keep tests focused and minimize DOM complexity

2. **Memory Leaks in Tests**
   - **Cause**: Event listeners or timers not cleaned up
   - **Solution**: Use cleanup functions and proper test teardown
   - **Prevention**: Always clean up side effects in beforeEach/afterEach

## Version Control Guidelines

### Commit Message Format

```
type(scope): brief description

Detailed explanation of changes made and why.

- Specific change 1
- Specific change 2

Fixes #issue-number
```

### Commit Types

- **feat**: New feature implementation
- **fix**: Bug fix
- **test**: Adding or updating tests
- **refactor**: Code refactoring without behavior change
- **perf**: Performance improvements
- **docs**: Documentation updates
- **style**: Code style changes (formatting, etc.)
- **ci**: CI/CD pipeline changes

### Branch Naming

- **feature/feature-name**: New feature development
- **fix/bug-description**: Bug fixes
- **test/test-description**: Test improvements
- **refactor/refactor-description**: Code refactoring
- **docs/documentation-update**: Documentation changes

## Security Guidelines

### Code Security

- **Input Validation**: Validate all user inputs and API responses
- **XSS Prevention**: Use proper escaping and sanitization
- **CSRF Protection**: Implement proper CSRF tokens for forms
- **Dependency Security**: Regular security audits of dependencies

### Testing Security

- **No Secrets in Tests**: Never commit API keys or credentials
- **Mock External Services**: Don't test against production services
- **Sanitize Test Data**: Ensure test data doesn't contain sensitive information

---

## Quick Reference Commands

```bash
# Development workflow
npm run dev                    # Start development server
npm run build                  # Build for production
npm run preview                # Preview production build

# Testing (MANDATORY before commits)
npm run test:core              # Core test suite (REQUIRED)
npm run test:coverage          # Full test suite with coverage
npm run test:integration       # Integration tests
npm run test:e2e               # End-to-end tests
npm run test:watch             # Watch mode for development

# Quality checks (MANDATORY before commits)
npm run lint                   # ESLint validation
npm run type-check             # TypeScript type checking
npm run format                 # Code formatting

# CI/CD simulation
npm run ci:quick               # Simulate quick validation track
npm run ci:full                # Simulate full CI pipeline
```

**Remember: Quality is not negotiable. Always follow TDD, test locally, and validate completely before any commit or push.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heymishy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/heymishy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
