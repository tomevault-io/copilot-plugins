## code-review-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a code review agent powered by Google's Gemini AI model. The agent analyzes git diffs and provides professional code review feedback using AI SDK with tools integration.

## Architecture

- **index.ts**: Main entry point that sets up the AI agent with streaming text generation
- **tools.ts**: Implements `getFileChangesInDirectoryTool` using simple-git to extract file diffs
- **prompts.ts**: Contains the expert code reviewer system prompt that defines the AI's personality and review focus

The agent uses:
- **AI SDK** (`ai` package) for LLM integration and tool calling
- **Google Gemini 2.5 Flash** model via `@ai-sdk/google`
- **simple-git** for git operations and diff extraction
- **Zod** for input validation schemas

## Commands

### Development
- `bun index.ts` - Run the code review agent
- `bun --hot index.ts` - Run with hot reload during development
- `bun test` - Run tests (using Bun's built-in test runner)

### Package Management
- `bun install` - Install dependencies
- `bun add <package>` - Add new dependencies

## Configuration

The agent requires a Google Generative AI API key set in `.env`:
```
GOOGLE_GENERATIVE_AI_API_KEY=your_api_key_here
```

## Key Features

- **Real-time streaming**: Streams AI responses in real-time to stdout
- **Git diff analysis**: Uses tool calling to dynamically fetch git diffs from any directory
- **Commit message generation**: Analyzes staged changes and generates appropriate commit messages
- **Review export**: Saves detailed code reviews to markdown files with timestamps
- **File exclusion**: Excludes specific files (dist, bun.lock) from review
- **Safety limits**: Limits to 10 AI reasoning steps to prevent infinite loops
- **Professional review focus**: Emphasizes correctness, clarity, maintainability, consistency, performance, security, testing, and scalability

## Available Tools

1. **getFileChangesInDirectoryTool**: Analyzes git diffs in a specified directory
2. **generateCommitMessageTool**: Examines staged changes and provides commit message suggestions
3. **writeReviewToFileTool**: Exports review content to markdown files with timestamps

## Usage Examples

### Basic Code Review
```bash
bun index.ts
# Default: Reviews changes in current directory
```

### Generate Commit Message
Modify the prompt in `index.ts` to:
```typescript
"Generate a commit message for the staged changes in './' directory"
```

### Save Review to File
Modify the prompt in `index.ts` to:
```typescript
"Review the code changes in './' directory and save the review to 'review-report.md'"
```

## Bun Runtime

Default to using Bun instead of Node.js:

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun install` instead of `npm install`
- Bun automatically loads .env files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/webmaster254) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
