## chrome-cdp-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chrome CDP Tools is a TypeScript-powered Chrome DevTools Protocol (CDP) logging and debugging toolkit for web development. It provides automated browser log capture, console output monitoring, network request tracking, and JavaScript error logging during development.

## Key Components

- **`browser-logger.ts`**: Core TypeScript logger using Playwright to connect to Chrome's CDP and capture browser events (console, network, errors, navigation)
- **`start.sh`**: Main orchestration script that manages server startup, Chrome debugging setup, and logger initialization
- **Profile isolation**: Uses dedicated Chrome profile at `~/chrome-cdp-tools/profile` to avoid conflicts with regular browsing

## Development Commands

### Main Commands
```bash
# Start development environment (dev server + Chrome + logging)
./start.sh

# Build and start production environment
./start.sh --server=build

# Debug existing server (if port 3000 already in use)
./start.sh
```

### Log Monitoring
```bash
# Monitor server output
tail -f ~/chrome-cdp-tools/server.log

# Monitor browser activity
tail -f ~/chrome-cdp-tools/browser.log
```

### Manual Process Control
```bash
# Stop all processes
pkill -f "browser-logger"
pkill -f "pnpm dev"
pkill -f "pnpm start"

# Check Chrome debugging port
curl localhost:9222/json/version

# Check if port 3000 is in use
lsof -i :3000
```

## Architecture

### Core Flow
1. Script checks for existing server on port 3000
2. Starts development or production server if needed (using pnpm)
3. Launches Chrome with remote debugging enabled on port 9222
4. Starts TypeScript logger that connects to Chrome CDP
5. All browser activity is captured and logged in real-time

### Key Technical Details

- **Target Application**: Assumes web applications running on `localhost:3000`
- **Chrome CDP Port**: Uses port 9222 for Chrome DevTools Protocol communication
- **Package Manager**: Expects projects to use `pnpm` with `dev`, `build`, and `start` scripts
- **Log Format**: Timestamped entries with categorized event types (CONSOLE, NETWORK, NAVIGATION, etc.)
- **Playwright Integration**: Uses Playwright's CDP connection for robust browser monitoring

### Event Monitoring

The browser logger captures:
- Console messages (log, error, warn, etc.)
- Page errors and stack traces
- Network requests and responses (including errors)
- Page navigation events
- New tab/page creation

## Dependencies

- **playwright**: For Chrome CDP connection and browser automation
- **tsx**: For TypeScript execution
- **@types/node**: TypeScript definitions
- **Google Chrome**: Must be installed at default macOS location

## Platform Requirements

- **Node.js 16+** with npm/pnpm
- **macOS**: Script uses macOS-specific Chrome path
- **Chrome**: Installed at `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`

## Usage Patterns

This tool is designed to be run from any web project directory. It expects the target project to have standard npm/pnpm scripts and be configured to run on port 3000. The tool provides comprehensive browser debugging without interfering with the host project's dependencies or configuration.

---
> Source: [elsigh/chrome-cdp-tools](https://github.com/elsigh/chrome-cdp-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
