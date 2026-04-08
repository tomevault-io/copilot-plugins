## magicappdev

> This document provides detailed documentation for each package in the MagicAppDev project, covering architecture, dependencies, usage guidelines, and integration points. MagicAppDev is a comprehensive monorepo-based fullstack app building platform inspired by Expo, utilizing Turborepo for efficient monorepo management, enabling rapid development, customization, and deployment of apps across web and mobile with no-code capabilities.

# Agents.md - Comprehensive Documentation

## Overview

This document provides detailed documentation for each package in the MagicAppDev project, covering architecture, dependencies, usage guidelines, and integration points. MagicAppDev is a comprehensive monorepo-based fullstack app building platform inspired by Expo, utilizing Turborepo for efficient monorepo management, enabling rapid development, customization, and deployment of apps across web and mobile with no-code capabilities.

## Mobile Repo Boundary

- Treat `apps/mobile` as its own independent repo/workflow surface inside this repository.
- When a task is scoped to mobile, interpret paths, commands, and docs relative to `apps/mobile` unless the task explicitly says it spans the root workspace.
- Mobile GitHub Actions live under `apps/mobile/.github/workflows/` in this repository. When operating as the mobile repo, treat that directory as the workflow root rather than the monorepo root `.github/workflows/`.
- Do not assume root-only commands, root `.env` files, or root workflow paths apply to mobile-only work.

## Tech Stack Overview

- **Monorepo Management**: Turborepo with pnpm for package management, including local/remote caching
- **Backend**: Cloudflare Workers with Hono framework
- **Database**: Cloudflare D1 with Drizzle ORM
- **Frontend (Web)**: Next.js with TypeScript
- **Frontend (Mobile)**: Ionic with Capacitor
- **Authentication**: GitHub OAuth with JWT session management
- **AI Integration**: Cloudflare AI Gateway for optimized model access
- **Build System**: Hybrid Nx + Turborepo setup (in progress of consolidation)
- **Testing**: Vitest for unit tests, Playwright for E2E tests (planned)
- **Deployment**: Cloudflare Workers platform with GitHub Actions CI/CD

## Discord Bot Configuration

For the community Discord server, configure the following bot settings:

- **Interactions Endpoint URL**: <https://nice-example.local/api/interactions>
  - Used to receive interactions via HTTP POSTs rather than over Gateway with a bot user.
- **Linked Roles Verification URL**: <https://nice-example.local/verify-user>
  - Configured to enable the application as a requirement in a server role's Links settings.
- **Terms of Service URL**: <https://my-cool-app.com/terms-of-service>
  - A link to the application's Terms of Service.
- **Privacy Policy URL**: <https://my-cool-app.com/privacy-policy>
  - A link to the application's Privacy Policy.

## Packages

### 1. @magicappdev/database

#### Description

Database utilities with Drizzle ORM for Cloudflare D1, providing the foundation for data persistence across all applications.

#### Architecture

- **ORM**: Drizzle ORM
- **Database**: Cloudflare D1
- **Schema Management**: Drizzle Kit
- **Tables**: `users`, `accounts` (OAuth), `sessions`, `profiles`, `projects`, `admin-api-keys`

#### Current Status (January 24, 2026)

- ✅ Database schema fully implemented with all core tables
- ✅ Migration system in place for both local development and production
- ✅ Integration with Cloudflare D1 completed
- ✅ Admin API keys schema added for enhanced security

#### Usage Guidelines

- Use Drizzle ORM for database operations.
- Ensure proper schema management using Drizzle Kit (`pnpm generate`).
- Use `migrate:prod` to apply migrations to Cloudflare D1.
- Relations are currently handled manually via joined queries in the API to ensure stability in the Worker environment.

#### Next Steps

- [ ] Implement proper relation handling in Drizzle ORM
- [ ] Add database seeding utilities
- [ ] Implement database backup and recovery procedures

### 2. @magicappdev/shared

#### Description

Shared utilities, types, and constants for MagicAppDev, providing a consistent foundation across all packages and applications.

#### Architecture

