## spruthub-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development Commands
- `npm run build` - Compile TypeScript to dist/ directory
- `npm run dev` - Run CLI in development mode using tsx
- `npm run test` - Run Jest tests
- `npm run lint` - Run ESLint on TypeScript files
- `npm run lint:fix` - Run ESLint with auto-fix
- `npm run prepublishOnly` - Full build, lint, and test pipeline (runs before npm publish)

### Local Development
- Use `npm run dev -- <command>` to test CLI commands during development
- Example: `npm run dev -- login` or `npm run dev -- hub list`

## Architecture

This is a TypeScript CLI tool built with ES modules that provides a command-line interface for Spruthub smart home devices.

### Core Architecture Components

1. **Dynamic Command System**: Commands are automatically generated from the `spruthub-client` library's schema system. The CLI discovers available API methods at runtime and creates corresponding CLI commands.

2. **Command Structure**:
   - **Core Commands** (`src/commands/core/`): Static commands like `login`, `status`, `logout`, `push`, `pull`
   - **Dynamic Commands** (`src/commands/dynamic/`): Auto-generated from spruthub-client API schema (hub, accessory, scenario, etc.)

3. **Client Wrapper** (`src/utils/client.ts`): Unified interface to spruthub-client with connection management, error handling, and method discovery.

4. **Configuration Management** (`src/config/manager.ts`): Handles profiles, credentials (via keytar), and settings stored in `~/.spruthub/`.

5. **Output Formatting** (`src/utils/formatter.ts`): Supports multiple output formats (JSON, YAML, table) via `--format` flag.

### Key Technical Details

- **ES Modules**: Uses `"type": "module"` with modern import/export syntax
- **TypeScript**: Strict mode with comprehensive type checking enabled (ES2022 target)
- **Node.js**: Requires Node.js >=20.0.0
- **Secure Credentials**: Uses keytar for encrypted credential storage in system keychain  
- **Commander.js**: For CLI argument parsing and command organization
- **WebSocket Client**: Via spruthub-client for real-time communication with devices
- **Testing**: Jest for unit tests (though test files should be created as needed)

### Command Generation Flow

1. CLI loads schema from spruthub-client on startup
2. Categories (hub, accessory, scenario, etc.) become top-level commands  
3. Methods within categories become subcommands
4. Parameters from schema become CLI options with proper types
5. Positional arguments are automatically detected for required parameters

### Configuration

- Config stored in `~/.spruthub/config.json`
- Passwords encrypted in system keychain via keytar
- Supports multiple device profiles via `--profile` flag
- Global options: `--verbose`, `--format <json|yaml|table>`

### Error Handling

- Global uncaught exception handlers in src/cli.ts
- Performance monitoring with response time display
- Comprehensive debug logging with `--verbose` flag
- Automatic client disconnection after command execution

## Code Style & Linting

- **ESLint Configuration**: Uses @typescript-eslint with strict rules
- **Unused Variables**: Must be prefixed with `_` if intentionally unused
- **Type Safety**: `noImplicitAny`, `strictNullChecks`, and other strict TypeScript options enabled
- **Module Boundaries**: Explicit function return types and module boundary types are optional

---
> Source: [shady2k/spruthub-cli](https://github.com/shady2k/spruthub-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
