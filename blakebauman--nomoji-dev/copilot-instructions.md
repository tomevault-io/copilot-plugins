## nomoji-dev

> Code style, formatting, and consistency guidelines


# Code Style and Formatting

This project uses Biome for formatting and linting. Follow these style guidelines for consistency.

## Formatter Configuration

### Biome Settings
Configuration in `biome.json`:
- Indent: 2 spaces
- Line width: 80 characters
- Line ending: LF
- Semicolons: Always
- Quotes: Double quotes
- Trailing commas: Always
- Arrow parentheses: Always
- Bracket spacing: True

### Running Formatter
```bash
# Format all files
npm run format

# Check formatting without changes
npm run format:check

# Format and lint
npm run check
```

## General Style Rules

### Indentation
Use 2 spaces for indentation:
```typescript
function example() {
  if (condition) {
    doSomething();
  }
}
```

### Line Length
Limit lines to 80 characters:
- Break long lines logically
- Use line breaks for readability
- Consider extracting to variables

```typescript
// Good
const config = await getOrCreateUserConfig(
  env,
  userId
);

// Avoid
const config = await getOrCreateUserConfig(env, userId, options, metadata);
```

### Semicolons
Always use semicolons:
```typescript
const value = getValue();
return value;
```

### Quotes
Use double quotes for strings:
```typescript
const message = "Hello world";
const key = "user:config";
```

Exception: Use backticks for template literals:
```typescript
const message = `Hello ${name}`;
```

### Trailing Commas
Always use trailing commas in multiline:
```typescript
const config = {
  enabled: true,
  version: "1.0.0",    // Trailing comma
};

const array = [
  "item1",
  "item2",             // Trailing comma
];
```

## TypeScript Style

### Type Annotations
Include return types for functions:
```typescript
function getConfig(): NomojiConfig {
  return DEFAULT_CONFIG;
}

async function fetchData(): Promise<Data> {
  return await fetch();
}
```

### Type vs Interface
- Use `interface` for objects that might be extended
- Use `type` for unions, intersections, primitives

```typescript
interface NomojiConfig {
  version: string;
  enabled: boolean;
}

type Status = "active" | "inactive" | "pending";
```

### Const Assertions
Use `as const` for literal types:
```typescript
const PRESETS = {
  strict: { /* ... */ },
  moderate: { /* ... */ },
} as const;
```

## Function Style

### Arrow Functions
Use arrow functions for callbacks and short functions:
```typescript
const filter = items.filter((item) => item.enabled);
const map = items.map((item) => item.id);
```

Always use parentheses around parameters:
```typescript
// Good
const fn = (x) => x * 2;

// Avoid
const fn = x => x * 2;
```

### Function Declarations
Use function declarations for named functions:
```typescript
function processConfig(config: NomojiConfig): void {
  // Implementation
}
```

### Async Functions
Declare async functions explicitly:
```typescript
async function fetchConfig(): Promise<Config> {
  const response = await fetch(url);
  return await response.json();
}
```

## Object and Array Style

### Object Properties
Use shorthand when possible:
```typescript
const userId = "123";
const enabled = true;

// Good
const obj = { userId, enabled };

// Avoid
const obj = { userId: userId, enabled: enabled };
```

### Object Destructuring
Use destructuring for multiple properties:
```typescript
const { userId, enabled, version } = config;
```

### Array Methods
Prefer array methods over loops:
```typescript
// Good
const enabled = configs.filter((c) => c.enabled);
const ids = configs.map((c) => c.id);

// Avoid
const enabled = [];
for (let i = 0; i < configs.length; i++) {
  if (configs[i].enabled) {
    enabled.push(configs[i]);
  }
}
```

### Spread Operator
Use spread for copying and merging:
```typescript
const updated = { ...config, enabled: false };
const combined = [...array1, ...array2];
```

## Conditional Style

### If Statements
Use braces for all blocks:
```typescript
// Good
if (condition) {
  doSomething();
}

// Avoid
if (condition) doSomething();
```

### Ternary Operator
Use for simple conditions:
```typescript
const value = condition ? "yes" : "no";
```

For complex logic, use if-else:
```typescript
// Avoid
const result = condition1 ? value1 
  : condition2 ? value2 
  : condition3 ? value3 
  : defaultValue;

// Good
let result;
if (condition1) {
  result = value1;
} else if (condition2) {
  result = value2;
} else if (condition3) {
  result = value3;
} else {
  result = defaultValue;
}
```

### Optional Chaining
Use for potentially undefined properties:
```typescript
const level = config?.metadata?.level;
```

