## codumentation

> Codumentation is a documentation validation tool that treats documentation as code.

# Codumentation - AI Context for Claude

## Project Overview


Codumentation is a documentation validation tool that treats documentation as code.
It enforces a strict validation layer between the codebase state and written documentation.

**Core Philosophy**: "Codify Your Documentation"

Turn documentation claims into provable assertions. Every statement in your docs can be backed by validation code - if reality changes, your documentation build fails instead of silently becoming outdated.


**Note for AI Assistants**: This repository uses codumentation for its documentation. All .md files with corresponding .codx folders (e.g., CLAUDE.md.codx/, README.md.codx/) are generated from TypeScript modules. When making documentation changes:
1. Edit the .codx modules, not the generated .md files directly
2. Each .codx module exports: `content` (text), `validate` (verification function), and `errorContent` (error message)
3. Run `npm run codumentation:validate` to verify changes before committing



## Architecture


### Core Components

1. **Discovery** (`src/discovery.ts`): Finds all .codx directories and parses templates
2. **Loader** (`src/loader.ts`): Compiles and loads TypeScript modules at runtime using ts-node
3. **Renderer** (`src/renderer.ts`): Template engine for variable substitution
4. **Validator** (`src/validator.ts`): Orchestrates validation and build processes
5. **Logger** (`src/logger.ts`): Logs validation failures to .codumentation.log
6. **CLI** (`src/cli.ts`): Command-line interface

### Module Interface

Each .codx module must export:
- `content`: string - The content to inject
- `validate`: async function - Throws if validation fails
- `errorContent`: string - Human-readable error explanation


### Exported Functions

Each core module exports the following key functions:

- **discovery.ts**: `discoverCodxTargets`, `extractVariables`
- **loader.ts**: `loadCodxModule`, `loadAllModules`
- **renderer.ts**: `renderTemplate`, `generateDiff`
- **validator.ts**: `validateTarget`, `buildTarget`
- **logger.ts**: `logValidationFailure`, `clearLog`, `readLog`


### Public API

The package exports all core functionality from `src/index.ts`:

- Types: `CodxModule`, `CodxTarget`, `BuildResult`, `ValidationResult`, `LogEntry`
- Discovery: `discoverCodxTargets`, `extractVariables`
- Loader: `loadCodxModule`, `loadAllModules`
- Renderer: `renderTemplate`, `generateDiff`
- Validator: `validateTarget`, `buildTarget`
- Logger: `logValidationFailure`, `clearLog`, `readLog`


### CLI Commands

The CLI (src/cli.ts) provides two main commands:

1. **validate** - Validates all documentation against the codebase
2. **build** - Builds/regenerates all documentation files

Both commands are implemented using Commander.js and properly registered in the program.


## Key Principles


1. **Documentation must be verified by code**: Never trust documentation; always validate it
2. **Fail fast**: If docs are out of sync, CI should fail immediately
3. **Self-hosting**: Codumentation uses itself to validate its own documentation
4. **Explicit is better than implicit**: Each claim in docs should have a corresponding validation
5. **Logging for improvement**: Track which validations fail frequently to identify brittleness


## Development Commands


- **Build**: `npm run build` - Compile TypeScript to dist/
- **Test**: `npm test` - Run test suite with Vitest
- **Validate Docs**: `npm run codumentation:validate` - Check docs are up-to-date
- **Build Docs**: `npm run codumentation:build` - Regenerate documentation files

**IMPORTANT**: After making any changes to the codebase, always run `npm run codumentation:validate` to ensure the documentation remains valid and in sync with the code. This is a core principle of the "Validate First, Generate Second" philosophy - if validation fails, the documentation needs to be updated before proceeding.


### Test Coverage

The project maintains comprehensive test coverage with the following test files:

- **tests/discovery.test.ts** - Tests for extractVariables
- **tests/renderer.test.ts** - Tests for renderTemplate and generateDiff
- **tests/integration.test.ts** - End-to-end integration tests using discoverCodxTargets, validateTarget, and buildTarget

All tests use Vitest and can be run with `npm test`.


## File Structure


```
codumentation/
├── src/
│   ├── types.ts          # Type definitions
│   ├── discovery.ts      # Find .codx directories
│   ├── loader.ts         # Load and validate modules
│   ├── renderer.ts       # Template rendering
│   ├── validator.ts      # Validation orchestration
│   ├── logger.ts         # Logging system
│   ├── cli.ts           # CLI entry point
│   └── index.ts         # Public API exports
├── tests/
│   ├── discovery.test.ts
│   ├── renderer.test.ts
│   └── integration.test.ts
├── README.md.codx/      # Self-hosted README
├── CLAUDE.md.codx/      # Self-hosted AI context
└── package.json
```

---
> Source: [benquemax/codumentation](https://github.com/benquemax/codumentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
