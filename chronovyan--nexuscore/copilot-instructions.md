## nexuscore

> **Global Rule: Testing Strategies & Philosophy (Template)**

**Global Rule: Testing Strategies & Philosophy (Template)**

**Purpose:** To guide the AI in suggesting, generating, and discussing software tests in a manner that aligns with the user's preferred testing methodologies, quality standards, and overall testing strategy.

**Instructions for the AI:**
"When generating, reviewing, or suggesting tests, or when discussing testing strategies, you **must** adhere to the following principles. If the current project has specific testing guidelines or uses particular testing frameworks with established conventions, integrate those and prioritize them."

**1. Overall Testing Goals:**
    * "(User to specify, e.g.:
        * 'The primary goal of testing is to increase confidence in the correctness and reliability of the software.'
        * 'Tests should prevent regressions effectively.'
        * 'Tests serve as a form of living documentation for the code's behavior.'
        * 'Aim for a balanced test suite that provides good coverage without being excessively slow or brittle.')"

**2. Preferred Testing Methodologies:**
    * "(User to specify, e.g.:
        * **Test-Driven Development (TDD):** 'If I indicate a TDD approach, guide me through the red-green-refactor cycle. Help draft failing tests first, then the minimal code to pass, then suggest refactorings.'
        * **Behavior-Driven Development (BDD):** 'If using BDD, help formulate Gherkin-style scenarios (`Given-When-Then`) or similar user-story-focused tests. Focus on testing observable behavior.'
        * **Test-After Development:** 'If writing tests after implementation, help identify key functionalities and edge cases to cover. Suggest tests that verify existing behavior and protect against future regressions.'
        * 'No strong preference, but ensure comprehensive coverage as appropriate for the context.')"

**3. Test Pyramid & Scope:**
    * "(User to specify emphasis, e.g.:)
        * **Strong emphasis on Unit Tests:** 'A large base of fast, isolated unit tests is preferred. These should form the bulk of the test suite.'
        * **Balanced Approach:** 'Aim for a healthy mix of unit, integration, and end-to-end (E2E) tests. The number of tests should decrease as you move up the pyramid (fewer E2E than integration, fewer integration than unit).'
        * **Focus on Integration Tests:** 'For [specific type of project, e.g., API-driven services], robust integration tests are critical and should be prioritized alongside necessary unit tests.'"

**4. Unit Testing Specifics:**
    * **Definition:** "Unit tests should verify the smallest testable parts of an application, isolated from their dependencies."
    * **Scope:** "Focus on a single class, module, or function."
    * **Isolation:** "External dependencies (e.g., database, network, file system, other services) **must** be mocked, stubbed, or faked to ensure tests are fast, deterministic, and isolated. (User to specify preferred mocking/stubbing libraries or techniques for common languages, e.g., `unittest.mock` in Python, `Moq` for C#, `Mockito` for Java, `gMock` for C++)"
    * **Characteristics:** "Good unit tests are Fast, Independent/Isolated, Repeatable, Self-Validating, and Timely (FIRST principles)."

**5. Integration Testing Specifics:**
    * **Purpose:** "Integration tests verify the interactions between two or more components, modules, or services of the application (e.g., interaction with a database, communication between microservices, API endpoint behavior with its underlying logic)."
    * **Scope:** "Test the integration points and contracts between components."
    * **Managing Dependencies:** "(User to specify, e.g., 'For database integration tests, use a dedicated test database or in-memory alternatives if feasible.' 'For external service integration, consider using test doubles like fakes or stubs, or test against controlled test instances of those services.')"

**6. End-to-End (E2E) Testing Specifics:**
    * **Purpose:** "E2E tests validate the entire application flow from the user's perspective, simulating real user scenarios through the UI or API entry points."
    * **When to Use:** "Use sparingly for critical user paths due to their higher cost, slower execution, and potential flakiness."
    * **Scope:** "Cover key workflows and user journeys."
    * **Tools:** "(User to specify preferred E2E testing tools if any, e.g., Selenium, Cypress, Playwright, Puppeteer)."

**7. Test Structure & Organization:**
    * **Naming Conventions:**
        * "Test files: (User to specify, e.g., `test_*.py`, `*.spec.ts`, `*_test.go`, `ClassNameTests.cs`)"
        * "Test functions/methods: (User to specify, e.g., `test_should_do_x_when_y()`, `Should_DoX_When_Y()`, `GivenA_WhenB_ThenC()`, `it('should do x when y')`)"
    * **Arrangement:** "(User to specify preferred pattern, e.g.:
        * **Arrange-Act-Assert (AAA):** 'Structure tests clearly with distinct Arrange (setup), Act (execution), and Assert (verification) sections.'
        * **Given-When-Then (BDD style):** 'Use comments or structure to reflect this pattern within test methods if applicable.')"
    * **Readability:** "Tests should be clear, concise, and easy to understand. They act as documentation."

**8. Code Coverage:**
    * "(User to specify stance, e.g.:)
        * 'While 100% code coverage is not always a practical or meaningful goal, strive for high coverage (e.g., >80%) of critical business logic.'
        * 'Use coverage reports as a guide to identify untested areas, not as an absolute measure of test quality.'
        * 'Focus on testing behavior and outcomes rather than just line coverage.'"

**9. Test Data Management:**
    * "Avoid magic values in tests; use well-named constants or variables for test data."
    * "For complex test data, consider using test data builders or fixture generation mechanisms."
    * "Ensure test data is isolated and does not conflict between tests."

**10. Writing Testable Code:**
    * "When generating or refactoring application code, proactively consider its testability."
    * "Favor dependency injection, clear separation of concerns, and interfaces/abstractions to make code easier to isolate and test."
    * "Avoid static methods/classes with side effects or global state where possible, as they are harder to test in isolation."

"When asked to write tests, or when new application code is generated, suggest and create tests following these guidelines. If existing code lacks adequate tests, you may propose adding them."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Chronovyan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
