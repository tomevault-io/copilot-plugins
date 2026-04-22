## dependencyinjectionsamples

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TypeScript educational project demonstrating **Dependency Injection (DI)** concepts through progressive examples. The codebase shows the evolution from tightly coupled code to proper dependency injection using interfaces.

## Common Commands

```bash
# Install dependencies
npm install

# Run demos for each DI stage
npm run demo:01  # Without DI (tightly coupled)
npm run demo:02  # Basic DI (constructor injection)
npm run demo:03  # Interface-based DI (best practice)

# TypeScript compilation (manual)
npx tsc

# Run specific examples directly
npx ts-node src/01-without-di/main.ts
npx ts-node src/02-basic-di/main.ts
npx ts-node src/03-interface-di/main.ts
```

## Architecture

### Learning Progression
The project demonstrates DI evolution through numbered directories:

1. **01-without-di/**: Shows tightly coupled code where `UserService` creates `Logger` internally
2. **02-basic-di/**: Introduces constructor injection but still depends on concrete classes
3. **03-interface-di/**: Best practice using interfaces (`ILogger`) for true abstraction
4. **04-06-**/: Additional advanced patterns (directories exist but may be incomplete)

### Key Pattern
Each stage follows the same structure:
- `UserService.ts`: Main business logic class
- `Logger.ts`/`*Logger.ts`: Logging implementations
- `main.ts`: Demonstration of the pattern
- `interfaces/`: TypeScript interfaces (in later stages)

### Core Interfaces
- `ILogger`: Defines logging contract with `log(message: string): void`

### Implementation Strategy
When modifying this codebase:
- Follow the progressive learning structure
- Maintain the demonstration format in `main.ts` files
- Keep examples simple and focused on DI concepts
- Use consistent naming: `UserService`, `Logger`, `ILogger`
- Include educational comments explaining DI benefits/problems

## TypeScript Configuration

- Target: ES2020
- Strict mode enabled
- Source maps and declarations generated
- Output directory: `./dist`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manntera) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
