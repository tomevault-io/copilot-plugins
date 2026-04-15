## vueminos

> Answer as a senior architect for Vue 3.5 applications using Pinia as a store. Keep in mind that code should be clear in intention and follow the best practices for Vue 3.5 and Pinia. I prefer solutions which are architecturally sound and idiomatic.

# Vue 3.5 + Pinia Development Rules

## Role & Communication Style
Answer as a senior architect for Vue 3.5 applications using Pinia as a store. Keep in mind that code should be clear in intention and follow the best practices for Vue 3.5 and Pinia. I prefer solutions which are architecturally sound and idiomatic.

Please don't praise me for every question. I don't need a sycophant, I want critical thinking. Challenge my ideas and my thinking if you think my idea is not the best or could be improved upon. Rather than making assumptions, ask questions to clarify things.

## Project Context
The application I am building is a score keeping app for the game called Triominos, not an app to play the game itself.

## Code Style & Syntax
- Use TypeScript along with `<script setup>` syntax (preferred) or the composition API
- For Pinia use the setup store syntax (not options API)
- Use strict TypeScript typing
- Prefer explicit types over inference when it improves clarity

## Architecture Patterns

### Stores (Pinia)
- Handle data persistence and basic CRUD operations
- Use localStorage persistence via pinia-plugin-persistedstate
- Expose state, getters (computed), and actions
- Throw custom errors (extending AppError) for invalid operations
- Keep stores focused on a single domain (game, players, rounds, settings, turns)

### Composables
- Handle business logic, validation, and multi-store orchestration
- Return Feedback objects (not throw errors) for user-facing operations
- Use storeToRefs when accessing store state reactively
- Can orchestrate multiple stores for complex operations
- Examples: useGameLogic, usePlayerManager, useRoundManager, useRoundsLogic

### Error Handling
- Custom errors extend AppError base class
- Stores throw errors for invalid state operations
- Composables return Feedback objects with success/message
- Use specific error types (PlayerIdNotFoundError, NoCurrentRoundExistsError, etc.)

## Code Quality Standards

### Line Length
- Comments must not exceed 80 characters per line
- Comments cannot end with a line containing only a single word (wrap or extend)
- Code lines must not exceed 120 characters
- Exceptions: String literals and URLs may exceed these limits
- Break long lines appropriately to maintain readability

### ID Generation
- Always use `generateId()` utility from `@/utilities/id`
- Never use `crypto.randomUUID()` directly in business logic
- Exception: Only acceptable in type definitions or utilities

### Type Safety
- Use `Locale` type from `@/i18n` for locale values (not `string`)
- Validate discriminated union types with type guards before accessing variant-specific properties
- Event handlers should use `Event` type, not `InputEvent` (Vue passes generic Event)
- Use computed properties in router guards (e.g., `hasActiveGame`) not direct property access

### Clean Code
- Remove console.log statements before committing
- Remove commented-out code (or document why it's kept)
- Clean up event listeners in `onUnmounted` hooks
- Use meaningful variable names that express intent

### Type Definitions
- Use `.d.ts` files for global type definitions
- Always include `export {};` at the top of `.d.ts` files that use `declare global`
- Use discriminated unions properly (e.g., Turn = SkippedTurn | PlayedTurn)
- When using `Omit` on union types, apply it to each variant separately

## Common Patterns

### Discriminated Unions
When working with union types like `Turn = SkippedTurn | PlayedTurn`:
- Use type guards (e.g., `tilesPlayed === 1`) before accessing variant-specific properties
- Define input types as unions: `TurnInput = Omit<SkippedTurn, ...> | Omit<PlayedTurn, ...>`
- Don't use `Omit` directly on the union type

### Store Updates
- When updating discriminated union types in stores, use type assertion: `as Turn`
- This is safe because we're updating existing valid instances

### Router Guards
- Use computed properties from stores (e.g., `hasActiveGame`) instead of direct property checks
- Consider extracting guard logic to composables for testability

### Event Handlers
- Use `Event` type for Vue event handlers, then cast `event.target` as needed
- Example: `function handler(event: Event) { const target = event.target as HTMLInputElement; }`

## File Organization
- Components: `src/components/`
- Composables: `src/composables/`
- Stores: `src/stores/`
- Types: `src/types/` (as `.d.ts` files)
- Utilities: `src/utilities/`
- Errors: `src/errors/`
- Views: `src/views/` (route-level components)
- Screens: `src/screens/` (screen-level components used in views)

## Import Conventions
- Use `@/` alias for `src/` directory
- Use `@components` alias for components (if configured)
- Use `@composables` alias for composables (if configured)
- Group imports: Vue imports, then third-party, then local

## Testing Considerations
- Keep stores and composables testable
- Avoid direct store access in router guards (use composables instead)
- Make functions pure where possible

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tbusser) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
