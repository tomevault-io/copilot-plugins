## testbench

> This is a TypeScript CLI application called "TestBench" that provides automated testing capabilities for remote servers.

# TestBench CLI Tool

This is a TypeScript CLI application called "TestBench" that provides automated testing capabilities for remote servers.

## Purpose
- Run API calls against remote servers using bearer token authentication
- Execute Playwright tests against the configured server
- Provide a clean CLI interface for test automation workflows

## Architecture
- **Entry Point**: `src/index.ts` - Main CLI using Commander.js
- **Commands**: 
  - `update-server` - Updates remote server environment via API calls (non-interactive)
  - `smoketest` - Runs end-to-end tests against the configured server
  - `get-token` - Generates bearer tokens for server authentication
- **Configuration**: `config.ts` - Environment variable management
- **Actions**: `src/actions/` - Contains executable scripts for server setup and configuration
- **Playwright Actions**: `src/actions/playwright/` - Browser automation scripts for server setup when API is insufficient
- **End-to-End Tests**: `src/tests/` - Actual test suites that validate server functionality
- **Unit Tests**: `src/__tests__/` - Tests for internal project components (utils, actions, commands)

## Key Dependencies
- **commander**: CLI framework
- **playwright**: Browser automation for testing
- **axios**: HTTP client for API calls
- **dotenv**: Environment variable management
- **vitest**: Unit testing framework

## Environment Variables
- `SERVER_URL`: Remote server URL (required)
- `BEARER_TOKEN`: Authentication token (required)

## Development
- Use `npm run dev` for development with hot reload
- Use `npm run build` to compile TypeScript
- Use `npm start` to run the compiled version
- Use `npm test` for unit tests (vitest)
- Use `npm run smoketest` for end-to-end tests (playwright)

## Commands and Scripts
- **`npm run update-server`**: Runs all scripts in `src/actions/` (excluding subfolders) to configure the server
- **`npm run update-server -- --script=configure-email`**: Runs only the specified script
- **`npm run smoketest`**: Runs end-to-end tests from `src/tests/` against the configured server
- **`npm run get-token`**: Runs Playwright script to generate bearer tokens for authentication
- **`npm test`**: Runs unit tests for project components in `src/__tests__/`

## Test Structure
- **`src/tests/`**: End-to-end tests that validate actual server functionality
- **`src/__tests__/`**: Unit tests for internal project components (utils, actions, commands)
- **`src/actions/playwright/`**: Browser automation scripts for server setup (not tests, but setup scripts with assertions)

## Update Server Command Behavior
- **Default**: `npm run update-server` runs all scripts in `src/actions/` (excluding subfolders)
- **Specific Script**: `npm run update-server -- --script=configure-email` runs only the specified script
- **Non-Interactive**: No user prompts or selections - fully automated execution
- **Sequential Execution**: When running all scripts, they execute one after another
- **Error Handling**: Stops execution on first error with clear error messages

## Best Practices
- Always validate environment variables on startup
- Use proper error handling and user-friendly messages
- Support verbose logging for debugging
- Follow TypeScript strict mode guidelines
- Use async/await for all asynchronous operations
- Actions should be self-contained and not require user interaction
- End-to-end tests should focus on actual functionality validation
- Unit tests should focus on internal component behavior
- Playwright actions are for setup/configuration, not testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ProcessMaker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
