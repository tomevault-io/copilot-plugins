## claude-code-router-next

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Router is a tool that routes Claude Code requests to different LLM providers. It uses a Monorepo architecture with four main packages:

- **cli** (`@wengine-ai/claude-code-router-cli`): Command-line tool providing the `ccr` command
- **server** (`@wengine-ai/claude-code-router-server`): Core server handling API routing and transformations
- **shared** (`@wengine-ai/claude-code-router-shared`): Shared constants, utilities, and preset management
- **ui** (`@wengine-ai/claude-code-router-ui`): Web management interface (React + Vite)

## Build Commands

### Build all packages
```bash
pnpm build
```

### Build individual packages
```bash
pnpm build:cli      # Build CLI
pnpm build:server   # Build Server
pnpm build:ui       # Build UI
```

### Development mode
```bash
pnpm dev:cli        # Develop CLI (ts-node)
pnpm dev:server     # Develop Server (ts-node)
pnpm dev:ui         # Develop UI (Vite)
```

### Publish
```bash
pnpm release        # Build and publish all packages
```

## Core Architecture

### 1. Routing System (packages/server/src/utils/router.ts)

The routing logic determines which model a request should be sent to:

- **Default routing**: Uses `Router.default` configuration
- **Project-level routing**: Checks `~/.claude/projects/<project-id>/claude-code-router.json`
- **Custom routing**: Loads custom JavaScript router function via `CUSTOM_ROUTER_PATH`
- **Built-in scenario routing**:
  - `background`: Background tasks (typically lightweight models)
  - `think`: Thinking-intensive tasks (Plan Mode)
  - `longContext`: Long context (exceeds `longContextThreshold` tokens)
  - `webSearch`: Web search tasks
  - `image`: Image-related tasks

Token calculation uses `tiktoken` (cl100k_base) to estimate request size.

### 2. Transformer System

The project uses the `@wengine-ai/llms` package (external dependency) to handle request/response transformations. Transformers adapt to different provider API differences:

- Built-in transformers: `anthropic`, `deepseek`, `gemini`, `openrouter`, `groq`, `maxtoken`, `tooluse`, `reasoning`, `enhancetool`, etc.
- Custom transformers: Load external plugins via `transformers` array in `config.json`

Transformer configuration supports:
- Global application (provider level)
- Model-specific application
- Option passing (e.g., `max_tokens` parameter for `maxtoken`)

### 3. Agent System (packages/server/src/agents/)

Agents are pluggable feature modules that can:
- Detect whether to handle a request (`shouldHandle`)
- Modify requests (`reqHandler`)
- Provide custom tools (`tools`)

Built-in agents:
- **imageAgent**: Handles image-related tasks

Agent tool call flow:
1. Detect and mark agents in `preHandler` hook
2. Add agent tools to the request
3. Intercept tool call events in `onSend` hook
4. Execute agent tool and initiate new LLM request
5. Stream results back

### 4. SSE Stream Processing

The server uses custom Transform streams to handle Server-Sent Events:
- `SSEParserTransform`: Parses SSE text stream into event objects
- `SSESerializerTransform`: Serializes event objects into SSE text stream
- `rewriteStream`: Intercepts and modifies stream data (for agent tool calls)

### 5. Configuration Management

Configuration file location: `~/.claude-code-router/config.json`

Key features:
- Supports environment variable interpolation (`$VAR_NAME` or `${VAR_NAME}`)
- JSON5 format (supports comments)
- Automatic backups (keeps last 3 backups)
- Hot reload requires service restart (`ccr restart`)

Configuration validation:
- If `Providers` are configured, both `HOST` and `APIKEY` must be set
- Otherwise listens on `0.0.0.0` without authentication

### 6. Logging System

Two separate logging systems:

**Server-level logs** (pino):
- Location: `~/.claude-code-router/logs/ccr-*.log`
- Content: HTTP requests, API calls, server events
- Configuration: `LOG_LEVEL` (fatal/error/warn/info/debug/trace)

**Application-level logs**:
- Location: `~/.claude-code-router/claude-code-router.log`
- Content: Routing decisions, business logic events

## CLI Commands

```bash
ccr start      # Start server
ccr stop       # Stop server
ccr restart    # Restart server
ccr status     # Show status
ccr code       # Execute claude command
ccr model      # Interactive model selection and configuration
ccr preset     # Manage presets (export, install, list, info, delete)
ccr activate   # Output shell environment variables (for integration)
ccr ui         # Open Web UI
ccr statusline # Integrated statusline (reads JSON from stdin)
```

### Preset Commands

```bash
ccr preset export <name>      # Export current configuration as a preset
ccr preset install <source>   # Install a preset from file, URL, or name
ccr preset list               # List all installed presets
ccr preset info <name>        # Show preset information
ccr preset delete <name>      # Delete a preset
```

## Subagent Routing

Use special tags in subagent prompts to specify models:
```
<CCR-SUBAGENT-MODEL>provider,model</CCR-SUBAGENT-MODEL>
Please help me analyze this code...
```

## Preset System

The preset system allows users to save, share, and reuse configurations easily.

### Preset Structure

Presets are stored in `~/.claude-code-router/presets/<preset-name>/manifest.json`

Each preset contains:
- **Metadata**: name, version, description, author, keywords, etc.
- **Configuration**: Providers, Router, transformers, and other settings
- **Dynamic Schema** (optional): Input fields for collecting required information during installation
- **Required Inputs** (optional): Fields that need to be filled during installation (e.g., API keys)

### Core Functions