- **API Client**: Unified `ApiClient` with support for JWT authentication, token refreshing, and Server-Sent Events (SSE) streaming.
- **Types**: Unified application types (User, Project, AiMessage) and API request/response contracts.
- **Utilities**: Logger, validation, and standard string manipulation tools.
- **Constants**: Theme constants, default configurations, and shared values.

#### Current Status (January 24, 2026)

- ✅ API Client with full authentication support
- ✅ Comprehensive type definitions for all core entities
- ✅ Utility functions for common operations
- ✅ Theme constants for mobile application
- ✅ Validation schemas using Zod

#### Integration Points

- **Web & Mobile**: Both applications use the shared `ApiClient` for all backend communication.
- **CLI**: Utilizes shared types and utilities for consistent behavior.
- **Templates**: Leverages shared types and schemas for template generation.

#### Next Steps

- [ ] Add more utility functions for common operations
- [ ] Implement internationalization support
- [ ] Add performance monitoring utilities

### 3. @magicappdev/api

#### Description

Backend API built with Hono and deployed on Cloudflare Workers, providing the core backend functionality for all applications.

#### Architecture

- **Framework**: Hono for lightweight HTTP handling
- **Authentication**: GitHub OAuth2 flow with JWT session management.
- **AI Gateway**: Routing through Cloudflare AI Gateway for optimized model access and usage tracking.
- **Security**: Robust CORS configuration and `authMiddleware` for protecting sensitive endpoints.
- **Streaming**: Real-time streaming capabilities for AI responses.

#### Current Status (January 24, 2026)

- ✅ Full authentication system with GitHub OAuth
- ✅ JWT session management with refresh tokens
- ✅ AI integration with streaming responses
- ✅ Project management endpoints
- ✅ Admin API endpoints with key-based authentication
- ✅ CORS configuration for web application
- ✅ Cloudflare AI Gateway integration

#### Usage Guidelines

- All endpoints should use the `authMiddleware` for authentication where required.
- Use streaming endpoints for AI responses to provide real-time feedback.
- Implement proper error handling and logging for all endpoints.

#### Next Steps

- [ ] Add rate limiting to prevent abuse
- [ ] Implement request/response logging for debugging
- [ ] Add more comprehensive API documentation

### 4. @magicappdev/cli

#### Description

Command-line interface tool for creating and managing MagicAppDev apps, providing a no-code experience for developers to quickly scaffold applications.

#### Architecture

- **Framework**: Commander.js for CLI argument parsing
- **Generators**: Template-based code generation
- **UI**: Chalk for colored terminal output
- **Completions**: Shell completion support for bash, zsh, fish, and pwsh

#### Current Status (January 24, 2026)

- ✅ Basic CLI structure with command parsing
- ✅ Shell completions for major shells
- ✅ Project scaffolding capabilities
- ✅ Package management utilities
- ✅ Version 0.0.8 published to npm

#### Usage Guidelines

- Use the CLI to create new projects with `npx create-magicappdev-app`.
- Leverage shell completions for improved developer experience.
- Use the doctor command for diagnostics and troubleshooting.

#### Next Steps

- [ ] Implement actual code generation in CLI
- [ ] Add MCP integration for developer tools
- [ ] Implement more comprehensive project templates
- [ ] Add interactive project configuration wizard

### 5. @magicappdev/templates

#### Description

App templates and generators for MagicAppDev, providing the foundation for no-code application generation.

#### Architecture

- **Templates**: Handlebars-based templates for apps, components, and screens
- **Generators**: Template generation utilities
- **Registry**: Template registry for management

#### Dependencies

- `@magicappdev/shared`: Shared utilities and types
- `handlebars`: Templating engine
- `tslib`: Runtime library for TypeScript

#### Dev Dependencies

- `@types/node`: Type definitions for Node.js
- `typescript`: TypeScript compiler
- `vitest`: Testing framework

#### Current Status (January 24, 2026)

- ✅ Basic template structure in place
- ✅ Handlebars-based templating system
- ✅ Template registry for management
- ✅ Integration with shared utilities

