## data-engineering-workflow

> description: Error handling best practices for the chat-ui frontend built with Next.js and TypeScript.

---
description: Error handling best practices for the chat-ui frontend built with Next.js and TypeScript.
globs: chat-ui/**/*.tsx, chat-ui/**/*.ts
alwaysApply: false
---

# Error Handling Guidelines for Frontend (Chat-UI)
- Use `try/catch` blocks in asynchronous functions to handle errors gracefully.
- Display user-friendly error messages in the UI when necessary (e.g., "Something went wrong, please try again").
- Log errors to a monitoring service like Sentry or equivalent tools for debugging purposes.
- Avoid exposing sensitive error details to users; sanitize error messages before displaying them in the UI.
- Use TypeScript's type system to catch potential issues at compile time:
  - Define custom error types where applicable.
  - Use union types or enums for error codes/messages.
- Handle API errors by checking HTTP response codes and parsing server messages appropriately.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zeus-Deus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
