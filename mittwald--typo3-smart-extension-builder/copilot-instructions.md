## typo3-smart-extension-builder

> This file provides guidance to coding agents like Claude Code or OpenCode when working with code in this repository.

# CLAUDE.md

This file provides guidance to coding agents like Claude Code or OpenCode when working with code in this repository.

## Project Overview

Smart Extension Builder is a TYPO3 MCP (Model Context Protocol) server that provides AI-powered code generation tools for TYPO3 extension development. It exposes scaffolding tools from `friendsoftypo3/kickstarter` to AI assistants via MCP.

## Build & Test Commands

All commands use the Docker-based test runner. Provide `-b docker` as a flag if Podman is not available on your system.

```bash
# Run unit tests
Build/Scripts/runTests.sh -p 8.4 -s unit

# Run a single test file
Build/Scripts/runTests.sh -p 8.4 -s unit -- Tests/Unit/MCP/Tools/ExtensionToolsTest.php

# Run functional tests (requires database)
Build/Scripts/runTests.sh -p 8.4 -d mysql -s functional

# Static analysis
Build/Scripts/runTests.sh -p 8.4 -s phpstan

# Code style check
Build/Scripts/runTests.sh -p 8.4 -s cgl -n

# Fix code style issues
Build/Scripts/runTests.sh -p 8.4 -s cgl

# PHP lint
Build/Scripts/runTests.sh -p 8.4 -s lint

# Install dependencies
Build/Scripts/runTests.sh -p 8.4 -s composerUpdate
```

## Architecture

### MCP Tool Pattern

Tools in `Classes/MCP/Tools/` follow this structure:
1. Class marked with `#[Autoconfigure(public: true)]` for DI discovery
2. Methods annotated with `#[McpTool]` attribute
3. Parameters validated via `#[Schema]` attributes (pattern, format, enum)
4. Throw `ToolCallException` for validation failures
5. Delegate to kickstarter Creator services for code generation
6. Return descriptive success message strings

### MCP Resource Pattern

Resources in `Classes/MCP/Resources/` provide read-only context:
1. Use `#[McpResource]` for static resources
2. Use `#[McpResourceTemplate]` for URI-parameterized resources
3. Return formatted Markdown content
4. Also require `#[Autoconfigure(public: true)]`

### Shared Traits

- `ExtensionInformationTrait` - Reads composer.json/ext_emconf.php for extension metadata
- `ExtensionCreationTrait` - Validates extension keys, class names, base paths

### Key Dependencies

- `mcp/sdk` - MCP protocol implementation for PHP
- `friendsoftypo3/kickstarter` - TYPO3 code generation engine (Creator classes)

## Naming Conventions

- Tool names: `typo3_{action}_{component}` (snake_case)
- Tool classes: `{Component}Tools.php`
- Test classes mirror source: `Tests/Unit/MCP/Tools/{Component}ToolsTest.php`

## Coding Standards

- PHP 8.4+ with strict types
- PSR-12 coding style (enforced via PHP-CS-Fixer)
- PHPStan level 12 (via TYPO3 baseline)
- Prefer readonly classes for tools
- Use Conventional Commits (feat:, fix:, refactor:, etc.)

---
> Source: [mittwald/typo3-smart-extension-builder](https://github.com/mittwald/typo3-smart-extension-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
