## cadrart2025-common

> - Separate words within a file name with hyphens (-). For example, a component named `EsfsUserProfileComponent` has a file name `user-profile.component.ts`.

# Angular and TypeScript Best Practices for ES-Form-System

## Naming conventions
- Separate words within a file name with hyphens (-). For example, a component named `EsfsUserProfileComponent` has a file name `user-profile.component.ts`.
- If the file contains more than one primary namable identifier, choose a name that describes the common theme to the code within. If the code in a file does not fit within a common theme or feature area, consider breaking the code up into different files.
- Use PascalCase for component and services names
- Use a noun or noun phrase to describe the component or service purpose
- Use a suffix to indicate the type of component (e.g., "Component", "Directive", "Pipe", "Service", etc.).
- Use camelCase for variable and functions names
- Use descriptive names that clearly convey the purpose of the variable or function
- Avoid single-letter variable names, except for loop counters
- Use consistent naming for related variables or functions
- Use a verb or verb phrase to describe the function’s purpose
- Interfaces start with an `I` (e.g. IEsfsConfig)
- Enums start with en `E` (e.g. EEsfsvalues)
- Private properties are prefixed with an underscore
- Generic types parameters are prefixed with T (e.g. `export class EsfsFieldComponent<TValue>`)

## Documentation
- The code should be self-explanatory thanks to clear organisation, naming conventions and concision
- Comment only when the code becomes complex and/or hard to read

## Coding principles
- Apply immutability principles and pure functions wherever possible, especially within services and state management, to ensure predictable outcomes and simplified debugging.
- Favor component composition over inheritance to enhance modularity, enabling reusability and easy maintenance.
- Define data models using interfaces for explicit types and maintain strict typing to avoid `any`.
- Structure files with imports at the top, followed by class definition, properties, methods, and ending with exports.
- Leverage optional chaining (`?.`) and nullish coalescing (`??`) to prevent null/undefined errors elegantly.
- Use standalone components as appropriate, promoting code reusability without relying on Angular modules.
- Use the `inject` function to inject services directly within component logic, directives, or services, reducing boilerplate code.
- Use single quotes (`'`) for string literals.
- Use 2-space indentation.
- Avoid trailing whitespace and unused variables.
- Prefer `const` for constants and immutable variables.
- Utilize template literals for string interpolation and multi-line strings.
- Use `async` pipe for observables in templates to simplify subscription management.
- Enable lazy loading for feature modules, optimizing initial load times.
- Ensure accessibility by using semantic HTML and relevant ARIA attributes.
- Use Angular's signals system for efficient reactive state management.
- For images, use `NgOptimizedImage` to improve loading and prevent broken links in case of failures.
- Implement deferrable views to delay rendering of non-essential components until they're needed.
- Always separate components, their template and stylesheet(s) in their own files

## Import Order
1. Angular core and common modules
2. RxJS modules
3. Angular-specific modules (e.g., `FormsModule`)
4. Core application imports
5. Shared module imports
6. Environment-specific imports (e.g., `environment.ts`)
7. Relative path imports

## Error Handling and Validation
- Apply robust error handling in services and components, using custom error types or error factories as needed.
- Implement validation through Angular's form validation system or custom validators where applicable.

## Testing and Code Quality
- Adhere to the Arrange-Act-Assert pattern for unit tests.
- Ensure high test coverage with well-defined unit tests for services, components, and utilities.

## Performance Optimization
- Apply pure pipes for computationally heavy operations, ensuring that recalculations occur only when inputs change.
- Avoid direct DOM manipulation by relying on Angular's templating engine.
- Leverage Angular's signals system to reduce unnecessary re-renders and optimize state handling.
- Use `NgOptimizedImage` for faster, more efficient image loading.

## Security Best Practices
- Prevent XSS by relying on Angular's built-in sanitization and avoiding `innerHTML`.
- Sanitize dynamic content using Angular's trusted sanitization methods to prevent vulnerabilities.

## Reference
Refer to Angular's official documentation for components, services, and modules to ensure best practices and maintain code quality and maintainability.

## General
- Follow `stylelintrc`, `eslintrc`, `editorconfig` and any other coding style rules found.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manuszep) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