### Nullish Coalescing
Use ?? for default values:
```typescript
const timeout = config.timeout ?? 5000;
```

## Comments Style

### Single-Line Comments
Use `//` for single-line comments:
```typescript
// This is a single-line comment
const value = getValue();
```

### Multi-Line Comments
Use `/* */` for multi-line comments:
```typescript
/*
 * This is a multi-line comment
 * that spans multiple lines
 */
```

### JSDoc Comments
Use JSDoc for function documentation:
```typescript
/**
 * Retrieves user configuration
 * 
 * @param userId - User identifier
 * @returns User configuration
 */
function getConfig(userId: string): Config {
  // Implementation
}
```

### Comment Placement
Place comments above the code they describe:
```typescript
// Calculate the total duration
const duration = endTime - startTime;
```

### Comment Content
- Explain why, not what
- Keep comments up to date
- Remove commented-out code
- No emojis in comments

## Import/Export Style

### Import Order
Order imports logically:
```typescript
// 1. External packages
import { Hono } from "hono";
import { corsMiddleware } from "hono/cors";

// 2. Types
import type { Env, Config } from "./types";

// 3. Internal modules
import { generateRules } from "./rules/generator";
import { getConfig } from "./utils/storage";

// 4. Relative imports
import { HomePage } from "../views/home";
```

### Named Exports
Prefer named exports:
```typescript
export function generateRules() { }
export function validateConfig() { }
export const DEFAULT_CONFIG = { };
```

### Default Exports
Use sparingly (main entry, components):
```typescript
// index.ts
export default {
  fetch: app.fetch,
};

// HomePage.tsx
export function HomePage() { }
```

## Naming Style

### Variables
Use camelCase:
```typescript
const userId = "123";
const isEnabled = true;
const configData = getData();
```

### Constants
Use UPPER_CASE:
```typescript
const DEFAULT_TIMEOUT = 5000;
const MAX_RETRIES = 3;
const API_VERSION = "1.0.0";
```

### Functions
Use camelCase, start with verb:
```typescript
function getConfig() { }
function validateInput() { }
function createUser() { }
```

### Classes/Interfaces
Use PascalCase:
```typescript
interface NomojiConfig { }
class ConfigManager { }
type UserPreferences = { };
```

### Boolean Names
Use is/has/can prefix:
```typescript
const isEnabled = true;
const hasError = false;
const canEdit = checkPermissions();
```

## File Organization Style

### File Structure
Organize file content in order:
1. Imports
2. Type definitions
3. Constants
4. Helper functions
5. Main exports
6. Default export (if any)

```typescript
// 1. Imports
import { Hono } from "hono";

// 2. Types
type Config = { };

// 3. Constants
const DEFAULT_VALUE = 10;

// 4. Helpers
function helper() { }

// 5. Main exports
export function main() { }

// 6. Default export
export default main;
```

### Line Spacing
Use blank lines to separate sections:
```typescript
import { Hono } from "hono";
                              // Blank line after imports
const app = new Hono();
                              // Blank line between sections
app.get("/", (c) => {
  return c.text("Hello");
});
                              // Blank line before export
export default app;
```

## Special Cases

### JSX/TSX
Follow React conventions:
```tsx
export function HomePage() {
  return (
    <html lang="en">
      <head>
        <title>nomoji.dev</title>
      </head>
      <body>
        <h1>Hello World</h1>
      </body>
    </html>
  );
}
```

### Chaining
Break method chains logically:
```typescript
const result = data
  .filter((item) => item.enabled)
  .map((item) => item.id)
  .sort();
```

### Template Literals
Use multi-line for readability:
```typescript
const html = `
  <div>
    <h1>${title}</h1>
    <p>${description}</p>
  </div>
`;
```

## Linting

### Running Linter
```bash
# Lint and auto-fix
npm run lint

# Check without fixing
npm run lint:check
```

### Disable Rules
Avoid disabling rules. If necessary, document why:
```typescript
// biome-ignore lint/suspicious/noExplicitAny: Legacy code, will be typed later
function legacy(data: any) { }
```

## Pre-Commit Checks

### Format and Lint
Run before committing:
```bash
npm run check
```

### CI Checks
Same checks run in CI:
```bash
npm run check:ci
```

## Editor Configuration

### VS Code
Install Biome extension for:
- Format on save
- Inline linting
- Auto-fix on save

### Settings
Recommended VS Code settings:
```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit"
  }
}
```

---

**Remember**: Consistency is more important than personal preference. Follow the established patterns and use automated formatting.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blakebauman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
