## joorappv2

> This project has separate backend and frontend codebases with their own rules.

# JoorApp V2 - Global Cursor Rules

## IMPORTANT: Always Check Directory-Specific Rules

This project has separate backend and frontend codebases with their own rules.

### Backend Rules (Node.js/Express)
**Location**: `nodeBE/.cursor/rules/`

Before writing any backend code, ALWAYS check and follow:
- `nodeBE/.cursor/rules/_master.mdc` - Critical backend rules
- `nodeBE/.cursor/rules/README.mdc` - Backend rules index
- `nodeBE/.cursor/rules/architecture.mdc` - Architecture patterns
- `nodeBE/.cursor/rules/logging.mdc` - Winston logging (NO console.log)
- `nodeBE/.cursor/rules/error-handling.mdc` - Error handling
- `nodeBE/.cursor/rules/module-structure.mdc` - Module creation
- `nodeBE/.cursor/rules/controller-splitting.mdc` - Controller splitting patterns
- `nodeBE/.cursor/rules/swagger.mdc` - Swagger documentation standards (CRITICAL for entity schemas)
- `nodeBE/.cursor/rules/model-creation.mdc` - Model creation guidelines (AuditableEntity)
- `nodeBE/.cursor/rules/master-data-entity.mdc` - Master data patterns (MasterDataEntity vs AuditableEntity)
- `nodeBE/.cursor/rules/repository-service-pattern.mdc` - Repository and service patterns
- `nodeBE/.cursor/rules/security.mdc` - Security practices
- `nodeBE/.cursor/rules/coding-standards.mdc` - Coding standards
- `nodeBE/.cursor/rules/testing.mdc` - Testing rules (test creation, file locations, standards)

**Key Backend Rules:**
- Use ES6 modules (import/export) - NO require()
- Use Winston logger - NO console.log
- Module-based architecture in `src/modules/`
- All routes under `/api/v2/{module}`
- NO hardcoded fallbacks for environment variables

### Frontend Rules (React)
**Location**: `ReactFE/.cursor/rules/`

Before writing any frontend code, ALWAYS check and follow:
- `ReactFE/.cursor/rules/_master.mdc` - Critical frontend rules
- `ReactFE/.cursor/rules/README.mdc` - Frontend rules index
- `ReactFE/.cursor/rules/component-structure.mdc` - Component patterns
- `ReactFE/.cursor/rules/coding-standards.mdc` - Coding standards
- `ReactFE/.cursor/rules/todo.mdc` - Task management

**Key Frontend Rules:**
- Use functional components with hooks
- Component-based architecture
- PascalCase for components, camelCase for functions
- Proper state management

## How to Use These Rules

1. **Identify the directory** you're working in (nodeBE or ReactFE)
2. **Read the master rule** first: `{directory}/.cursor/rules/_master.mdc`
3. **Check the README** for rule index: `{directory}/.cursor/rules/README.mdc`
4. **Follow specific rules** based on what you're building
5. **Reference rules explicitly** in chat: "Follow logging.mdc rules"

## When Writing Code

### For Backend (nodeBE/):
```
"Create a new module following:
- Architecture: nodeBE/.cursor/rules/architecture.mdc
- Logging: nodeBE/.cursor/rules/logging.mdc
- Module Structure: nodeBE/.cursor/rules/module-structure.mdc"

"Create a new model following:
- Model Creation: nodeBE/.cursor/rules/model-creation.mdc
- Master Data Entity: nodeBE/.cursor/rules/master-data-entity.mdc (if master data)
- Repository Pattern: nodeBE/.cursor/rules/repository-service-pattern.mdc"
```

### For Frontend (ReactFE/):
```
"Create a component following:
- Component Structure: ReactFE/.cursor/rules/component-structure.mdc
- Coding Standards: ReactFE/.cursor/rules/coding-standards.mdc"
```

## Project Structure

```
joorappV2/
├── .cursorrules          # This file (global rules reference)
├── nodeBE/               # Backend (Node.js/Express)
│   └── .cursor/rules/    # Backend-specific rules
└── ReactFE/              # Frontend (React)
    └── .cursor/rules/    # Frontend-specific rules
```

## Critical Reminders

- **ALWAYS** check directory-specific rules before writing code
- **NEVER** use console.log in backend (use Winston)
- **ALWAYS** use ES6 modules in backend (no require())
- **ALWAYS** follow module-based architecture in backend
- **ALWAYS** use functional components in frontend
- **ALWAYS** include requestId in backend logs

## Rule Enforcement

If you see code that violates these rules:
1. Point to the specific rule file
2. Ask to follow the pattern from that rule
3. Reference the rule explicitly: "Please follow nodeBE/.cursor/rules/logging.mdc"

---

**Remember**: These rules exist to maintain code quality and consistency. Always reference them when writing code!

---
> Source: [joorapp/joorappV2](https://github.com/joorapp/joorappV2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
