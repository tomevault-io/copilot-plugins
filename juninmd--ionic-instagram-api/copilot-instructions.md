## ionic-instagram-api

> These guidelines are designed to ensure the consistent, high-quality, and maintainable development of AGENTS.md, a repository for AI coding agents.  Strict adherence to these principles is crucial for building a robust and scalable system.

```markdown
# AGENTS.md - AI Coding Agent Guidelines

These guidelines are designed to ensure the consistent, high-quality, and maintainable development of AGENTS.md, a repository for AI coding agents.  Strict adherence to these principles is crucial for building a robust and scalable system.

**1. DRY (Don't Repeat Yourself)**

*   **Code Reuse:**  All functionality, algorithms, and data structures should be encapsulated within reusable components.  Avoid duplicating code across multiple files.
*   **Centralized Logic:**  All core logic, algorithms, and data management should reside in a single, well-defined module or class.  
*   **Abstraction:**  Implement abstractions where possible to reduce code duplication and improve flexibility.

**2. KISS (Keep It Simple, Stupid)**

*   **Simplicity:**  Prioritize simple solutions over complex ones.  Assume the user understands the basic concepts and won't require extensive explanations.
*   **Readability:**  Code should be easy to understand, even by someone unfamiliar with the specific algorithm.  Use clear variable names, consistent indentation, and comments where necessary.
*   **Minimize Complexity:**  Avoid unnecessary features or operations.  Focus on the core requirements.

**3. SOLID Principles**

*   **Single Responsibility Principle:** Each class or module should have one primary responsibility.
*   **Open/Closed Principle:**  The system should be extensible without modifying existing code.  Add new functionality through new classes without altering the core logic.
*   **Liskov Substitution Principle:**  Subclasses should be substitutable for their base classes without altering the correctness of the program.
*   **Interface Segregation Principle:**  Clients shouldn't be forced to implement interfaces they don't use.
*   **Dependency Inversion Principle:**  High-level modules should not depend on low-level modules; they should depend on abstractions.

**4. YAGNI (You Aren't Gonna Need It)**

*   **Avoid Over-Engineering:** Don't implement features or logic that are not currently needed.  Refactor and simplify when appropriate.
*   **Focus on Requirements:**  Build functionality that directly addresses the defined requirements, not based on speculative assumptions.
*   **Progressive Development:**  Implement features in stages, testing and validating each stage before moving on.

**5. Development Workflow & Best Practices**

*   **Unit Testing:** All code must be thoroughly unit tested using a mocking framework. Test expectations clearly.
*   **Code Reviews:** All code changes must undergo rigorous code review by at least two team members.
*   **Documentation:**  Provide clear and concise documentation for each component, including inputs, outputs, and any assumptions made.
*   **Version Control:**  Use Git for version control and adhere to a strict branching strategy (e.g., Gitflow).
*   **Dependency Management:** Carefully manage dependencies using a reliable package manager (e.g., `pip`, `npm`).  Automate dependency updates.
*   **Logging:** Implement structured logging to aid in debugging and monitoring.  Log sufficient information to enable effective debugging.
*   **Error Handling:**  Handle errors gracefully and provide informative error messages. Implement recovery mechanisms where appropriate.

**6. Code Structure & File Size Constraints**

*   **Maximum Code Length:** Each file should have a maximum of 180 lines of code.
*   **File Organization:**  Files should be logically grouped by functionality or component. A file structure should be well-defined.
*   **Modularization:**  Break down large functions into smaller, well-defined modules.

**7.  Testing - Emphasis on Mocking**

*   **Mocking Framework:**  Utilize a robust mocking framework (e.g., `unittest.mock` in Python) for testing.
*   **Mocking Scope:** Mocking should be focused on isolating the component being tested.
*   **Test Coverage:** Target 80% test coverage.  Automated test suite is mandatory.

**8.  Coding Standards**

*   **Consistent Indentation:** Use 2 spaces for indentation.
*   **Meaningful Variable Names:** Use descriptive variable names.
*   **Comments:**  Add comments where necessary to explain complex logic.
*   **Error Handling:**  Implement comprehensive error handling and logging.

**9.  Repository Conventions**

*   **Naming Conventions:** Follow a consistent naming convention for files, functions, and variables.
*   **Directory Structure:**  Organize files into logical directories.
*   **README:**  Include a README file explaining the project's purpose, setup instructions, and usage.


These guidelines are subject to change as the project evolves. Regular reviews and updates are necessary to maintain the quality and consistency of AGENTS.md.  All developers are expected to understand and adhere to these principles.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juninmd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/juninmd)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
