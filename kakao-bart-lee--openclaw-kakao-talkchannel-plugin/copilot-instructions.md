## openclaw-kakao-talkchannel-plugin

> Guide for AI coding agents working in this repository.

# AGENTS.md

Guide for AI coding agents working in this repository.

## Project Overview

OpenClaw Kakao TalkChannel Plugin - connects KakaoTalk channels to OpenClaw platform.

- **Runtime**: Node.js 22.12+, ESM modules (`"type": "module"`)
- **Language**: TypeScript 5.3+ with strict mode
- **Package Manager**: pnpm

## Commands

### Development

```bash
pnpm dev          # Watch mode (tsc --watch)
pnpm build        # Build to dist/
pnpm typecheck    # Type check without emitting
```

### Testing

```bash
pnpm test                 # Watch mode
pnpm test:run             # Single run (CI)
pnpm test:coverage        # With coverage report

# Run single test file
pnpm vitest run tests/unit/channel.test.ts

# Run tests matching pattern
pnpm vitest run -t "should send reply"

# Run specific test directory
pnpm vitest run tests/unit/relay/
```

### Code Quality

```bash
pnpm lint         # ESLint (src/ and tests/)
pnpm typecheck    # TypeScript strict check
```

## Project Structure

```
src/
├── adapters/     # OpenClaw adapter implementations
├── config/       # Zod schemas for configuration
├── kakao/        # Kakao API types and utilities
├── relay/        # SSE relay client
├── types.ts      # Type definitions
├── channel.ts    # Main plugin export
└── runtime.ts    # Runtime singleton
tests/
├── unit/         # Mirrors src/ structure
├── integration/  # E2E tests
├── fixtures/     # Test data
└── setup.ts      # Vitest setup
```

## Code Style

### TypeScript

- **Strict mode**: All strict checks enabled
- **Module system**: NodeNext (ESM)
- **Target**: ES2022

```typescript
// Use explicit type imports
import type { KakaoSkillPayload } from "../types.js";

// Always include .js extension for relative imports
import { sendReply } from "../relay/client.js";

// Prefer interfaces over types for objects
export interface GatewayContext {
  account: ResolvedKakaoTalkChannel;
  accountId: string;
}

// Use type assertions sparingly, never `as any`
const data = result as SendReplyResponse;

// Prefer unknown over any, validate with type guards
function isObject(value: unknown): value is Record<string, unknown> {
  return value !== null && typeof value === "object";
}
```

### Naming Conventions

```typescript
// Files: kebab-case
src/relay/client.ts
tests/unit/kakao/response.test.ts

// Interfaces: PascalCase, descriptive
interface KakaoSkillPayload { ... }
interface OutboundResult { ... }

// Functions: camelCase, verb-first
function validateAccountConfig() { ... }
function buildMessageContext() { ... }

// Constants: SCREAMING_SNAKE_CASE
const DEFAULT_TIMEOUT_MS = 10000;
const DEFAULT_RELAY_URL = "https://k.tess.dev/";

// Types: PascalCase, use union types liberally
type KakaoDmPolicy = "pairing" | "allowlist" | "open" | "disabled";
```

### Comments and Documentation

```typescript
/**
 * JSDoc for exported functions
 *
 * Single line for simple types
 */
export function validateAccountConfig(input: unknown): ValidationResult<KakaoAccountConfig> {

// Inline comments for complex logic
// Always explain WHY, not WHAT
const sessionKey = `agent:main:kakao-talkchannel:dm:${normalized.userId}`;
```

### Error Handling

```typescript
// Always use instanceof for error type checking
try {
  await sendReply(config, messageId, response);
} catch (err) {
  const errMsg = err instanceof Error ? err.message : String(err);
  log?.error(`Reply failed: ${errMsg}`);
}

// Use Result pattern for validation
type ValidationResult<T> =
  | { ok: true; data: T }
  | { ok: false; errors: string[] };
```

### Zod Schemas

```typescript
// Define schemas with validation messages (Korean OK)
export const KakaoAccountConfigSchema = z.object({
  enabled: z.boolean().default(true),
  reconnectDelayMs: z.number()
    .min(500, "reconnectDelayMs는 최소 500ms 이상이어야 합니다")
    .max(10000, "reconnectDelayMs는 최대 10000ms 이하여야 합니다")
    .default(1000),
});

// Infer types from schemas
export type KakaoAccountConfig = z.infer<typeof KakaoAccountConfigSchema>;
```

## Testing Patterns

### Structure

```typescript
import { describe, it, expect, beforeEach, vi } from "vitest";

describe("ComponentName", () => {
  describe("methodName", () => {
    it("should do expected behavior", () => {
      // Arrange, Act, Assert
    });
  });
});
```

### Mocking

```typescript
// Mock globals
global.fetch = vi.fn();

// Create mock utilities in tests/setup.ts
export const createMockRuntime = () => ({
  logger: { debug: vi.fn(), info: vi.fn(), warn: vi.fn(), error: vi.fn() },
  config: {},
});

// Reset mocks
beforeEach(() => {
  vi.clearAllMocks();
});
```

### Coverage Requirements

- Lines: 80%
- Functions: 80%
- Branches: 70%
- Statements: 80%

## Commit Convention

**CRITICAL**: Uses release-please with Conventional Commits.

### Format

```
<type>: <message in Korean or English>
```

### Types

| Type | Description | Version Bump |
|------|-------------|--------------|
| `feat:` | New feature | Minor |
| `fix:` | Bug fix | Patch |
| `feat!:` | Breaking change | Major |
| `docs:` | Documentation | None |
| `refactor:` | Code refactor | None |
| `test:` | Tests | None |
| `chore:` | Build/config | None |

### Examples

```bash
# Correct
feat: 새로운 기능 추가
fix: 타임아웃 오류 수정

# WRONG - No emojis!
✨ feat: 기능 추가    # release-please fails
```

## Key Dependencies

- **zod**: Schema validation
- **openclaw**: Peer dependency (plugin SDK)
- **vitest**: Testing framework

## Import Order

```typescript
// 1. Node built-ins (rare in this project)
// 2. External packages
import { z } from "zod";

// 3. Internal absolute (type imports first)
import type { KakaoSkillPayload } from "../types.js";

// 4. Internal relative
import { sendReply } from "./client.js";
```

## Anti-patterns to Avoid

```typescript
// NEVER use type assertions to silence errors
const data = result as any;  // BAD
// @ts-ignore               // BAD
// @ts-expect-error         // BAD

// NEVER empty catch blocks
catch(e) {}                  // BAD

// NEVER forget .js extension
import { foo } from "./bar"; // BAD - must be "./bar.js"
```

---
> Source: [kakao-bart-lee/openclaw-kakao-talkchannel-plugin](https://github.com/kakao-bart-lee/openclaw-kakao-talkchannel-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
