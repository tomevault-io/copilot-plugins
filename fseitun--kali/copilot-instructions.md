## kali

> TypeScript best practices and type safety requirements


# TypeScript Standards

## Strict Mode Requirements

- Use TypeScript in strict mode (ES2022)
- Enable all compiler warnings
- Fix all type errors before committing
- Run `npm run type-check` regularly

## Type Safety Best Practices

### Avoid `any`
- ❌ Never use `any` in production code
- ✅ Use proper types or `unknown` with type guards
- ✅ Create interfaces for complex structures

### Proper Error Handling
```typescript
try {
  // risky operation
} catch (error) {
  if (error instanceof Error) {
    logger.error('Operation failed', error.message)
  }
}
```

### Type Guards
```typescript
function isValidAction(action: unknown): action is PrimitiveAction {
  return (
    typeof action === 'object' &&
    action !== null &&
    'action' in action
  )
}
```

### Interface Over Type for Objects
```typescript
// ✅ Preferred
interface GameState {
  phase: GamePhase
  turn: string
}

// ❌ Avoid for object shapes
type GameState = {
  phase: GamePhase
  turn: string
}
```

## Testing Standards

Test file flexibility (use of `any`, `@ts-nocheck`, etc.): see `development-guidelines.mdc` "Test File Standards" for the full rules.

## Common Patterns

### Path-Based State Access
```typescript
const value = stateManager.get('players.p1.position')
// Returns: unknown (must validate/cast)
```

### State Mutations (Orchestrator Only)
```typescript
await stateManager.set('players.p1.position', newPosition)
// Only called from orchestrator
```

### LLM Client Interface
```typescript
interface LLMClient {
  generateResponse(
    systemPrompt: string,
    userMessage: string
  ): Promise<string>
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fseitun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