#### Usage Guidelines

- Use Handlebars for template generation.
- Leverage shared utilities for consistent functionality.
- Ensure proper template management using the registry.

#### Integration Points

- **Shared Package**: Utilizes shared utilities and types from `@magicappdev/shared`.
- **CLI**: Designed to work with CLI for template generation.
- **Apps**: Designed to generate templates for apps, components, and screens.

#### Next Steps

- [ ] Expand template library with more app types
- [ ] Implement component-level templates
- [ ] Add template validation and testing

### 6. @magicappdev/agent

#### Description

AI agent integration using Cloudflare Agents SDK, providing intelligent code generation and project management capabilities.

#### Architecture

- **AI Framework**: Cloudflare Agents SDK
- **Deployment**: Cloudflare Workers
- **Integration**: Connected to AI Gateway for model routing
- **State Management**: Durable Objects for persistent state

#### Current Status (January 24, 2026)

- ✅ Basic agent structure implemented
- ✅ Cloudflare Workers deployment setup
- ✅ Integration with AI Gateway
- ✅ Minimal and test deployment configurations

#### Usage Guidelines

- Use the agent for intelligent code generation and project management.
- Leverage Durable Objects for maintaining state across sessions.
- Implement proper error handling for AI operations.

#### Next Steps

- [ ] Implement advanced agent capabilities
- [ ] Add more sophisticated tool use
- [ ] Implement agent-led project setup wizard

### 7. @magicappdev/llmchat

#### Description

LLM chat integration for AI-powered conversations within applications, providing the foundation for the no-code chat interface.

#### Architecture

- **Chat Framework**: WebSocket-based real-time communication
- **AI Integration**: Connected to AI Gateway for model access
- **Deployment**: Cloudflare Workers
- **Streaming**: Real-time streaming of AI responses

#### Current Status (January 24, 2026)

- ✅ Basic chat functionality implemented
- ✅ WebSocket-based real-time communication
- ✅ AI Gateway integration
- ✅ Streaming responses for real-time feedback

#### Usage Guidelines

- Use the chat component for AI-powered conversations.
- Implement proper error handling for AI operations.
- Leverage streaming for improved user experience.

#### Next Steps

- [ ] Add more sophisticated conversation management
- [ ] Implement conversation history
- [ ] Add context-aware responses

## Applications

### 1. @magicappdev/web

#### Description

Next.js-based web application providing the primary user interface for MagicAppDev, including AI chat, project management, and no-code app generation capabilities.

#### Architecture

- **Framework**: Next.js with TypeScript
- **UI**: Tailwind CSS for styling
- **State Management**: React Context API
- **Authentication**: GitHub OAuth with JWT
- **Real-time**: WebSocket connection for AI chat
- **Deployment**: Cloudflare Workers

#### Current Status (January 24, 2026)

- ✅ Modern Next.js application structure
- ✅ Real-time AI Chat with streaming
- ✅ Project management interface
- ✅ GitHub OAuth integration
- ✅ Responsive design with Tailwind CSS
- ✅ Admin interface for management

#### Usage Guidelines

- Use the web application as the primary interface for MagicAppDev.
- Leverage the AI chat for code generation and project assistance.
- Use the project management interface for organizing and tracking projects.

#### Next Steps

- [ ] Add real-time UI preview for generated code
- [ ] Implement more sophisticated project management features
- [ ] Add collaborative features for teams

### 2. @magicappdev/mobile

#### Description

Ionicframeworkd based mobile application mirroring the web interface features, providing on-the-go access to MagicAppDev capabilities.

#### Architecture

- **Framework**: Ionicframework with Capacitor
- **UI**: Native components with custom styling
- **State Management**: React Context API
- **Authentication**: GitHub OAuth with JWT
- **Real-time**: WebSocket connection for AI chat
- **Themes**: Light/Dark/Auto theme support

#### Current Status (January 24, 2026)

