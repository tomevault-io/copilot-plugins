## commit-ai

> `commit-ai` is a command-line tool that uses AI to automatically generate commit messages for git repositories. It analyzes the changes in staged files and creates a concise and descriptive commit message that follows conventional commit standards.

# commit-ai

## Goal of the Project

`commit-ai` is a command-line tool that uses AI to automatically generate commit messages for git repositories. It analyzes the changes in staged files and creates a concise and descriptive commit message that follows conventional commit standards.

## Tech Stack

- **Language:** TypeScript
- **AI:** Anthropic's Claude Sonnet 4.0 (via `@ai-sdk/anthropic` and `ai`)
- **CLI Framework:** Custom-built using Node.js modules
- **Key Dependencies:**
  - `chalk` for terminal styling
  - `dotenv` for environment variable management
  - `zod` for schema validation
- **Build Tool:** `tsc` (TypeScript Compiler)
- **Package Manager:** pnpm

## Current Scope

The current scope of the `commit-ai` tool includes the following features:

- **AI-powered commit messages:** Analyzes staged file changes to generate high-quality commit messages.
- **Custom instructions:** Allows users to provide custom instructions to the AI for generating the commit message (e.g., to follow conventional commits).
- **Configuration:** The tool can be configured using a `commit-ai.json` file in the root of the project, with the following options:
  - `useReadme`: If `true`, the tool includes the project's `README.md` in the AI prompt for better context.
  - `customInstructions`: A string for providing specific instructions to the AI.
  - `readFullChangedFiles`: If `true`, the tool reads the entire content of changed files, not just the diff, for more accurate but potentially slower and more expensive commit messages.
- **Command-line interface:** The tool is intended to be used as a command-line interface within a git repository.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lukas-karsch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lukas-karsch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
