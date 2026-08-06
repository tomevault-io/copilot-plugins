## command

> This project is a Lua-based Balatro mod. There are no formal build, lint, or test commands. The mod is loaded directly by the game.

## Build/Lint/Test Commands

This project is a Lua-based Balatro mod. There are no formal build, lint, or test commands. The mod is loaded directly by the game.

- **Running the mod**: Place the `command` directory into the Balatro mods folder.

## Code Style Guidelines

### Imports

- All modules are loaded at the start of main.lua, you do not use require
- Use globals in modules

### Formatting

- Indent with 4 spaces.
- Use `local` keyword for local variables.
- Tables can be defined with a newline after the opening brace and before the closing brace, with each key-value pair on a new line.

### Naming Conventions

- **Variables**: `snake_case`
- **Functions**: `snake_case`
- **Modules/Files**: `snake_case.lua`
- **Constants**: `UPPER_SNAKE_CASE`

### Error Handling

- Use `error()` for critical errors that should halt execution.
- For API responses, use `server:error_response()` with appropriate HTTP status codes and messages.
- Use `pcall` for protected calls to functions that might error, especially when dealing with external libraries like `json.decode`.

### Logging

- Use the `Logger` module for all informational, warning, and error messages.
- `Logger:debug(message, component)`: For detailed debugging information.
- `Logger:info(message, component)`: For general information about the application flow.
- `Logger:warn(message, component)`: For potential issues that don't halt execution.
- `Logger:error(message, component)`: For errors that prevent a specific operation from completing.

---
> Source: [lotrez/command](https://github.com/lotrez/command) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
