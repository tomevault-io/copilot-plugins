## claude-code-mastra

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code × Mastra Agent Integration** library that bridges Claude Code's TypeScript SDK with the Mastra framework. The core architecture consists of:

- **ClaudeCodeAgent**: Main class extending Mastra's Agent base class, implementing `generate()` and `stream()` methods that return ai package-compatible results
- **ToolBridge**: Handles integration between Claude Code SDK and Mastra custom tools, with automatic tool conflict resolution
- **MessageConverter**: Converts between Claude Code SDK message formats and Mastra response formats, with cleanup of internal SDK artifacts
- **SessionManager**: Manages Claude Code session lifecycle, cost tracking, and automatic cleanup

The integration works by:
1. Detecting when Mastra tools are present and automatically disabling Claude Code's built-in tools to prevent conflicts
2. Using a tool execution loop that processes JSON tool calls in Claude responses
3. Converting all responses to match ai package interfaces (GenerateTextResult/StreamTextResult)

## Development Commands

### Testing
```bash
# Unit tests only (uses mocks)
npm run test:unit

# Integration tests (components with mocks)  
npm run test:integration

# E2E tests (real Claude Code SDK - requires auth)
npm run test:e2e

# All tests except E2E
npm run test

# All tests including E2E
npm run test:all

# Watch mode for development
npm run test:watch

# Coverage report
npm run test:coverage

# Interactive test UI
npm run test:ui
```

### Build & Type Checking
```bash
# TypeScript compilation
npm run build

# Type checking without emit
npm run typecheck
```

### E2E Test Requirements
E2E tests require:
- `claude login` or `ANTHROPIC_API_KEY` environment variable
- Set `CLAUDE_CODE_E2E_TEST=true` to enable
- Internet connection (consumes API credits)

## Architecture Details

### Tool Integration Strategy
When Mastra tools are present, the agent:
- Generates system prompts instructing Claude to output JSON tool calls
- Disables Claude Code built-in tools (`['Task', 'Bash', 'Read', 'Write', 'Edit', 'LS', 'Glob', 'Grep']`)
- Implements tool execution loop with max 5 iterations to prevent infinite loops
- Returns `toolCalls` and `toolResults` arrays in ai package format

### Message Flow
1. Input messages → `extractPromptFromMessages()` → Claude Code SDK
2. SDK responses → `MessageConverter.extractContentFromMessage()` → Clean text (removes `[tool: xxx]` artifacts)
3. Tool calls detected via `ToolBridge.detectToolCall()` → JSON parsing
4. Tool execution → `ToolBridge.executeTool()` → Results fed back to Claude
5. Final response → `MastraResponse` with metadata (sessionId, cost, duration)

### Session Management
- Auto-creates sessions per request with unique IDs
- Tracks cost and duration metadata
- 30-second automatic cleanup timer
- Session info available via `getSessionInfo()` and `getAllActiveSessions()`

### Security Considerations
**CRITICAL**: Claude Code SDK tool restrictions (`allowedTools`/`disallowedTools`) are currently not working properly. This creates security risks in production:
- Unrestricted file operations
- Arbitrary command execution via Bash tool
- Resource abuse potential

Use in sandboxed environments and prefer Mastra tools over SDK built-ins.

## Debugging Tools
Debug scripts in `test/debug/` for development:
- `tool-call-debug.ts`: Test JSON tool call detection
- `tool-execution-debug.ts`: Test tool execution flow

Enable debug logging by checking console output in agent methods.

## Git Hooks
Pre-push hook runs `gitleaks` to prevent secret leaks. Override with `git push --no-verify` if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/t3ta)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/t3ta)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
