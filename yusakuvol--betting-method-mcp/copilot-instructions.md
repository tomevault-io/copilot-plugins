## betting-method-mcp

> Instructions for GitHub Copilot when working on betting-method-mcp.

# GitHub Copilot Instructions

Instructions for GitHub Copilot when working on betting-method-mcp.

## Code Style

### TypeScript
- Use **double quotes** for strings
- Always use **semicolons**
- Use **2 spaces** for indentation
- Maximum line length: **100 characters**
- Always use **trailing commas** in arrays and objects
- Always use **parentheses** around arrow function parameters

### Example
```typescript
const example = (param: string): void => {
  console.log("This is the correct style");
};
```

## Naming Conventions

- **Classes**: `PascalCase` (e.g., `MonteCarloMethod`, `MartingaleMethod`)
- **Interfaces**: `PascalCase` ending with `State` (e.g., `MonteCarloState`)
- **Functions/Methods**: `camelCase` (e.g., `recordResult`, `initSession`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `DEFAULT_MAX_BET`)
- **MCP Tools**: `lowercase_snake_case` (e.g., `montecarlo_init`)

## Patterns to Follow

### 1. Betting Method Class Structure

Always implement these 5 methods:

```typescript
export class MethodNameMethod {
  private state: MethodNameState;

  constructor() {
    // Initialize with default values
  }

  initSession(/* parameters */): void {
    // 1. Validate all parameters
    // 2. Initialize state
    // 3. Set sessionActive to true
  }

  recordResult(result: BetResult): void {
    // 1. Check if sessionActive
    // 2. Update totalProfit
    // 3. Calculate next bet
    // 4. Update state
  }

  getState(): MethodNameState {
    // ALWAYS return a copy
    return { ...this.state };
  }

  reset(): void {
    // Reset to initial state
  }
}
```

### 2. State Interface Definition

All state interfaces must extend `BettingMethodState`:

```typescript
export interface MethodNameState extends BettingMethodState {
  // Add method-specific properties here
  baseUnit: number;
  // ... other properties
}
```

### 3. Parameter Validation

Always validate parameters:

```typescript
if (baseUnit <= 0) {
  throw new Error("baseUnit must be positive");
}
if (maxBet < baseUnit) {
  throw new Error("maxBet must be greater than or equal to baseUnit");
}
```

### 4. Session Activity Check

Check before processing:

```typescript
if (!this.state.sessionActive) {
  throw new Error("No active session. Please initialize a session first.");
}
```

### 5. State Immutability

Never expose internal state directly:

```typescript
// ❌ Wrong
getState() {
  return this.state;
}

// ✅ Correct
getState() {
  return { ...this.state };
}
```

## MCP Tool Registration

When adding new methods, register 4 tools:

```typescript
// In ListToolsRequestSchema handler
{
  name: "{method}_init",
  description: "Initialize a new {Method} betting session",
  inputSchema: { /* ... */ }
},
{
  name: "{method}_record",
  description: "Record a bet result and get next bet amount",
  inputSchema: { /* ... */ }
},
{
  name: "{method}_status",
  description: "Get current session status",
  inputSchema: { type: "object", properties: {} }
},
{
  name: "{method}_reset",
  description: "Reset the session to initial state",
  inputSchema: { type: "object", properties: {} }
}
```

## Testing Guidelines

### Test Structure

Use Vitest with descriptive test names:

```typescript
import { beforeEach, describe, expect, it } from "vitest";

describe("MethodNameMethod", () => {
  let method: MethodNameMethod;

  beforeEach(() => {
    method = new MethodNameMethod();
  });

  describe("initSession", () => {
    it("should initialize with provided parameters", () => {
      // test implementation
    });

    it("should throw error if baseUnit is zero or negative", () => {
      // test implementation
    });
  });
});
```

### Required Test Coverage

