## vue-architecture-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vue3 + Pinia Kanban board application implementing Clean Architecture principles. Currently in pre-implementation phase with specification and architecture docs only.

## Architecture

This project follows Clean Architecture with explicit dependency injection:

```
View → Composable → UseCase → Domain
                        ↘ Port (interface)
                            ↑
                      Infrastructure
```

### Layer Responsibilities

| Layer | Location | Responsibility |
|-------|----------|----------------|
| Domain | `src/domain/` | Entity types, pure functions (no external dependencies) |
| Port | `src/repositories/` | Repository interfaces only |
| UseCase | `src/usecases/` | Pure functions with dependencies as arguments |
| Store | `src/stores/` | State + Getters + Mutations only (no business logic) |
| Infrastructure | `src/infrastructure/` | Repository implementations |
| Composable | `src/composables/` | Adapter layer - assembles dependencies and injects into UseCases |
| View | `src/views/`, `src/components/` | Templates and event bindings |

### Key Design Principles

1. **UseCases are pure functions** - All dependencies (Repository, Store) received as arguments, never imported directly
2. **Composables are adapters** - They assemble dependencies and inject them into UseCases
3. **Stores have no business logic** - Only state management, derived state, and mutations
4. **Dependency inversion** - UseCases depend on interfaces, implementations injected at runtime

### UseCase Pattern

```typescript
// Dependencies defined as type
export type CreateTaskDeps = {
  repository: TaskRepository
  store: BoardStore
}

// Pure function with deps as argument
export async function createTask(
  input: CreateTaskInput,
  deps: CreateTaskDeps
): Promise<Task | null> {
  const { repository, store } = deps
  const newTask = await repository.create(input)
  store.addTask(newTask)
  return newTask
}
```

### Testing Strategy

- **Domain**: Unit tests, no mocks needed (pure functions)
- **UseCase**: Unit tests with mock Repository/Store passed as arguments
- **Store**: Unit tests, no mocks needed
- **View**: E2E tests

## Tech Stack

- Vue 3 (Composition API, `<script setup>`)
- Pinia (Setup Store style)
- Vue Router 4
- TypeScript
- Vite

## Reference Documents

- `docs/ARCHITECTURE.md` - Detailed architecture explanation
- `docs/SPEC.md` - Application specification

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiraaaken)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hiraaaken)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
