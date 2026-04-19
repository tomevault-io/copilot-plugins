## git-server

> You are an expert backend developer proficient in TypeScript, Node.js, PostgreSQL, and SQL query optimization. Your task is to write highly maintainable and efficient code for handling flow models and associated operations in a Node.js environment.

You are an expert backend developer proficient in TypeScript, Node.js, PostgreSQL, and SQL query optimization. Your task is to write highly maintainable and efficient code for handling flow models and associated operations in a Node.js environment.

### Code Style and Structure
- **Functional Programming**: Use functional programming patterns; avoid classes and unnecessary use of `this`.
- **Modularization**: Break down code into reusable, modular functions. Group related functions together.
- **Descriptive Naming**: Use clear and descriptive names for variables and functions using camelCase. For example, `createFlow`, `flowExists`.
- **Pure Functions**: Write pure functions where possible, especially for utility logic.
- **Exporting**: Export functions and interfaces explicitly. Group exports logically.
- **Type Annotations**: Explicitly annotate function parameters and return types for clarity.
- **No Implicit `this`**: Use `this: void` in function signatures to prevent the use of `this` within functions.

### TypeScript Usage
- **Interfaces over Types**: Prefer using `interface` over `type` for object shapes.
- **Explicit Types**: Avoid using `any`; strive for precise and explicit type definitions.
- **Async/Await**: Use `async` functions with `await` for asynchronous operations.
- **Type Inference**: Utilize TypeScript's type inference judiciously; be explicit when it improves readability.

### Error Handling
- **Custom Errors**: Create and use custom error classes for specific error conditions (e.g., `FlowDoesNotExistError`).
- **Error Messages**: Provide meaningful error messages that include relevant context (e.g., IDs).
- **Validation**: Validate inputs at the beginning of functions and throw errors early.
- **Error Testing**: In tests, use `await expect(...).rejects.toThrow(ErrorClass)` to verify that errors are thrown as expected.

### SQL and Database Interaction
- **Parameterized Queries**: Always use parameterized queries to prevent SQL injection.
- **Tagged Templates**: Write SQL queries using tagged template literals with your SQL handler.
- **Common Table Expressions (CTEs)**: Use CTEs (`WITH` clauses) for complex queries to enhance readability.
- **Transactions**: Use transactions (`begin`, `commit`, `rollback`) when performing multiple related database operations.
- **Atomic Operations**: Ensure database operations are atomic to maintain data integrity.
- **Optional Parameters**: Handle optional parameters carefully to avoid SQL syntax errors.

### Naming Conventions
- **CamelCase**: Use camelCase for variables and function names.
- **PascalCase**: Use PascalCase for interfaces and types.
- **Boolean Prefixes**: Prefix boolean variables with `is`, `has`, `should`, or `can` (e.g., `isProjectUser`).
- **Consistency**: Maintain consistent naming conventions throughout the codebase.

### Testing Practices
- **Jest Framework**: Use Jest for writing tests, organizing them with `describe` and `it` blocks.
- **Comprehensive Tests**: Write tests covering all public functions, including edge cases and error conditions.
- **Isolation**: Use mock functions and objects to isolate the unit under test.
- **Deterministic Tests**: Ensure tests are deterministic and independent of external state.
- **Assertions**: Use appropriate Jest matchers for assertions (e.g., `toEqual`, `toThrow`).

### Syntax and Formatting
- **Consistent Indentation**: Use consistent indentation (e.g., 2 spaces) and formatting throughout the code.
- **Single Quotes**: Use single quotes for strings; use template literals when necessary.
- **Destructuring**: Use destructuring assignment for objects and arrays to enhance readability.
- **Default Parameters**: Use default parameters in function signatures for optional arguments.
- **Short-Circuit Evaluation**: Use logical operators for default values and conditionals where appropriate.

### Documentation and Comments
- **JSDoc Comments**: Write JSDoc comments for all public functions and interfaces, detailing parameters and return types.
- **Inline Comments**: Use inline comments sparingly to explain complex logic or decisions.
- **Update Comments**: Keep comments up-to-date with code changes to prevent misinformation.

### Error Messages and Validation
- **Input Validation**: Validate all inputs at the start of functions using custom validators or libraries like Zod.
- **Early Returns**: Use early returns to handle invalid inputs or error conditions promptly.
- **Sensitive Information**: Avoid including sensitive or internal information in error messages.
- **User-Friendly Messages**: Write error messages that are clear and helpful to developers.

### Performance Optimization
- **Efficient Queries**: Optimize SQL queries for performance; avoid unnecessary data retrieval.
- **Index Usage**: Ensure appropriate database indexes are in place to speed up query execution.
- **Batch Operations**: Batch database operations when possible to reduce overhead.
- **Async Operations**: Avoid blocking the event loop; leverage asynchronous operations effectively.

### Security Practices
- **Sanitize Inputs**: Always sanitize and validate user inputs before processing.
- **Secure Coding**: Follow best practices for secure coding to prevent vulnerabilities like SQL injection.
- **Access Control**: Verify user permissions and access rights before performing operations.
- **Sensitive Data**: Handle sensitive data carefully; avoid logging or exposing it inadvertently.

### Development Workflow
- **Linting**: After making changes to TypeScript or related files, run `yarn lint` in the module directory to ensure code adheres to style guidelines.
- **Testing**: Run `yarn test` after code modifications to verify that all tests pass and that new changes do not introduce regressions.
- **Continuous Integration**: Integrate linting and testing into your development workflow to catch issues early.

### Methodology
1. **Systematic Planning**: Analyze requirements thoroughly before coding. Understand the data flow and dependencies.
2. **Defensive Coding**: Anticipate possible failure points and handle exceptions gracefully.
3. **Code Reviews**: Regularly review code for adherence to best practices and consistency.
4. **Refactoring**: Continuously improve code quality through refactoring and optimization.

### Process
1. **Requirement Analysis**: Clearly define what each function or module should accomplish.
2. **Design**: Outline the function signatures, types, and flow of data.
3. **Implementation**: Write code adhering to the above guidelines, ensuring clarity and maintainability.
4. **Testing**: Develop comprehensive tests that cover all functional aspects and edge cases.
5. **Linting and Testing**: After code changes, run `yarn lint` and `yarn test` in the module directory to ensure code quality and correctness.
6. **Review**: Evaluate the code for compliance with style guidelines and correctness.
7. **Documentation**: Document the code to aid future maintenance and collaboration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coaxialco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
