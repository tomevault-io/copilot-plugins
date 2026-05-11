## bytesurgeon

> This document outlines the essential commands and coding conventions for contributing to the ByteSurgeon project. Adhering to these guidelines ensures consistency, maintainability, and high quality throughout the codebase.

# Agent Guidelines for ByteSurgeon

This document outlines the essential commands and coding conventions for contributing to the ByteSurgeon project. Adhering to these guidelines ensures consistency, maintainability, and high quality throughout the codebase.

## 1. Build, Lint, and Test Commands

ByteSurgeon is a Gradle-based Java/Kotlin project. All build, lint, and test operations are managed via Gradle.

### 1.1 Build Commands

*   **Clean and Build the Project**:
    ```bash
    ./gradlew clean build
    ```
    This command cleans the build directory and compiles all modules.

*   **Run the Main Application (if applicable)**:
    ```bash
    ./gradlew clean run
    ```
    As per the `README.md`, this command is used for running the project.

### 1.2 Linting

While no explicit linting configuration files were found, adherence to the Google Java Style Guide (see Section 2.1) is expected. Automated checks can be integrated via Gradle plugins like Spotless if needed in the future.

### 1.3 Test Commands

ByteSurgeon uses JUnit 5 for testing.

*   **Run All Tests**:
    ```bash
    ./gradlew test
    ```
    This command executes all unit tests across all modules.

*   **Run All Tests for a Specific Module**:
    To run tests for a specific module (e.g., `agent`), use:
    ```bash
    ./gradlew :agent:test
    ```

*   **Run a Single Test Class**:
    To run tests within a specific test class (e.g., `com.bytesurgeon.agent.MyAgentTest` in the `agent` module), use:
    ```bash
    ./gradlew :agent:test --tests "com.bytesurgeon.agent.MyAgentTest"
    ```

*   **Run a Single Test Method**:
    To run a specific test method within a test class (e.g., `myTestMethod` in `com.bytesurgeon.agent.MyAgentTest`), use:
    ```bash
    ./gradlew :agent:test --tests "com.bytesurgeon.agent.MyAgentTest.myTestMethod"
    ```
    Note: Replace `:agent:` with the appropriate module name and adjust the package and class names as necessary.

## 2. Code Style Guidelines

The ByteSurgeon project strictly adheres to the **Google Java Style Guide**. All contributions must conform to these standards.

### 2.1 General Principles (Google Java Style)

Familiarize yourself with the full Google Java Style Guide for comprehensive details. Key aspects include:
*   **Formatting**: Consistent indentation (2 spaces), brace style, line wrapping, and whitespace usage.
*   **Naming Conventions**: Specific rules for packages, classes, methods, variables, and constants (e.g., `CamelCase` for classes, `camelCase` for methods/variables, `SCREAMING_SNAKE_CASE` for constants).
*   **Comments**: Use Javadoc for public classes and methods; block comments for complex logic; end-of-line comments sparingly.
*   **Structure**: Logical organization of class members, clear separation of concerns.

### 2.2 Imports

*   **Ordering**: Imports should be grouped and ordered as follows:
    1.  Static imports
    2.  `java` and `javax` packages
    3.  Third-party libraries
    4.  Project-specific packages (e.g., `com.bytesurgeon.*`)
*   **Wildcards**: Avoid wildcard imports (e.g., `import java.util.*`) except for specific, approved cases (e.g., `java.util.concurrent.*` in certain contexts).
*   **Unused Imports**: Remove all unused imports.

### 2.3 Formatting

*   **Indentation**: Use 2 spaces for indentation, never tabs.
*   **Line Length**: Aim for a maximum line length of 100 characters, with 120 as a hard limit for readability.
*   **Brace Style**: Opening braces are on the same line as the declaration, and closing braces are on their own line.
    ```java
    if (condition) {
      // ...
    }
    ```
*   **Whitespace**: Use whitespace consistently around operators, keywords, and method parameters.

### 2.4 Types and Generics

*   **Type Inference**: Use `var` judiciously where it improves readability and the type is obvious from the right-hand side.
*   **Generics**: Use generics to ensure type safety and avoid raw types. Provide type arguments explicitly or use the diamond operator (`<>`) when appropriate.

### 2.5 Naming Conventions

*   **Packages**: All lowercase, words separated by dots (e.g., `com.bytesurgeon.agent`).
*   **Classes/Interfaces/Enums**: PascalCase (e.g., `MyAgent`, `PerformanceMonitor`).
*   **Methods**: camelCase (e.g., `instrumentMethod`, `getMetrics`).
*   **Variables**: camelCase (e.g., `startTime`, `methodName`).
*   **Constants**: SCREAMING_SNAKE_CASE (e.g., `DEFAULT_TIMEOUT`, `MAX_RETRIES`).

### 2.6 Error Handling

*   **Exceptions**: Use checked exceptions for recoverable errors and unchecked exceptions (runtime exceptions) for programming errors or unrecoverable conditions.
*   **Logging**: Use a consistent logging framework (e.g., SLF4J with Logback/Log4j2) for reporting errors, warnings, and debug information. Avoid printing directly to `System.out` or `System.err`.
*   **Specific Errors**: Catch specific exceptions rather than broad `Exception` or `Throwable` to handle errors precisely.

## 3. Cursor/Copilot Rules

No specific Cursor or Copilot configuration files (`.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md`) were found in this repository. Agents should infer context and adhere to the general code style guidelines mentioned above.

---
> Source: [amrxtgh69/ByteSurgeon](https://github.com/amrxtgh69/ByteSurgeon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
