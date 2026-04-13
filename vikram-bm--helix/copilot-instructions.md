## helix

> - Setup: `npm install`

# CLAUDE.md - Guidelines for Claude & other agents

## Build & Run Commands
- Setup: `npm install`
- Run app: `npm start`
- Lint: `npm run lint`
- Type check: `npm run typecheck`
- Test all: `npm test`
- Test single: `npm test -- -t "test name"`
- Alternative test: `pytest path/to/test_file.py::test_function`

## Code Style Guidelines
- **Formatting**: Prettier with 100 character line limit
- **Imports**: Group imports (react, third-party, local) with blank lines between
- **Types**: Use TypeScript types/interfaces for all functions and components
- **Naming**:
  - camelCase for variables, functions, instances
  - PascalCase for classes, components, types, interfaces
  - UPPER_SNAKE_CASE for constants
- **Error handling**: Use try/catch blocks or error boundaries as appropriate
- **Documentation**: JSDoc comments for all public functions and components
- **Testing**: Write unit tests for all new functionality

## Folder Structure
- autogen_flows/: AutoGen workflow project
  - agents/: Agent definitions and logic
  - app/: Application code
  - config/: Configuration files
  - flows/: Flow definitions
  - utils/: Shared utilities

## Allowed Commands 
```json
{
  "allowed_command_prefixes": ["npm ", "pytest ", "git "]
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vikram-BM)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Vikram-BM)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
