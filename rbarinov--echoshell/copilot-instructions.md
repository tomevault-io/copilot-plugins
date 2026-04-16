## echoshell

> Distributed system for remote terminal management and execution via voice commands from mobile devices (iPhone/Apple Watch), with real-time output streaming and text-to-speech responses.

# EchoShell - Voice-Controlled Terminal Management System

## Project Overview
Distributed system for remote terminal management and execution via voice commands from mobile devices (iPhone/Apple Watch), with real-time output streaming and text-to-speech responses.

## Mandatory Rules

### English Only
**CRITICAL**: All code, comments, documentation, commit messages, and communication MUST be in English.

### Conventional Commits
All commits MUST follow the format: `<type>(<scope>): <subject>`
Types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert

## Technology Stack
- **Server**: TypeScript 5.0+, Node.js 20+, Express/Fastify
- **Client**: Swift 5.9+, SwiftUI (iOS 17+, watchOS 10+)
- **AI**: OpenAI (GPT-4, Whisper, TTS), Anthropic Claude
- **Tools**: Cloudflare Tunnel, LangChain.js, node-pty

## Available Commands

### /review
Review a pull request with detailed feedback including:
1. Code quality assessment
2. Security implications
3. Performance considerations
4. Suggested improvements

### /security-review
Complete security review analyzing:
- Authentication/authorization issues
- Input validation and sanitization
- API key handling and storage
- Potential vulnerabilities (OWASP Top 10)
- Command injection risks

### /init
Initialize or update CLAUDE.md with codebase documentation including:
- Project structure overview
- Key files and their purposes
- Important patterns and conventions
- Setup instructions

## Code Standards
- TypeScript strict mode enabled
- Explicit error handling required
- Security-first approach for all features
- Comprehensive logging for debugging
- Real-time output streaming for terminal sessions

## iOS/WatchOS Development Rules

### Agent Expertise
**CRITICAL**: The agent is an expert in iOS and WatchOS mobile application development using Swift programming language, following best practices and Apple's recommendations.

### Code Quality Requirements
1. **Compilation Verification**: Before submitting code for review, the agent MUST verify compilation with zero errors and zero warnings. This is mandatory for all builds.
2. **Architectural Patterns**: The agent MUST follow maximum correct architectural patterns in accordance with community best practices and Apple's official recommendations.
3. **Best Practices**: All code MUST adhere to:
   - Swift best practices and style guidelines
   - Apple's Human Interface Guidelines
   - SwiftUI best practices
   - Combine framework patterns (when applicable)
   - MVVM architecture pattern (for this project)

### State Management & Lifecycle
**CRITICAL**: The application MUST work correctly regardless of:
- Navigation transitions (screen changes, tab switching, back navigation)
- Screen state changes (app becoming active/inactive/background)
- Device locking/unlocking
- Phone call interruptions
- Control Center or Notification Center interactions
- Any other system-level interruptions

**Requirements:**
- All state MUST be properly preserved and restored
- No logic failures or crashes during state transitions
- Proper handling of app lifecycle events (applicationDidBecomeActive, applicationWillResignActive, etc.)
- State persistence using appropriate mechanisms (UserDefaults, CoreData, etc.)
- Proper cleanup in deinit methods

### Screen Sleep Prevention
**CRITICAL**: During active application usage, the screen MUST NOT sleep or lock:
- On iPhone: Screen must stay active during recording, playback, or active terminal sessions
- On Apple Watch: Screen must stay active during interactive operations
- Use `IdleTimerManager` or similar mechanisms to prevent screen sleep
- Properly release screen sleep prevention when operations complete
- Handle edge cases (app backgrounding, interruptions, etc.)

### Testing Requirements
- All code changes MUST compile without errors or warnings
- Unit tests MUST pass (when applicable)
- Integration tests MUST pass (when applicable)
- Manual testing scenarios MUST be considered for state transitions and lifecycle events

## TypeScript Architecture Rules

**CRITICAL**: All TypeScript code (laptop-app and tunnel-server) MUST follow the architecture patterns and best practices defined in `TYPESCRIPT_ARCHITECTURE_RULES.md`.

### Key Principles:
- **Separation of Concerns**: Single responsibility per module
- **Type Safety**: No `any` types, runtime validation with Zod
- **Error Handling**: Explicit, structured error handling
- **Structured Logging**: JSON logs with appropriate levels
- **Testing**: Comprehensive test coverage
- **Security**: Input validation, secure secrets management
- **CI/CD**: GitHub Actions workflows with npm audit and tests (MANDATORY for publishable packages)

### Project-Specific Rules:
- **laptop-app**: See `laptop-app/.cursorrules` for laptop-app specific rules
- **tunnel-server**: See `tunnel-server/.cursorrules` for tunnel-server specific rules

Both projects inherit these core rules and add project-specific guidelines.

## Security Considerations
- Master API keys stored only on laptop
- Ephemeral keys for mobile devices (1-hour expiration)
- Rate limiting per device
- TLS/WSS for all communication
- Dangerous command blocking (rm -rf, mkfs, dd)

## Project Context
See CLAUDE.md for full technical specification and implementation details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rbarinov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
