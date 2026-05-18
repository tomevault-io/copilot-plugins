## mcp-server-apple-events

> This project is a Model Context Protocol (MCP) server that provides native integration with Apple Reminders and Apple Calendar on macOS. It allows AI agents to read, create, update, and delete reminders and calendar events, manage lists, and organize tasks using natural language.

# Apple Events MCP Server

## Project Overview
This project is a Model Context Protocol (MCP) server that provides native integration with Apple Reminders and Apple Calendar on macOS. It allows AI agents to read, create, update, and delete reminders and calendar events, manage lists, and organize tasks using natural language.

**Key Technologies:**
*   **Runtime:** Node.js (TypeScript)
*   **Native Integration:** Swift (EventKit)
*   **Protocol:** Model Context Protocol (MCP) SDK
*   **Validation:** Zod
*   **Testing:** Jest
*   **Linting/Formatting:** Biome

## Architecture
The project follows a 4-layer Clean Architecture:
1.  **Server Layer** (`src/server/`): Handles MCP protocol communication and request routing.
2.  **Handlers Layer** (`src/tools/handlers/`): Business logic for specific tools (reminders, calendars).
3.  **Utils/Repository Layer** (`src/utils/`): Helper functions, validation, and data access patterns.
4.  **Native Bridge** (`src/swift/`): A Swift binary (`EventKitCLI`) that directly interacts with the macOS EventKit API. This is crucial for handling permissions and accessing system data reliably.

## Building and Running

### Prerequisites
*   Node.js 18+
*   macOS (for EventKit integration)
*   Xcode Command Line Tools (for Swift compilation)
*   pnpm

### Commands
*   **Install Dependencies:** `pnpm install`
*   **Build (TS & Swift):** `pnpm build` (Required before starting)
*   **Build Swift Only:** `pnpm build:swift`
*   **Start Server:** `pnpm start` (Runs via stdio)
*   **Development Mode:** `pnpm dev`
*   **Run Tests:** `pnpm test`
*   **Lint & Format:** `pnpm lint` (Uses Biome)

## Available Tools
The server exposes 5 main tools:

1.  **`reminders_tasks`**: CRUD operations for individual reminders (create, read, update, delete). Supports filtering by list, due date, etc.
2.  **`reminders_lists`**: Manage reminder lists (create, read, update, delete).
3.  **`calendar_events`**: CRUD operations for calendar events. Supports time blocking and scheduling.
4.  **`calendar_calendars`**: Read-only access to available calendars.
5.  **`reminders_subtasks`**: Manages subtasks/checklists within reminders (create, read, update, delete, toggle, reorder).

## Permission Handling
The server implements a two-layer permission prompt strategy to ensure reliability in non-interactive environments:
1.  **Proactive AppleScript Prompt**: On first access, an AppleScript command is triggered to force the permission dialog to appear.
2.  **Swift Binary Check**: The binary checks authorization status via EventKit.
3.  **Retry Strategy**: If a permission error occurs, the system attempts to recover using the AppleScript fallback.

## Critical Constraints
*   **macOS Only**: Strictly requires macOS with the EventKit framework.
*   **Binary Security**: The Swift binary location is validated to prevent execution from unauthorized paths.
*   **Date Formats**: Favors `YYYY-MM-DD HH:mm:ss` for local time and ISO 8601 for UTC.

## Development Guidelines

### Coding Style
*   **Language:** TypeScript (NodeNext) & Swift.
*   **Formatting:** Strictly follow Biome configuration (`biome.json`). Single quotes, space indentation.
*   **Imports:** Organize imports.

### Testing
*   **Framework:** Jest.
*   **Scope:** Unit tests for TypeScript logic, integration tests for the Swift bridge (mocked or actual).
*   **Command:** `pnpm test`

### Contribution
*   **Commits:** Follow Conventional Commits (e.g., `feat:`, `fix:`, `chore:`).
*   **Workflow:** Ensure `pnpm build` and `pnpm test` pass before committing.

## Key Files
*   `src/index.ts`: Entry point.
*   `src/server/server.ts`: Server configuration.
*   `src/swift/EventKitCLI.swift`: Native Swift bridge implementation.
*   `src/tools/definitions.ts`: MCP tool schema definitions.
*   `src/tools/handlers/subtaskHandlers.ts`: Subtask business logic.
*   `package.json`: Project configuration and scripts.

---
> Source: [FradSer/mcp-server-apple-events](https://github.com/FradSer/mcp-server-apple-events) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
