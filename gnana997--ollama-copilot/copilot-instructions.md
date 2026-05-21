## ollama-copilot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Visual Studio Code extension that integrates Ollama language models for code completion and chat functionality. The extension provides inline code completions and a chat interface within VS Code.

## Development Commands

### Build & Development
- `npm run dev` - Development build with watch mode
- `npm run build` - Production webpack build
- `npm run compile` - Development webpack build

### Testing & Quality
- `npm test` - Run VS Code tests (compiles and lints first)
- `npm run lint` - Run ESLint

### Publishing
- `npm run vscode:prepublish` - Production build for publishing

## Architecture Overview

### Entry Points
- **Main**: `src/extension.ts` - Activates extension, registers commands and providers
- **Output**: `out/extension.js` (webpack bundled)

### Core Modules

**Inline Completion System** (`src/inlineCompletionProvider/`)
- `OllamaInlineCompletionProvider.ts` - Main provider with LRU caching and 200ms debouncing
- `promptGenerators.ts` - Creates context-aware prompts from file content
- Context detection: Analyzes file type, frameworks, and code patterns
- Response cleaning: Processes AI outputs for proper formatting

**Chat Interface** (`src/chatInterface/`)
- `SidebarChatViewProvider.ts` - Sidebar webview integration
- `ChatPanel.ts` - Standalone chat panel implementation
- `ChatMessageHandler.ts` - Message processing and Ollama API interaction
- Uses webview message passing for UI communication

### Key Design Patterns
- **Provider Pattern**: Both inline completion and chat implement VS Code provider interfaces
- **Singleton Management**: Global instances for providers managed in extension.ts
- **Caching Strategy**: LRU cache with TTL for completion results
- **Debouncing**: 200ms delay prevents excessive API calls
- **Message Passing**: Webviews communicate via postMessage/onDidReceiveMessage

### Configuration
The extension connects to Ollama at `http://localhost:11434` by default. Users can:
- Select different Ollama models via command palette
- Update the API host through settings
- Clear the completion cache when needed

## Important Files
- `package.json` - Extension manifest, commands, and npm scripts
- `webpack.config.js` - Bundling configuration with source maps
- `tsconfig.json` - TypeScript strict mode, ES2022 target
- `.vscode/launch.json` - Debug configurations for development

## Development Notes
- Uses TypeScript with strict mode enabled
- Webpack bundles with TerserPlugin for optimization
- ESLint configured for code quality
- Targets VS Code API 1.74.0 minimum
- Webviews require proper CSP and nonce handling for security

## Dependency Injection Architecture

The extension uses a custom dependency injection (DI) system for better testability and maintainability.

### Key Components
- **ServiceContainer** (`src/di/ServiceContainer.ts`) - Main DI container with lifecycle management
- **Service Interfaces** (`src/services/interfaces/`) - All services have TypeScript interfaces
- **Service Implementations** (`src/services/implementations/`) - Concrete implementations
- **ServiceRegistration** (`src/services/ServiceRegistration.ts`) - Central service registration

### Service Lifecycle
- **Singleton**: Created once, shared across the application (most services)
- **Transient**: New instance for each resolution
- **Scoped**: Shared within a scope (future enhancement)

### Adding a New Service
1. Create interface in `src/services/interfaces/IMyService.ts`
2. Add to `SERVICE_IDENTIFIERS` in `src/di/index.ts`
3. Create implementation in `src/services/implementations/MyService.ts`
4. Register in `src/services/ServiceRegistration.ts`
5. Resolve in components: `container.resolve<IMyService>(SERVICE_IDENTIFIERS.IMyService)`

### Testing with DI
```typescript
// Create test container with mocks
const container = new ServiceContainer();
container.registerSingleton(
  SERVICE_IDENTIFIERS.IOllamaApiService,
  () => new MockOllamaApiService()
);

// Test your component
const service = container.resolve<IChatService>(SERVICE_IDENTIFIERS.IChatService);
```

### Best Practices
- Always use interfaces for dependencies
- Implement `onDispose()` for cleanup
- Avoid circular dependencies
- Use `tryResolve()` for optional dependencies
- Keep services focused on single responsibility

## Development Memories
- Nice improvements. I will test manually and then need to work adding unit test cases
- Still need to fix the build errors: There are still async/await issues in BaseChatComponent and WebViewMessage type issues in DIBaseChatComponent. The codebase has many remaining issues that need refactoring to properly work with the new dependency injection architecture.

  Summary of Refactoring Progress:

  1. ✅ Created IWebViewService interface and WebViewService implementation for managing webview communication
  2. ✅ Updated BaseChatComponent to use dependency injection by accepting services as constructor parameters
  3. ✅ Updated ChatPanel and SidebarChatViewProvider to receive and pass injected services
  4. ✅ Updated ViewProviderFactory to resolve services from the DI container
  5. ✅ Registered WebViewService in ServiceRegistration with proper dependencies
  6. ✅ Fixed ValidationService implementation to remove dependency on deleted InputValidationService
  7. ✅ Fixed many import errors by removing references to deleted services

  Remaining Build Errors:
  - WebViewMessage type issues due to TypeScript strict checking with index signatures
  - Async/await syntax errors in some methods
  - MockServices.ts has parameter errors
  - Some files still reference old patterns

  The chat architecture has been significantly improved to use dependency injection, but the codebase needs further cleanup to fully compile without errors.

---
> Source: [gnana997/ollama-copilot](https://github.com/gnana997/ollama-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
