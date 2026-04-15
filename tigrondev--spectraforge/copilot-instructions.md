## spectraforge

> Test automation framework - Design patterns, best practices, AAA pattern enforcement


# Test Automation Framework for SpectraForge

@context {
  "type": "testing_guidelines",
  "purpose": "Guide the use of design patterns and best practices for test automation in C++ projects",
  "format_version": "2.0",
  "priority": 400,
  "applies_to": "All test files and automation scripts",
  "supported_patterns": ["AAA Pattern", "Factory", "Mock Objects", "Test Fixtures"],
  "frameworks": ["Google Test", "Catch2"],
  "ai_model": "Claude 4.5 Sonnet"
}

> **CLAUDE DIRECTIVE**: ALL tests MUST follow AAA pattern (Arrange, Act, Assert). NO exceptions.

## 1. Design Patterns Overview
Use these patterns to enhance code quality and maintainability.

@patterns_rules [
  {
    "id": "page_object_model",
    "rule": "Separate test code from UI actions using page classes.",
    "severity": "high",
    "example": "public class PageGoogle extends PageFactory { ... }"
  },
  {
    "id": "singleton",
    "rule": "Ensure single instance for shared resources like drivers.",
    "severity": "medium",
    "example": "public class SingletonDriver { private static WebDriver driver; ... }"
  },
  {
    "id": "factory",
    "rule": "Create objects dynamically based on input (e.g., browser type).",
    "severity": "medium",
    "example": "public class WebDriverFactory { public static WebDriver createDriver(String browserType) { ... } }"
  },
  {
    "id": "facade",
    "rule": "Simplify complex logic with a facade class.",
    "severity": "medium",
    "example": "public class PlaceOrderFacade { ... }"
  }
]

- Promote reusability and scalability.

## 2. Best Practices
Guidelines for effective implementation.

@best_practices_rules [
  {
    "id": "understand_problem",
    "rule": "Analyze requirements before choosing a pattern.",
    "severity": "high"
  },
  {
    "id": "keep_simple",
    "rule": "Avoid over-engineering; focus on efficiency.",
    "severity": "medium"
  },
  {
    "id": "test_refactor",
    "rule": "Regularly test and refactor code.",
    "severity": "high"
  }
]

## 3. Tables for Quick Reference

### Design Patterns Comparison
| Pattern | Category | Benefits | Use Case |
|---------|----------|----------|----------|
| POM | Structural | Reusability, maintainability | UI testing |
| Singleton | Creational | Resource control | Shared instances |
| Factory | Creational | Flexibility | Object creation |
| Facade | Structural | Simplification | Complex workflows |

## 4. C++ Specific Testing Patterns

@cpp_testing_patterns [
  {
    "id": "test_fixtures",
    "rule": "Use test fixtures for setup/teardown",
    "severity": "high",
    "example": "class MyTest : public ::testing::Test {\nprotected:\n  void SetUp() override { /* setup */ }\n  void TearDown() override { /* cleanup */ }\n};"
  },
  {
    "id": "mock_objects",
    "rule": "Mock dependencies using GoogleMock",
    "severity": "high",
    "example": "class MockRenderer : public IRenderer {\npublic:\n  MOCK_METHOD(void, render, (), (override));\n};"
  },
  {
    "id": "parameterized_tests",
    "rule": "Use parameterized tests for multiple inputs",
    "severity": "medium",
    "example": "INSTANTIATE_TEST_SUITE_P(MyTests, MyTestFixture, ::testing::Values(1, 2, 3));"
  }
]

## 5. Best Practices and Enforcement

- Adhere to maintainability and reusability principles
- Minimum 80% code coverage for all new code
- Tests MUST be independent and run in any order
- Use descriptive test names: TEST(ComponentName, WhatIsBeingTested)

---
**Version**: 2.0 | **Optimized for**: Claude 4.5 Sonnet

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TiGRoNdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TiGRoNdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
