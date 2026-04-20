## jiang-clips

> Automated short-form clip extraction pipeline for YouTube videos using AI.

# AGENTS.md - jiang-clips

Automated short-form clip extraction pipeline for YouTube videos using AI.

## Project Overview

- **Runtime**: Bun (not Node.js)
- **Language**: TypeScript with strict mode
- **Module System**: ESNext with bundler module resolution
- **Package Manager**: Bun (uses bun.lock)

## Commands

```bash
# Run the pipeline
bun run src/index.ts pipeline <youtube-url>
bun run src/index.ts batch <channel-url>
bun run src/index.ts resume <run-id>
bun run src/index.ts status [run-id]
bun run src/index.ts clean <run-id>

# Testing
bun test                     # All tests
bun test tests/utils         # Single test directory
bun test tests/utils/fs.test.ts  # Single test file
bun test --reporter=verbose tests/  # Verbose output

# Linting & Formatting
oxlint                       # Lint src/ and tests/
oxfmt --write src tests      # Format and write changes
oxfmt --check src tests      # Check formatting without writing
```

## Code Style

### TypeScript
- Use `type` for simple type aliases, `interface` for complex types
- Prefer `export class` for public classes
- Use `enum` for related constants (e.g., `PipelineStage`, `StageStatus`)
- All functions must have explicit return types
- Use `unknown` instead of `any` for generic error handling

### Imports
Order imports separated by blank lines:
1. External packages (e.g., `chalk`, `commander`, `zod`)
2. Internal modules (e.g., `../utils/logger`, `./config`)
3. Type imports use `import type`

```typescript
import chalk from "chalk";
import { join } from "path";
import { createLogger } from "../utils/logger";
import type { VideoMetadata } from "../pipeline/types";
```

### Naming Conventions
- **Files**: kebab-case (e.g., `video-processor.ts`, `clip-identifier.ts`)
- **Classes**: PascalCase (e.g., `PipelineOrchestrator`, `CheckpointManager`)
- **Functions/variables**: camelCase (e.g., `extractVideoId`, `runId`)
- **Enums**: PascalCase with UPPER_SNAKE values (e.g., `PipelineStage.DOWNLOAD`)
- **Types/Interfaces**: PascalCase (e.g., `VideoMetadata`, `ClipCandidate`)
- **Constants**: camelCase or UPPER_SNAKE (logger instances use camelCase module names)

### Error Handling
- Throw `Error` objects with descriptive messages
- Wrap async operations in try/catch with proper cleanup in `finally` blocks
- Log errors before re-throwing: `log.error(\`Failed: ${err}\`); throw err;`
- Handle subprocess failures with exit code checks

```typescript
try {
  const result = await someAsyncOperation();
  log.info(`Success: ${result}`);
} catch (err) {
  log.error(`Operation failed: ${err}`);
  throw err;
} finally {
  cleanup();
}
```

### Path Aliases
Configured in `tsconfig.json`:
```typescript
import { createLogger } from "@/utils/logger";  // resolves to ./src/utils/logger
```

### Logging
Use the `createLogger` utility with module name:
```typescript
const log = createLogger("module-name");
log.info("message");
log.error("failed: ${err}");
```

Log levels: `debug`, `info`, `warn`, `error`

## Project Structure

```
src/
├── index.ts          # CLI entry point (commander)
├── config.ts        # Zod schema & config loader
├── pipeline/        # Pipeline orchestration
│   ├── orchestrator.ts
│   ├── checkpoint.ts
│   └── types.ts
├── modules/         # Processing modules
│   ├── downloader.ts
│   ├── transcriber.ts
│   ├── clip-identifier.ts
│   ├── video-processor.ts
│   └── caption-generator.ts
├── utils/           # Utilities
│   ├── logger.ts
│   ├── fs.ts
│   └── ffmpeg.ts
└── remotion/        # Video rendering
    └── types.ts

tests/
├── pipeline/
├── modules/
└── utils/
```

## Testing

Uses `bun:test` with `describe`/`test`/`expect`:
```typescript
import { describe, test, expect } from "bun:test";

describe("FeatureName", () => {
  test("does something", () => {
    expect(result).toBe(expected);
  });
});
```

## Configuration

Environment variables validated via Zod in `src/config.ts`:
- `GEMINI_API_KEY` (required)
- `WHISPER_MODEL` (tiny|base|small|medium|large, default: base)
- `MAX_PARALLEL_CLIPS` (1-10, default: 3)
- `SILENCE_THRESHOLD_DB` (default: -35)
- `OUTPUT_WIDTH`/`OUTPUT_HEIGHT` (default: 1080x1920)

## Dependencies

- **@google/genai**: AI content generation
- **@remotion/***: Video rendering
- **chalk**: Terminal colors
- **commander**: CLI framework
- **zod**: Schema validation

---
> Source: [ayush-that/jiang-clips](https://github.com/ayush-that/jiang-clips) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