- ✅ Ionicframework with Capacitor
- ✅ GitHub Auth integration
- ✅ Projects listing interface
- ✅ Streaming AI Chat
- ✅ Dark Mode support with Light/Dark/Auto themes
- ✅ App icons converted to PNG with transparency
- ✅ Metro bundler fixes for Windows/pnpm
- ✅ Android build path length issues resolved

#### Usage Guidelines

- Use the mobile application for on-the-go access to MagicAppDev.
- Leverage the AI chat for code generation and project assistance.
- Use the theme system for a consistent user experience.

#### Next Steps

- [ ] Implement more sophisticated mobile-specific features
- [ ] Add offline capabilities
- [ ] Implement push notifications

## CI/CD & Infrastructure

### GitHub Actions

- **ci.yml**: Automated testing, linting, and typechecking.
- **deploy.yml**: Automatic deployment to Cloudflare on merge to `main`.

### Build System

- **Turborepo**: Primary build system with caching
- **Nx**: Secondary build system for project management
- **pnpm**: Package management with workspaces

### Current Issues & Optimizations Needed

#### Build System Consolidation 🔧

- Hybrid Nx + Turborepo setup causing inconsistencies
- Missing `project.json` files for apps/mobile and apps/web
- Mobile project not properly registered in Nx workspace
- Inconsistent build targets across projects

#### Mobile CI/CD Gaps 📱

- No automated mobile app builds in GitHub Actions
- Missing iOS/Android deployment workflows
- No mobile app testing configuration
- No app store deployment automation

#### E2E Testing Gaps 🧪

- No end-to-end testing setup with Playwright or similar
- Missing mobile app E2E tests
- No web app E2E test coverage

#### Integration Gaps 🔗

- MCP integration not implemented across components
- Code generation not fully implemented in CLI
- Docker-Compose quickstart not available

#### Community Features 🚀

- Discord bot not implemented for community server

## 🚀 Development Roadmap

### Phase 1: Foundation (Completed)

- Monorepo setup, Database schema, Core API, Basic Auth.

### Phase 2: AI Orchestration (In Progress)

- [x] Stateful MagicAgent with Cloudflare Agents SDK.
- [x] Model Routing (Fast/Complex/Code).
- [x] Real-time streaming AI responses.
- [ ] Tool Use: Agent-driven template selection.
- [ ] Agent-led project setup wizard.

### Phase 3: No-Code Capabilities (In Progress)

- [x] Basic project scaffolding.
- [x] AI-powered code generation.
- [ ] Real-time UI preview.
- [ ] Visual app builder interface.
- [ ] Template marketplace.

### Phase 4: Automation & Scaling (Upcoming)

- [x] AI Issue Reviewer Agent.
- [ ] Automated code fixers.
- [ ] Advanced CI/CD pipelines.
- [ ] Mobile app store deployment automation.
- [ ] E2E testing with Playwright.

## 📝 TODO List

### High Priority

- [ ] Implement `MagicAgent` tool calling for `registry.generate()`.
- [ ] Add real-time UI preview in Web app using the generated code.
- [ ] Consolidate Nx and Turborepo build systems.
- [ ] Implement E2E testing suite with Playwright.
- [ ] Add automated mobile app deployment to CI/CD.

### Medium Priority

- [ ] Enhance mobile deep-link reliability on Android.
- [ ] Implement global search across templates and projects.
- [ ] Add Docker-Compose Quickstart for App Developing.
- [ ] Implement Discord bot with interactions endpoint and linked roles verification.

### Low Priority

- [ ] Add more comprehensive API documentation.
- [ ] Implement database seeding utilities.
- [ ] Add internationalization support.
- [ ] Implement offline capabilities for mobile app.

## Conclusion

This document provides a comprehensive overview of the architecture, dependencies, usage guidelines, and integration points for each package and application in the MagicAppDev project. Use this documentation to ensure consistency and maintainability across all components. The project is focused on building a full-stack framework for people to build mobile and web apps with no-code capabilities, including a website with agent chat, mobile app, and CLI to generate code setups, all hosted on Cloudflare Workers and designed to be free-tier compatible.

## Development Guidelines for Agentic Coding

### 1. Build, Lint, and Quality Commands

