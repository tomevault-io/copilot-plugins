## joomla-mcp-php

> This file provides guidance to AI code agents when working with code in this repository.

# CLAUDE.md

This file provides guidance to AI code agents when working with code in this repository.

## Project Overview

MCP4Joomla is an MCP (Model Context Protocol) server for Joomla 5+, written in PHP. It allows LLMs to interact with Joomla installations via the Joomla Web Services API. The server communicates over stdio transport using the `php-mcp/server` library.

## Development Commands

```bash
# Install dependencies
composer install

# Run the MCP server (stdio transport)
php mcp4joomla.php server

# Run with debug logging
php mcp4joomla.php server --debug

# Custom log file location
php mcp4joomla.php server --log=/path/to/file.log
```

There are no tests, linters, or CI pipelines configured.

## Required Environment Variables

- `JOOMLA_BASE_URL` — Base URL of the Joomla site (no trailing `/api` or `/index.php`)
- `BEARER_TOKEN` — Base64-encoded Joomla Super User API token

## Architecture

### Entry Point

`mcp4joomla.php` — Bootstraps Composer autoloading, creates the DI container, configures the MCP server, auto-discovers tools in `src/Server/`, and listens on stdio.

### Dependency Injection

Uses **Pimple** with service providers registered in `src/Container/Container.php`:

- `EnvironmentProvider` — Validates and exposes `JOOMLA_BASE_URL` and `BEARER_TOKEN`
- `OptionsParserProvider` — CLI argument parsing (`server`, `--debug`, `--log`)
- `InputProvider` — Parsed CLI input
- `LogProvider` — Monolog with rotating file handler (`log/debug.log`)
- `HttpProvider` — `HttpDecorator` wrapping Joomla HTTP client with API auth headers

Services are accessed via `Factory::getContainer()->get('serviceName')`. The container is a singleton managed by `src/Container/Factory.php`.

### MCP Tool Classes

Located in `src/Server/`. The `php-mcp/server` library auto-discovers tools by scanning this directory for `#[McpTool]` PHP attributes. Each public method annotated with `#[McpTool]` becomes an MCP tool.

Tool annotations use `ToolAnnotations` to declare hints: `readOnlyHint`, `destructiveHint`, `idempotentHint`.

Currently implemented: `src/Server/Content/Articles.php` (CRUD operations for Joomla articles).

### Adding New MCP Tools

1. Create a class under `src/Server/` (mirroring Joomla component structure, e.g., `Content/Articles.php`)
2. Add `#[McpTool]` attributes to public methods
3. Use the utility traits for common functionality
4. Access the DI container via `Factory::getContainer()` to get `http` and `log` services
5. Tools are auto-discovered — no registration needed

### Utility Traits (`src/Utility/`)

Tool classes compose behavior via traits:

- `AutoLoggingTrait` — Call `$this->autologMCPTool()` at the start of each tool method; uses `debug_backtrace` + reflection to log the call with all arguments
- `ArticleTextTrait` — `toHtml()` converts Markdown to HTML via CommonMark
- `TitleToAliasTrait` — `titleToAlias()` generates URL slugs from titles
- `HandleJoomlaAPIErrorTrait` — Parses Joomla API error responses and throws exceptions
- `GetDataFromResponseTrait` — Extracts and validates JSON API response data
- `HttpDecorator` — Wraps `Joomla\Http\Http` with automatic auth headers (`Authorization: Bearer` + `X-Joomla-Token`) and request/response logging; provides `getUri()` for building API endpoint URIs

### Joomla API Integration Pattern

All Joomla API calls follow this pattern:
```php
$http = Factory::getContainer()->get('http');
$uri = $http->getUri('v1/content/articles');  // builds full API URL
$response = $http->get($uri);
$this->handlePossibleJoomlaAPIError($response);
return $this->getDataFromResponse($response, 'articles');
```

The API uses JSON:API format (`application/vnd.api+json`).

## Code Style

- PHP 8.1+ with `declare(strict_types=1)`
- PSR-4 autoloading under namespace `Dionysopoulos\Mcp4Joomla\`
- Allman brace style (braces on their own line)
- Tab indentation
- Commented-out `#[Schema]` attributes on tool method parameters (removed at runtime due to compatibility; kept as documentation)

---
> Source: [nikosdion/joomla-mcp-php](https://github.com/nikosdion/joomla-mcp-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
