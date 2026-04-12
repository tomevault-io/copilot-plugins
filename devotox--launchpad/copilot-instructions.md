## launchpad

> File organization and code structure standards for Launchpad CLI


# Code Organization Standards

## File Organization
- Keep all files under **500 lines maximum**
- Keep all functions under **100 lines maximum**
- Keep cyclomatic complexity under **20**
- Break large files into focused modules using directories
- Each command should be a directory with multiple focused files

## Code Structure
- Break complex functions into smaller, focused utilities
- Use dependency injection and composition over inheritance
- Each file should have a **single responsibility**
- Group related functionality into directories

## Naming Conventions
- Use **PascalCase** for classes and types
- Use **camelCase** for functions and variables
- Use **kebab-case** for file and directory names
- Use descriptive, intention-revealing names

## Examples

✅ **Good file structure:**
```
commands/
├── deploy/
│   ├── deploy-command.ts
│   ├── deployment-validator.ts
│   └── deployment-logger.ts
└── build/
    ├── build-command.ts
    └── build-optimizer.ts
```

✅ **Good naming:**
```typescript
class DatabaseConnectionManager {} // PascalCase
const validateUserInput = () => {} // camelCase
// file: user-authentication.ts     // kebab-case
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devotox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
