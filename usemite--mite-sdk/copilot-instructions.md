## mite-sdk

> MiteSDK is an SDK for bug reporting and release management.

# MiteSDK Development Guide

MiteSDK is an SDK for bug reporting and release management.

## Build Commands
- Run typecheck: `bun typescript` or `bun --filter="**" typescript`
- Clean build artifacts: `bun clean`
- Run example app: `cd example && bun start`
- Run tests: `cd example && bun test`
- Run linter: `cd package && bun lint`

## Code Style & Conventions
- **Formatting**: Using Biome with 2-space indentation, 90 char line width
- **Quotes**: Single quotes for JS/TS, double quotes for JSX
- **Semicolons**: Optional, only when needed
- **Imports**: Organized automatically, grouped by type
- **Types**: Always use TypeScript interfaces for object types, prefer type aliases for unions
- **Error Handling**: Use try/catch blocks with detailed error messages, prefix logs with `[Mite]`
- **Naming**:
  - Classes: PascalCase (e.g., `Mite`)
  - Variables/methods: camelCase
  - Interfaces: PascalCase with descriptive names (e.g., `MiteConfig`)
  - Private class members: Use `private` keyword
- **Comments**: Use JSDoc-style comments for public APIs, normal comments for implementation details
- **Testing**: Jest with snapshot testing for components

## Project Structure
- `/package`: Core SDK code
- `/example`: Demo React Native app using the SDK

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/usemite) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
