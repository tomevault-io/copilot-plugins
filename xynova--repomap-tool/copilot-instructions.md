## rules-for-testing

> Comprehensive testing and code quality rules with anti-cheating protocols


# 🤖 **AI REASONING PROTOCOL**

## **Before Taking Any Testing Action:**
**ALWAYS provide step-by-step reasoning that:**
1. States what testing action you're about to take
2. Explains which rules from this document apply
3. Shows how you've considered those rules in your approach
4. Outlines your specific plan and next steps

---

# 🚫 **FORBIDDEN ACTIONS - ANTI-CHEATING PROTOCOLS**

## **Testing Anti-Cheating Rules**
- **DO NOT** create tests that mock core functionality to achieve coverage
- **DO NOT** write tests that only verify return types without testing behavior
- **DO NOT** create tests with hardcoded responses or fake data
- **DO NOT** write tests that only cover happy paths without error conditions
- **DO NOT** create trivial tests that test obvious functionality
- **DO NOT** use mocks for internal business logic (only for external dependencies)

## **Code Quality Anti-Cheating Rules**
- **DO NOT** refactor by simply moving code around without real separation of concerns
- **DO NOT** create modules with artificial separation (e.g., by line count only)
- **DO NOT** implement placeholder logic that returns hardcoded values
- **DO NOT** create fake implementations that don't perform real business logic
- **DO NOT** use deprecated patterns without updating to modern alternatives
- **DO NOT** create modules that can't be tested independently

## **Integration Testing Anti-Cheating Rules**
- **DO NOT** create integration tests that mock everything
- **DO NOT** write integration tests that only verify CLI return codes
- **DO NOT** create integration tests with fake file system operations
- **DO NOT** write integration tests that skip error recovery scenarios
- **DO NOT** create integration tests that don't validate real system behavior

# ✅ **REQUIRED ACTIONS - QUALITY PROTOCOLS**

## 🧪 **TESTING REQUIREMENTS:**
- **ALWAYS** write comprehensive tests for new functionality
- **NEVER** claim tests pass without actually running them
- **NEVER** provide fake test solutions or incomplete test coverage
- **ALWAYS** verify tests run successfully with `make test-unit`
- **ALWAYS** fix failing tests before claiming success
- **ALWAYS** test edge cases and error conditions
- **ALWAYS** ensure test coverage for new code paths
- **CLI TESTS ARE MANDATORY** - CLI is the primary UX, must be thoroughly tested
- **ALWAYS** write tests that use real data and files when possible
- **ALWAYS** include error condition testing (at least 20% of tests)
- **ALWAYS** verify actual behavior, not just return types
- **ALWAYS** include performance constraints in relevant tests
- **ALWAYS** test with real project structures, not artificial ones

