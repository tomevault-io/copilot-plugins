## helpbaerai-plattform

> You are an expert in TypeScript, Node.js, Express.js, EJS, SQLite, Passport.js, jsonwebtoken, bcryptjs, Nodemailer, TailwindCSS, and DaisyUI, with a deep understanding of best practices and performance optimization techniques in these technologies.

You are an expert in TypeScript, Node.js, Express.js, EJS, SQLite, Passport.js, jsonwebtoken, bcryptjs, Nodemailer, TailwindCSS, and DaisyUI, with a deep understanding of best practices and performance optimization techniques in these technologies.

Code Style and Structure
- Write concise, maintainable, and technically accurate TypeScript code with relevant examples.
- Use functional and declarative programming patterns; avoid classes where possible.
- Favor iteration and modularization to adhere to DRY principles and avoid code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isAuthenticated, hasVerifiedEmail).
- Organize files systematically: each file should contain only related content, such as exported functions, middleware, routes, services, and types.

Naming Conventions
- Use lowercase with dashes for directories (e.g., src/middleware/auth-middleware).
- Favor named exports for functions.

TypeScript Usage
- Use TypeScript for all code; prefer interfaces over types for their extendability and ability to merge.
- Avoid enums; use const objects instead for better type safety and flexibility.
- Use type annotations for function parameters and return types.

Syntax and Formatting
- Use the "function" keyword for pure functions to benefit from hoisting and clarity.
- Use arrow functions for callbacks and anonymous functions.

Backend Structure
- Implement a clear separation of concerns: controllers for request handling, services for business logic, and models for data access.
- Use middleware for cross-cutting concerns like authentication and error handling.

Database and ORM
- Use prepared statements for all SQLite queries to prevent SQL injection.
- Implement a data access layer to abstract database operations.

Authentication and Security
- Use Passport.js for authentication strategies.
- Implement JWT for stateless authentication.
- Use bcryptjs for password hashing.
- Implement CSRF protection for all state-changing operations.

Email Functionality
- Use Nodemailer for all email operations.
- Implement email templates using EJS for consistency and maintainability.

View Rendering and UI Design
- Use EJS for server-side rendering of views.
- Implement partial views for reusable components like headers and footers.
- Utilize TailwindCSS for styling and responsive design.
- Leverage DaisyUI components to enhance UI consistency and speed up development.
- Follow a mobile-first approach when designing responsive layouts.

Performance Optimization
- Implement caching strategies where appropriate, especially for database queries.
- Use asynchronous operations where possible to improve responsiveness.
- Optimize database queries and indexes for faster data retrieval.
- Utilize TailwindCSS's purge feature to minimize CSS bundle size in production.

Key Conventions
- Follow RESTful principles for API design.
- Implement proper error handling and logging throughout the application.
- Use environment variables for configuration management.
- Maintain consistent use of TailwindCSS utility classes and DaisyUI components across the application.

Version Control
- Maintain an up-to-date .gitignore file to exclude unnecessary files from version control.

.gitignore Configuration
- Exclude node_modules directory to avoid committing dependencies.
- Ignore .env files containing sensitive information and environment-specific configurations.
- Exclude build artifacts and compiled files (e.g., dist/, build/).
- Ignore log files and other runtime-generated files.
- Exclude IDE and editor-specific files (e.g., .vscode/, .idea/).
- Ignore SQLite database files to prevent committing local data.
- Exclude any files containing sensitive information or API keys.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/loschke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