Every method needs tests for:
- ✅ Initialization with valid parameters
- ✅ Parameter validation errors
- ✅ Win scenarios
- ✅ Loss scenarios
- ✅ Mixed win/loss sequences
- ✅ State immutability (`getState()` returns copy)
- ✅ Session reset
- ✅ Error handling (session not active)
- ✅ Edge cases (limits, boundaries)

## Common Mistakes to Avoid

### ❌ Don't: Modify state directly
```typescript
this.state = newState;
```

### ✅ Do: Update state immutably
```typescript
this.state = {
  ...this.state,
  currentBet: newBet,
  totalProfit: this.state.totalProfit + profit,
};
```

### ❌ Don't: Skip parameter validation
```typescript
initSession(baseUnit: number) {
  this.state.baseUnit = baseUnit; // Missing validation!
}
```

### ✅ Do: Validate all inputs
```typescript
initSession(baseUnit: number) {
  if (baseUnit <= 0) {
    throw new Error("baseUnit must be positive");
  }
  this.state.baseUnit = baseUnit;
}
```

### ❌ Don't: Use single quotes
```typescript
const message = 'Hello'; // Wrong
```

### ✅ Do: Use double quotes
```typescript
const message = "Hello"; // Correct
```

### ❌ Don't: Forget semicolons
```typescript
const value = 10 // Wrong
```

### ✅ Do: Always use semicolons
```typescript
const value = 10; // Correct
```

## Type Hints

### Common Types

```typescript
// Bet result
type BetResult = "win" | "loss";

// Base state interface
interface BettingMethodState {
  currentBet: number;
  totalProfit: number;
  sessionActive: boolean;
}

// MCP tool response
{
  content: [
    {
      type: "text",
      text: JSON.stringify(data, null, 2),
    },
  ],
}
```

## Error Messages

Use clear, actionable error messages:

```typescript
// ✅ Good
throw new Error("baseUnit must be positive");
throw new Error("No active session. Please initialize a session first.");
throw new Error("Session ended: maximum bet limit reached");

// ❌ Bad
throw new Error("Invalid input");
throw new Error("Error");
throw new Error("Something went wrong");
```

## Comments

Add JSDoc comments for public methods:

```typescript
/**
 * Initialize a new betting session with the given parameters.
 * @param baseUnit The base unit amount for betting
 * @param maxBet Maximum bet amount (optional)
 */
initSession(baseUnit: number, maxBet?: number): void {
  // implementation
}
```

## Import Organization

Order imports logically:

```typescript
// 1. External dependencies
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { beforeEach, describe, expect, it } from "vitest";

// 2. Internal modules
import type { BetResult, MonteCarloState } from "./types.js";
import { MonteCarloMethod } from "./methods/montecarlo.js";
```

## File Structure

### Method Implementation File (`src/methods/{method}.ts`)
```typescript
import type { BetResult, MethodState } from "../types.js";

export class MethodMethod {
  private state: MethodState;

  constructor() { }
  initSession() { }
  recordResult() { }
  getState() { }
  reset() { }
}
```

### Test File (`src/methods/__tests__/{method}.test.ts`)
```typescript
import { beforeEach, describe, expect, it } from "vitest";
import { MethodMethod } from "../method.js";

describe("MethodMethod", () => {
  let method: MethodMethod;

  beforeEach(() => {
    method = new MethodMethod();
  });

  // Test suites...
});
```

## Build & Test Commands

Before suggesting code changes, remember:
- Code must compile: `npm run build`
- Tests must pass: `npm test`
- Linting must pass: `npm run lint`
- Formatting must pass: `npm run format:check`

## Project Context

- **Purpose**: Educational tool for betting strategy calculations
- **Not for**: Production gambling systems
- **Focus**: Type safety, testability, extensibility
- **Users**: Developers learning betting strategies or integrating with Claude Desktop

## References

- Type definitions: `src/types.ts`
- Existing implementations: `src/methods/montecarlo.ts`, `src/methods/martingale.ts`
- Architecture docs: `.docs/architecture.md`

---
> Source: [yusakuvol/betting-method-mcp](https://github.com/yusakuvol/betting-method-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
