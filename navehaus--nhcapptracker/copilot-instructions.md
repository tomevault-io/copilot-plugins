## nhcapptracker

> **Table of Contents**

# Context, Rules, and Guidelines for AI Agents

**Table of Contents**
- [Overview](#overview)
- [Important References](#important-references)
- [Tech stack](#tech-stack)
- [Safety & Operational Rules (MANDATORY)](#safety--operational-rules-mandatory)
- [Terminology](#terminology)
- [Process](#process)
  - [Requirements (MANDATORY)](#requirements-mandatory)
- [Major Features](#major-features)
- [Testing](#testing)
  - [Requirements (MANDATORY)](#requirements-mandatory-2)
- [Example OpenSpec Workflows](#example-openspec-workflows)
  - [One-Shot Implementation](#one-shot-implementation)
  - [One-Shot Exploration to Implementation](#one-shot-exploration-to-implementation)
  - [Iterative Exploration to Implementation](#iterative-exploration-to-implementation)

## Overview

## Tech stack
- C# 10
- GitHub Flavored Markdown

## Safety & Operational Rules (MANDATORY)
To maintain repository integrity, agents MUST follow these rules:
- **No Secrets**: NEVER commit secrets, API keys, or credentials
- **Git Safety**:
  - DO NOT modify git configuration
  - DO NOT use `--no-verify` or bypass hooks unless explicitly requested
  - DO NOT force push to protected branches, e.g. `master` or `main`
  - DO NOT automatically fix `git` errors---ALWAYS ask the user for confirmation first
  - DO warn the user if attempting to commit to `master` or `main`
  - DO warn the user if `git` returns an error or reports that the remote branch is missing
  - DO offer to resolve `git` errors, presenting the user with 1-4 options for doing so
- **Path Handling**: Always use absolute paths when interacting with file system tools
- **Build Consistency**: DO NOT directly modify or remove any files or directories listed in `.gitignore`
- **Verification**: Always run build and test commands after modifications

## Codebase Knowledge Graph (codebase-memory-mcp)
`codebase-memory-mcp` is used to maintain a codebase knowledge graph

### Rules (MANDATORY)
- You MUST prefer codebase-memory-mcp tools over ripgrep/grep/glob/file-search for code discovery
- You MUST offer to call `index_repository` first if the graph is not indexed yet

### High-to-Low Tool Priority Order
1. `search_graph` — find functions, classes, routes, variables by pattern
2. `trace_path` — trace who calls a function or what it calls
3. `get_code_snippet` — read specific function/class source code (NOT Read/cat)
4. `query_graph` — run Cypher queries for complex patterns
5. `get_architecture` — high-level project summary
6. `search_code` — for text search (graph-augmented grep)

### When to fall back to ripgrep/grep/glob/file-search
- Searching for string literals, error messages, config values
- Searching non-code files (Dockerfiles, shell scripts, configs)
- When MCP tools return insufficient results

## Process

### Prerequisites (MANDATORY)
Evaluate the following **IF** statements, then **ask the user** how to proceed for each true **IF** statement:
- **IF** the `openspec` directory is missing or inaccessible in the current workspace
- **IF** the `tdd`, `nhc-openspec-commit`, or `nhc-conventional-commit` skill is unavailable

### Requirements (MANDATORY)
- You MUST use an OpenSpec spec-driven workflow to plan significant features
- You MUST use the `nhc-openspec-commit` skill to generate a `git` commit message and commit changes made within an OpenSpec workflow
- You MUST use the `nhc-conventional-commit` skill to generate a `git` commit message for changes made outside an OpenSpec workflow
- You MUST use the test-driven development (TDD) `tdd` skill:
  - To generate testing tasks in the OpenSpec `tasks.md` artifact
  - To test new implementation
  - To test changes to existing code
- You MAY skip `tdd` testing for non-implementation changes unless the user requests it

### Major Features
- Major features are comprised of two or more distinct OpenSpec changes, e.g. an MVP for a new front-end user interface, the implementation of a new back-end service, or resolving multiple related code issues
- You MUST use `openspec-explore` to research and develop a plan for implementing the changes comprising a major feature
- You MUST use the `docs/plans/<major-feature-name>` directory as a workspace during `openspec-explore`:
  - DO NOT store OpenSpec artifacts under `docs/plans/<major-feature-name>`
  - You MUST ignore ALL files in `docs/plans/archive` unless the user explicitly requests to review them
  - Before creating planning documents, you MUST **ask the user** to provide the name of the major feature if the user has not already provided one
  - You MUST review all relevant planning documents for context while developing `<major-feature-name>`
  - You MUST offer to record decisions, constraints, milestones, dependencies, or any other important information relevant to planning the major feature
  - You MUST offer to create or update a current list of changes as they become clear
  - You MUST include a dependency diagram when recording the current list of changes, if the dependencies are clear; prefer a mermaid diagram for ease-of-use
  - You MAY store high-level planning files directly under `docs/plans`, e.g. project-level guidance or decisions rather than feature-level information
  - You MAY offer to create the OpenSpec artifacts for a concrete, implementable change, but they MUST be stored in the correct directory under `openspec`
  - Prefer `openspec-ff-change` over `openspec-new`/`openspec-continue` when creating change artifacts for a major feature

**Most common OpenSpec workflow:**
1. `openspec-new <name>`: Initialize the change
2. `openspec-ff-change <name>`: Generate/update artifacts
  - **CRITICAL** OpenSpec implementation testing tasks in `tasks.md` MUST use test-driven development (TDD) using the `tdd` skill
3. `nhc-openspec-refine <name>`: Make artifacts implementation-ready
4. `openspec-apply-change <name>`: Implement the tasks (following TDD process - see [Testing](#testing))
5. `dotnet tests`: Verify all tests before completing a code change
6. `nhc-opsx-verify <name>`: Validate implementation against specs (see [Error Handling Guidance](#error-handling-guidance) if this fails)
7. `openspec-sync-specs <name>`: Make OpenSpec specs artifacts permanent
8. `openspec-archive-change <name>`: Archive the OpenSpec change
9. `nhc-openspec-commit`: Generate a standard `git` commit message for the changes made by the OpenSpec workflow

## Testing

### Requirements (MANDATORY)
- Testing MAY be skipped for non-code changes, e.g. documentation-only changes
- Tests MUST be stored under `tests/<category>`, where `<category>` MUST be the name of the library or tool containing the feature/class under test

---
> Source: [NaveHaus/NhcAppTracker](https://github.com/NaveHaus/NhcAppTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
