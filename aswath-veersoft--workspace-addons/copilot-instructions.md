## workspace-addons

> description: This rule explains Express.js conventions and best practices for Node.js backend development.

---
description: This rule explains Express.js conventions and best practices for Node.js backend development.
globs: **/*.js,**/*.ts
alwaysApply: false
---

# Express.js rules

- Use proper middleware order: body parsers, custom middleware, routes, error handlers
- Organize routes using Express Router for modular code structure
- Use async/await with proper error handling and try/catch blocks
- Create a centralized error handler middleware as the last middleware
- Use environment variables for configuration with a config module
- Implement request validation using libraries like express-validator
- Use middleware for authentication and authorization
- Use appropriate HTTP status codes in responses

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aswath-veersoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
