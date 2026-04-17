## sn-troubleshooting-kb

> Core best practices for Node.js and TypeScript development, focusing on maintainability and common pitfalls.


# Node.js & TypeScript Core Best Practices

This rule set guides the AI in generating robust, performant, and type-safe Node.js applications using TypeScript.

## 1. Type Safety and Strictness

* **Always** use TypeScript's strong typing features to prevent runtime errors and ensure data integrity.
* Define explicit types and interfaces for all data structures, function parameters, and return values.
* Enable strict type checking in `tsconfig.json` (`"strict": true`).
* Prefer `interface` for defining object shapes and `type` for aliases or union types.

## 2. Asynchronous Programming

* **Always** use `async/await` for asynchronous operations. Avoid callback hell.
* Handle promises correctly with `try...catch` blocks for error management.
* Utilize `Promise.all` or `Promise.allSettled` for parallel asynchronous operations when appropriate.

## 3. Error Handling

* Implement centralized error handling using middleware (e.g., in Express.js) to catch unhandled exceptions and promise rejections.
* Distinguish between operational errors (e.g., invalid input) and programming errors (e.g., bugs).
* **Never** expose sensitive error details (stack traces, database errors) to client-facing APIs. Log them securely.

## 4. Module System

* Use ES Modules (`import/export`) over CommonJS (`require/module.exports`) for new codebases.
* Organize code into small, focused modules with clear responsibilities.

## 5. Environment Variables

* **Always** load configuration from environment variables (e.g., using `process.env`).
* Use a library like `dotenv` for local development, but ensure proper secret management in production (e.g., Kubernetes Secrets, AWS Secrets Manager).
* Define a clear schema for environment variables using a validation library (e.g., `Zod`, `Joi`, `class-validator`).

## 6. Performance Considerations

* Favor non-blocking I/O operations.
* Avoid CPU-bound tasks in the main event loop; consider worker threads for heavy computation.
* Implement caching strategies where appropriate (e.g., Redis).

## 7. Dependency Management

* Keep `package.json` dependencies and devDependencies up-to-date.
* Use `npm audit` or `yarn audit` regularly to check for known vulnerabilities.
* Pin specific versions or use lock files (`package-lock.json`, `yarn.lock`) to ensure reproducible builds.
* **Never** include unnecessary packages.

## 8. Linting and Formatting

* Configure ESLint and Prettier for consistent code style and to enforce best practices.
* Use `eslint-plugin-security` to catch common security issues during development.
* Integrate linting into pre-commit hooks (e.g., with Husky and lint-staged).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mnichols-snyk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
