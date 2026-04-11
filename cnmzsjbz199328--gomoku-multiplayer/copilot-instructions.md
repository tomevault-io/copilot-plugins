## gomoku-multiplayer

> This standard aims to ensure consistency, maintainability, and high performance of the codebase. All developers must strictly adhere to it.

# Gomoku Multiplayer Coding Standards (Gemini.md)

This standard aims to ensure consistency, maintainability, and high performance of the codebase. All developers must strictly adhere to it.

## 1. Code Organization Principles
- **Separation of Concerns (SoC)**: Separate frontend views from logic (Stores/Utils); strictly decouple backend routes, business logic (Services), and data models (Models).
- **Directory Consistency**: Follow the established directory structure. Components must be placed in `components`, and business logic in `services` or `stores`.
- **Single Source of Truth (SSOT)**: The game state should be maintained by the authoritative backend server. The frontend only acts as a reflection of the state and a collector of user input.
- **File Size Constraint**: Maintain high cohesion and low coupling. Each file should ideally not exceed **200 lines of code** to ensure readability and focus.

## 2. Naming Conventions
- **File Naming**:
  - Components: PascalCase, e.g., `GameBoard.tsx`.
  - Logic/Utility files: camelCase or kebab-case, e.g., `useGameLogic.ts`.
- **Variables and Functions**: Use camelCase. Boolean values should have prefixes like `isGameOver`, `hasPlayerJoined`.
- **Constants and Enums**: Uppercase with underscores, e.g., `MAX_BOARD_SIZE`, `GAME_STATUS_WAITING`.
- **WebSocket Events**: Use verb-noun phrases separated by colons or slashes, e.g., `room:join`, `move:make`, `game:start`.

## 3. Commenting Requirements
- **JSDoc**: All public functions and Service methods must include JSDoc comments describing parameters, return values, and exceptions.
- **Logic Explanation**: Complex algorithms (e.g., Gomoku winning check) must be explained in detail above the code block.
- **TODO Standard**: Use `// TODO(name): description` to mark incomplete items.

## 4. Modularization Standards
- **High Cohesion**: A module should only be responsible for a single functional point (e.g., `WinChecker` only handles winning conditions).
- **Low Coupling**:
  - Modules communicate through interfaces; avoid direct access to internal states.
  - Frontend components should be designed as controlled components as much as possible, reducing internal state.
  - Backend Service layers should not directly depend on `req/res` objects and should only process pure business data.

## 5. Type Safety (TypeScript)
- **No `any`**: The use of `any` is strictly prohibited. All API responses and WebSocket message payloads must have defined Interfaces.
- **Shared Types**: Define shared types in `shared/types` (if applicable) or the `types` directory of each side.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cnmzsjbz199328)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cnmzsjbz199328)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