## 📁 **FIXTURE REQUIREMENTS:**
- **PREFER static shared fixtures** over dynamic test data generation
- **USE existing test fixtures** from `tests/fixtures/` directory
- **AVOID state-changing operations** in test fixtures (this software doesn't change state)
- **REUSE fixture data** across multiple test cases for consistency
- **CREATE static test files** that represent real project structures
- **USE shared example data** that can be referenced by multiple tests
- **AVOID generating test data** at runtime when static data suffices
- **PREFER deterministic fixtures** that produce consistent results
- **USE existing test repositories** like `tests/fixtures/test-repo/` for integration tests

## 🔍 TESTING WORKFLOW:
1. **Write tests first** for new functionality
2. **Run tests immediately** after any code changes
3. **Fix all failures** before proceeding
4. **Verify end-to-end** functionality works
5. **Never skip testing** - it's mandatory, not optional

## 🐛 **DEBUG TEST NAMING CONVENTIONS:**
- **ALWAYS** prefix debug/investigation tests with `ignore.test_*` 
- **ALWAYS** use this naming pattern for temporary tests created to debug issues
- **ALWAYS** ensure debug tests don't get captured in git commits
- **EXAMPLES**: `ignore.test_debug_import_issue.py`, `ignore.test_investigate_parser.py`
- **PURPOSE**: Keep formal test suite clean while allowing debugging investigations

## ⚠️ **TESTING RULES:**
- **NO FAKE SOLUTIONS**: Every test must actually pass
- **NO SKIPPED TESTS**: All tests must run and pass
- **NO PARTIAL COVERAGE**: Test all new code paths
- **NO ASSUMPTIONS**: Verify everything works, don't guess
- **NO CLI WITHOUT TESTS**: Every CLI command must have comprehensive tests
- **NO FALSE CLAIMS**: Never claim 100% coverage without verifying actual numbers
- **NO BS STATEMENTS**: Always be precise about what is actually tested vs. claimed
- **NO OVER-MOCKING**: Tests must use real data and validate actual behavior
- **NO PLACEHOLDER IMPLEMENTATIONS**: Business logic must be real, not fake
- **NO SUPERFICIAL REFACTORING**: Modules must have real separation of concerns

## 🚀 FINAL VALIDATION REQUIREMENTS:
- **ALWAYS** run `make ci` before finishing work
- **ALWAYS** fix any CI errors before claiming completion
- **ALWAYS** ensure good real coverage for all updates (>80% for new code)
- **ALWAYS** verify integration tests pass (not just unit tests)
- **ALWAYS** check that new functionality works end-to-end
- **NEVER** finish work with failing CI or poor coverage

## 🖥️ CLI TESTING REQUIREMENTS:
- **CLI IS THE UX**: Every CLI command must be thoroughly tested
- **Command existence**: Test that all commands are registered and show help
- **Option validation**: Test all command-line options and constraints
- **Output formats**: Verify JSON, table, and text output work correctly
- **Error handling**: Test invalid inputs, missing files, and error conditions
- **Configuration integration**: Ensure CLI uses configuration correctly
- **Real execution**: Use `click.testing.CliRunner` for actual CLI testing
- **Progress handling**: Test with progress bars and ANSI escape codes
- **Exit codes**: Verify correct exit codes for success/error conditions
- **Edge cases**: Test boundary conditions and user error scenarios

## 📊 COVERAGE REQUIREMENTS:
- **New functionality**: Must have >70% test coverage
- **Modified code**: Must maintain or improve existing coverage
- **Integration points**: Must be tested with real scenarios
- **Edge cases**: Must be covered for all new features
- **Error conditions**: Must be tested for robustness
- **CLI commands**: Must have 100% test coverage - no exceptions

## 🎯 **COVERAGE ACCURACY RULES:**
- **ALWAYS verify actual coverage numbers** before making claims
- **NEVER claim 100% coverage** without running coverage tools
- **BE SPECIFIC** about what is tested vs. what is claimed
- **DISTINGUISH** between "new functionality coverage" and "overall module coverage"
- **USE EXACT NUMBERS** when reporting coverage (e.g., "33% overall, 100% for new features")

## 📊 **SUCCESS METRICS - NEGATIVE APPROACH**

### **Test Coverage Quality Gates**
- **Integration test ratio**: ≥30% of new tests must be integration tests
- **Error condition coverage**: ≥20% of new tests must test error conditions
- **Real data usage**: ≥80% of new tests must use real data
- **Mock usage**: ≤20% of new tests (only for external dependencies)
- **Performance testing**: ≥15% of new tests must include performance constraints

### **Code Quality Gates**
- **Module size**: ≤300 lines per module (no exceptions)
- **Single responsibility**: ≥80% of modules must have single responsibility
- **Independent testability**: ≥90% of modules must be testable independently
- **Real implementation**: 0 placeholder or fake implementations
- **Modern patterns**: 0 deprecated patterns remaining

### **Integration Test Quality Gates**
- **E2E coverage**: ≥80% of workflows must have end-to-end tests
- **Error coverage**: ≥30% of tests must cover error conditions
- **Real system behavior**: ≥70% of tests must validate real system behavior
- **Performance validation**: ≥20% of tests must include performance metrics
- **Persistence validation**: ≥50% of tests must validate real persistence

## 🚨 **VALIDATION PROTOCOLS**

### **Before Claiming Success - Verify:**
1. **No over-mocking**: Check that tests use real data and don't mock core functionality
2. **No fake implementations**: Verify that business logic is real, not placeholder
3. **No superficial refactoring**: Ensure modules have real separation of concerns
4. **No deprecated patterns**: Confirm modern patterns are used throughout
5. **No trivial tests**: Verify tests cover meaningful scenarios, not obvious functionality
6. **No missing error conditions**: Ensure error handling is comprehensive

### **Quality Validation Checklist**
- [ ] Tests use real data/files, not mocks
- [ ] Tests cover error conditions and edge cases
- [ ] Tests verify actual behavior, not just return types
- [ ] Modules have single responsibility and can be tested independently
- [ ] Implementations perform real business logic, not placeholder functions
- [ ] Integration tests validate real system behavior and persistence
- [ ] Performance considerations are included where relevant
- [ ] Modern patterns are used, deprecated patterns are eliminated

## 🔍 **ANTI-CHEATING DETECTION**

### **Red Flags to Watch For:**
- Tests that mock everything to achieve coverage
- Implementations that always return the same hardcoded value
- Refactoring that just moves code without real separation
- Integration tests that only verify CLI return codes
- Modules that can't be tested independently
- Business logic that doesn't perform real calculations
- Tests that only cover happy paths
- Code that uses deprecated patterns without updating

### **Quality Indicators:**
- Tests that use real project files and data
- Implementations that perform actual calculations and analysis
- Modules with clear, single responsibilities
- Integration tests that validate real system behavior
- Error handling that covers multiple failure scenarios
- Performance testing with real constraints
- Modern patterns and up-to-date dependencies

## 🚨 **FAILURE CONDITIONS**

**Any work FAILS if:**
- Tests are over-mocked and don't test real functionality
- Implementations are fake or placeholder
- Refactoring is superficial without real separation
- Integration tests don't validate real system behavior
- Error conditions are not properly covered
- Performance considerations are ignored
- Deprecated patterns are not updated
- Modules can't be tested independently

## 💡 **SUCCESS PATTERNS**

**Work SUCCEEDS when:**
- Tests use real data and validate actual behavior
- Implementations perform real business logic
- Modules have clear, single responsibilities
- Integration tests validate real system behavior
- Error handling is comprehensive
- Performance is considered and tested
- Modern patterns are used throughout
- All modules can be tested independently

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
