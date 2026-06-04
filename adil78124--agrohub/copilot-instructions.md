## architecture

> The project is built with Next.js 13+ (App Router) and follows an adapted Crystal Architecture.


# Project Rules

## Project Architecture

The project is built with Next.js 13+ (App Router) and follows an adapted Crystal Architecture.

## General Rules

- All code is stored in the `src/` folder.
- Code is organized into modules inside `src/modules/`.
- Each module is isolated and contains its own business logic, UI, and supporting parts.
- Shared and reusable code is stored in `src/shared/`.
- All tests must be written (unit, component, integration, e2e).
- Unit/Component tests are placed next to the file being tested inside a `__tests__/` folder.
- Integration/E2E tests are located in the `tests/` folder at the root of the project.

## Module Structure

src/modules/{moduleName}/
ui/
components/ # presentational components, no business logic, props only, can use local state/effects
widgets/ # smart components, connect UI with business logic, can use hooks (including orval hooks)
schemas/ # validation schemas (zod), DTOs, data types
utils/ # helper functions for the module
hooks/ # custom hooks at module level
constants/ # constants, enums, dictionaries
model/ # state management (zustand, jotai, redux slice)
index.ts # public API of the module (re-exports)

## Shared Structure

src/shared/
ui/ # shared UI components (Button, Modal, Table, Icon)
utils/ # shared utilities
hooks/ # shared hooks
constants/ # global constants and enums
config/ # global configurations (env, api endpoints)
lib/ # library adapters (axios, next-auth, orval client)

---
> Source: [Adil78124/AgroHub](https://github.com/Adil78124/AgroHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
