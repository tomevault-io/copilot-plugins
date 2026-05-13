## 050-engineering-standards

> Enforces TDD (Red-Green-Refactor) and high engineering standards (SOLID, DRY, KISS, DDD)


# Engineering Standards & TDD Protocol

This rule enforces strict engineering standards and the Test-Driven Development (TDD) workflow for all code changes.

## 1. TDD Workflow (Red-Green-Refactor)

You MUST follow the Red-Green-Refactor cycle for every logical change:

### 🔴 RED: Write a Failing Test
1.  **Create/Update Test:** Write a test case that describes the expected behavior.
2.  **Verify Failure:** Ensure the test fails for the correct reason (e.g., missing class, method, or logic).
3.  **Do Not Implement Logic Yet:** Only write enough code to make the test compile (if necessary), but not pass.

### 🟢 GREEN: Make it Pass
1.  **Minimal Implementation:** Write the *minimum* amount of code required to make the test pass.
2.  **Ignore Perfection:** Do not worry about elegance or optimization at this stage. Just satisfy the test.
3.  **Verify Success:** Run the test to confirm it passes.

### 🔵 REFACTOR: Improve Code Quality
1.  **Clean Up:** Apply design patterns, remove duplication, and improve readability.
2.  **Apply Standards:** Ensure code adheres to SOLID, DRY, KISS, and GRASP principles.
3.  **Verify Safety:** Run tests again to ensure refactoring didn't break anything.

---

## 2. Architectural Principles

### 🏗️ Domain-Driven Design (DDD)
-   **Domain First:** Focus on the business logic and domain models, not the framework.
-   **Bounded Contexts:** Keep related logic together.
-   **Rich Models:** Prefer rich domain models over anemic ones (where appropriate).
-   **Services/Actions:** Encapsulate complex business logic in Services or Action classes.

### 🧱 SOLID Principles
-   **S**ingle Responsibility: Classes should have one reason to change.
-   **O**pen/Closed: Open for extension, closed for modification.
-   **L**iskov Substitution: Subtypes must be substitutable for base types.
-   **I**nterface Segregation: Many client-specific interfaces are better than one general-purpose interface.
-   **D**ependency Inversion: Depend on abstractions, not concretions.

### ✂️ DRY (Don't Repeat Yourself)
-   Extract common logic into traits, services, or helper methods.
-   Avoid duplicating knowledge or business rules.

### 💋 KISS (Keep It Simple, Stupid)
-   Avoid over-engineering.
-   Write code that is easy to understand and maintain.
-   Prefer simple solutions over complex ones unless necessary.

### 🤝 GRASP (General Responsibility Assignment Software Patterns)
-   **Controller:** Assign responsibility to the class representing the overall system or a use case scenario.
-   **Creator:** Who creates A? B creates A if B aggregates A.
-   **Information Expert:** Assign responsibility to the class that has the information needed to fulfill it.
-   **Low Coupling:** Keep dependency low.
-   **High Cohesion:** Keep related things together.

---

## 3. Laravel Specific Standards

### 🎮 Thin Controllers
-   **Controllers should be thin:** They should only handle HTTP requests/responses.
-   **Delegate Logic:** Move validation to FormRequests and business logic to Services/Actions/Jobs.
-   **No Queries in Controllers:** Use Repositories or Scopes for complex queries.

### 💉 Dependency Injection
-   Use Constructor Injection for dependencies.
-   Type-hint interfaces instead of concrete classes where possible.

### 🧪 Testing
-   **Feature Tests:** Use for integration/end-to-end scenarios (GWT pattern recommended).
-   **Unit Tests:** Use for isolated logic (AAA pattern recommended).
-   **Coverage:** Aim for high test coverage, especially for critical paths.

---

## 4. Code Smells to Avoid
-   **Long Methods:** Break them down.
-   **Large Classes:** Extract responsibilities.
-   **Primitive Obsession:** Use Value Objects.
-   **Feature Envy:** Move logic to the data owner.
-   **Magic Numbers/Strings:** Use constants or enums.

---

## 5. Enforcement

When generating code:
1.  **Start with the test.**
2.  **Show the failing test output (simulated or real).**
3.  **Implement the solution.**
4.  **Show the passing test output.**
5.  **Refactor and explain applied principles.**

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