Located in `packages/shared/src/preset/`:

- **export.ts**: Export current configuration as a preset directory
  - `exportPreset(presetName, config, options)`: Creates preset directory with manifest.json
  - Automatically sanitizes sensitive data (api_key fields become `{{field}}` placeholders)

- **install.ts**: Install and manage presets
  - `installPreset(preset, config, options)`: Install preset to config
  - `loadPreset(source)`: Load preset from directory
  - `listPresets()`: List all installed presets
  - `isPresetInstalled(presetName)`: Check if preset is installed
  - `validatePreset(preset)`: Validate preset structure

- **merge.ts**: Merge preset configuration with existing config
  - Handles conflicts using different strategies (ask, overwrite, merge, skip)

- **sensitiveFields.ts**: Identify and sanitize sensitive fields
  - Detects api_key, password, secret fields automatically
  - Replaces sensitive values with environment variable placeholders

### Preset File Format

**manifest.json** (in preset directory):
```json
{
  "name": "my-preset",
  "version": "1.0.0",
  "description": "My configuration",
  "author": "Author Name",
  "keywords": ["openai", "production"],
  "Providers": [...],
  "Router": {...},
  "schema": [
    {
      "id": "apiKey",
      "type": "password",
      "label": "OpenAI API Key",
      "prompt": "Enter your OpenAI API key"
    }
  ]
}
```

### CLI Integration

The CLI layer (`packages/cli/src/utils/preset/`) handles:
- User interaction and prompts
- File operations
- Display formatting

Key files:
- `commands.ts`: Command handlers for `ccr preset` subcommands
- `export.ts`: CLI wrapper for export functionality
- `install.ts`: CLI wrapper for install functionality

## Dependencies

```
cli → server → shared
server → @wengine-ai/llms (core routing and transformation logic)
ui (standalone frontend application)
```

## Development Notes

1. **CCR service management**: Always use `ccr restart` instead of `ccr stop` followed by `ccr start`. Stopping the service interrupts all active LLM routing, making the current Claude Code session unusable. Restarting minimizes downtime.
2. **Node.js version**: Requires >= 18.0.0
2. **Package manager**: Uses pnpm (monorepo depends on workspace protocol)
3. **TypeScript**: All packages use TypeScript, but UI package is ESM module
4. **Build tools**:
   - cli/server/shared: esbuild
   - ui: Vite + TypeScript
5. **@wengine-ai/llms**: This is an external dependency package providing the core server framework and transformer functionality, type definitions in `packages/server/src/types.d.ts`
6. **Code comments**: All comments in code MUST be written in English
7. **Documentation**: When implementing new features, add documentation to the docs project instead of creating standalone md files

## Changelog & Release Notes Convention

When releasing a new version, keep changelog records in three places consistent:

1. **`CHANGELOG.md`**: The complete, detailed changelog (Keep a Changelog style, `Added`/`Changed`/`Fixed`). Every released version is recorded here permanently — never trim it.
2. **`README.md` / `README_en.md` changelog tables**: A bilingual, concise release-summary table that keeps **only the latest 10 versions**. Add the new version as the top row when releasing.
3. **`CHANGELOG-archive.md`**: When a README table would exceed 10 versions, move the oldest summary rows out of the README and into this archive (it has a `中文` section and an `English` section mirroring the two READMEs). This file holds the overflow summaries only; the full history still lives in `CHANGELOG.md`.

Release checklist: bump the `version` in all 6 `package.json` files (root + 5 packages) to the same value, prepend a new section to `CHANGELOG.md`, add the new top row to both README tables, and if either table now exceeds 10 rows move the oldest rows into `CHANGELOG-archive.md`.

**Automated gate**: `scripts/release.sh` runs `validate_release_docs` before publishing anything (all modes, including dry-run). It aborts the release unless: all 6 `package.json` versions equal the version being released, `CHANGELOG.md` has a non-empty `## [<version>]` section, both README tables have a `| **v<version>** |` row, and the version is strictly greater than the latest published on npm (numeric per-segment compare; skipped with a warning if the registry is unreachable).

### Version numbering

Daily/minor iterations may extend the patch segment with an extra digit (e.g. `2.3.23` → `2.3.231` → `2.3.232`) to keep headline version numbers from inflating. Rules (enforced by the release gate's monotonic check):

- Patch segments compare **numerically**, so once a `2.3.23x` version ships, `2.3.24` would be a downgrade (`24 < 231`) and is rejected. The next "feature" version after `2.3.23x` is `2.3.240` (then `2.3.241`… for its dailies), or bump the minor to `2.4.0` for a clean number.
- Stable releases must never carry a pre-release suffix. Pre-release validation versions (`2.3.x-beta.0`, published manually with `npm publish --tag beta`) must NOT go through `scripts/release.sh`: both the release gate and the CLI's update check (`compareVersions` in `packages/cli/src/utils/update.ts`) do plain numeric segment comparison and do not understand suffixes.

## Configuration Example Locations

- Main configuration example: Complete example in README.md
- Custom router example: `custom-router.example.js`

## Agent Workflow
- **Parallel execution**: For development tasks, default to dispatching multiple developer subAgents in parallel (e.g., `voltagent-core-dev:backend-developer`, `voltagent-lang:java-architect`) to maximize efficiency
- After code completion: call `code-reviewer` for review
- For security-sensitive code: call `security-auditor`
- For core architecture changes: call `architect-reviewer`

---
> Source: [xiaoliu10/claude-code-router-next](https://github.com/xiaoliu10/claude-code-router-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