Always run these commands from the root directory using `pnpm`.

- **Build Everything**: `pnpm build` (uses Turborepo)
- **Build Package**: `turbo build --filter=@magicappdev/<package-name>`
- **Format Code**: `pnpm format` (runs Prettier)
- **Fix Linting**: `pnpm lint:fix` (runs ESLint)
- **Typecheck All**: `pnpm typecheck` (runs Nx/TypeScript)
- **Full Check**: `pnpm check` (format + lint:fix + typecheck + build)

### 2. Code Style Guidelines

#### Formatting (Prettier)

Adhere to the `.prettierrc` configuration:

- **Width**: 80 characters
- **Indentation**: 2 spaces (no tabs)
- **Semicolons**: Required
- **Quotes**: Double quotes
- **Trailing Commas**: All (arrays, objects, imports)
- **Arrow Functions**: Avoid parentheses for single parameters (`x => x`)

#### Imports & Modules

- **Extensions**: Always use explicit `.js` extensions for local imports in TypeScript files (e.g., `import { foo } from "./foo.js"`).
- **Organization**: Imports are automatically sorted (External > Workspace > Relative).
- **Workspace Packages**: Use `@magicappdev/` prefix for internal dependencies.

#### TypeScript & Types

- **Strict Mode**: `strict: true` is enabled. Avoid `any` unless absolutely necessary.
- **Inference**: Use Type Inference for local variables.
- **Schema Types**: Export inferred types from Drizzle schemas:
  ```typescript
  export type User = typeof users.$inferSelect;
  export type NewUser = typeof users.$inferInsert;
  ```
- **Interfaces**: Prefer `interface` for public APIs and props.
- **Utility Types**: Use `Record<string, unknown>` for generic objects instead of `object` or `any`.

#### Naming Conventions

- **Variables/Functions**: `camelCase`
- **Components/Types/Interfaces**: `PascalCase`
- **Files**: `kebab-case.ts` or `kebab-case.tsx`
- **Constants**: `UPPER_SNAKE_CASE`
- **Database Tables/Columns**: `snake_case`

#### Error Handling

- **API Requests**: Always check `response.ok` before parsing JSON.
- **Error Extraction**: Parse error messages from the backend response:
  ```typescript
  const errorData = await response.json().catch(() => ({}));
  const message = errorData.error?.message || "Request failed";
  throw new Error(message);
  ```
- **API Format**: Backend errors follow `{ success: false, error: { code, message } }`.
- **Validation**: Use Zod schemas for request body validation.

#### React Patterns (Web & Mobile)

- **Components**: Use functional components with hooks.
- **Ref Forwarding**: Use `forwardRef` for UI components:
  ```typescript
  export const Button = forwardRef<HTMLButtonElement, ButtonProps>(...)
  ```
- **Styling**:
  - **Web**: Tailwind CSS utility classes with `cn()` helper. Avoid inline styles.
  - **Mobile**: `StyleSheet.create()` for native components. Use theme constants from `@magicappdev/shared`.
- **Context**: Use React Context for global state (Auth, Theme).

#### Database & API (Cloudflare)

- **Drizzle**: Define schemas in `packages/database/src/schema/`. Export the combined schema object in `index.ts`.
- **Hono API**: Use the middleware pattern for auth: `app.use("/path/*", authMiddleware)`.
- **Auth**: Access the authenticated user via `c.get("userId")` in Hono.
- **Environment**: Access variables through `c.env` (e.g., `c.env.DB`, `c.env.JWT_SECRET`).

#### Comments

- **Minimalism**: Write self-documenting code. Use comments only for "why", not "what".
- **JSDoc**: Use sparingly for complex exported utilities.

### 3. Testing and Quality Assurance

- **Framework**: Vitest is used for unit and integration tests.
- **Execution**: Run `pnpm test` to execute all tests in the workspace.
- **Continuous Integration**: GitHub Actions runs linting, typechecking, and tests on every push.
- **Pre-commit Workflow**: Husky and lint-staged ensure code quality before commits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magicappdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/magicappdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
