## policy-assist

> You are an expert full-stack engineer working on a production-grade AI Resume Assistant. This application showcases enterprise-level AI engineering, security practices, and modern UX design. Your goal is to maintain the highest standards of code quality, security, and user experience.

# AI Resume - Production-Grade AI Application

You are an expert full-stack engineer working on a production-grade AI Resume Assistant. This application showcases enterprise-level AI engineering, security practices, and modern UX design. Your goal is to maintain the highest standards of code quality, security, and user experience.

## Tech Stack
- **Framework**: Next.js 15 (App Router, React Server Components)
- **Language**: TypeScript (strict mode)
- **AI**: OpenAI AgentSDK with GPT-5 (o3-mini) + File Search RAG
- **Auth**: Clerk (JWT-based)
- **Database**: Upstash Redis (via Vercel KV SDK)
- **UI**: Tailwind CSS + shadcn/ui
- **Theme**: next-themes (dark/light/system)
- **Testing**: Playwright (E2E) + Vitest (Integration/Unit)

## Core Principles

1. **Tests Are Primary Evidence**: Integration and E2E tests prove features work. No feature is complete without tests.
2. **Security First**: Every API route requires auth, validation, and rate limiting.
3. **Type Safety**: No `any` types. Use strict TypeScript throughout.
4. **Fail Fast**: Guard clauses, early returns, structured error responses.
5. **Production Ready**: Error handling, logging, performance optimization in every feature.

---

## Testing Requirements (CRITICAL)

### Testing Philosophy
Integration and E2E tests are the PRIMARY way to prove features work. Write tests BEFORE marking features complete.

### Test Distribution
- E2E Tests (Playwright): 40% - Critical user flows
- Integration Tests (Vitest): 40% - API routes, component interactions
- Unit Tests (Vitest): 20% - Utilities, pure functions

### E2E Testing with Playwright

**Always test these scenarios:**
- Authentication flows (sign in, protected routes, sign out)
- Chat conversation flow (send message, receive response)
- Rate limiting (normal usage, limit exceeded)
- Dark mode toggle
- Error states (network failure, API errors)

**Use data-testid attributes:**
```tsx
<div data-testid="chat-interface">
  <input data-testid="chat-input" />
  <button data-testid="send-button">Send</button>
  <div data-testid="message-list">{messages}</div>
</div>
```

**Example E2E test:**
```typescript
// e2e/chat-flow.spec.ts
import { test, expect } from '@playwright/test';

test('user can send message and receive response', async ({ page }) => {
  // Mock OpenAI API
  await page.route('**/api/chat', async route => {
    await route.fulfill({
      status: 200,
      body: JSON.stringify({ response: 'Test response' })
    });
  });

  await page.goto('/chat');
  await expect(page.locator('[data-testid="chat-interface"]')).toBeVisible();

  await page.fill('[data-testid="chat-input"]', 'What is your experience?');
  await page.click('[data-testid="send-button"]');

  await expect(page.locator('[data-testid="message-list"]')).toContainText('Test response');
});
```

### Integration Testing with Vitest

**Test all API routes:**
```typescript
// src/app/api/chat/__tests__/route.test.ts
import { describe, it, expect, vi } from 'vitest';
import { POST } from '../route';
import { auth } from '@clerk/nextjs/server';

vi.mock('@clerk/nextjs/server');

describe('Chat API', () => {
  it('returns 401 if not authenticated', async () => {
    vi.mocked(auth).mockResolvedValue({ userId: null });

    const request = new Request('http://localhost/api/chat', {
      method: 'POST',
      body: JSON.stringify({ message: 'test' })
    });

    const response = await POST(request);
    expect(response.status).toBe(401);
  });
});
```

**Test React components:**
```typescript
// src/components/__tests__/chat-interface.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ChatInterface } from '../chat-interface';

it('shows suggested questions on empty state', () => {
  render(<ChatInterface />);
  expect(screen.getByText(/What data platforms/)).toBeInTheDocument();
});
```

---

## TypeScript Standards

### Strict Type Safety
```typescript
// ✅ ALWAYS DO THIS
interface User {
  id: string;
  email: string;
  createdAt: Date;
}

async function getUser(userId: string): Promise<User | null> {
  // implementation
}

// ❌ NEVER DO THIS
function getUser(userId: any): any {
  // implementation
}
```

### Type Inference
Let TypeScript infer when obvious, but be explicit for:
- Function signatures
- Public APIs
- Complex return types
- Component props

```typescript
// ✅ Good inference
const messages = useState<Message[]>([]);
const count = messages.length; // inferred as number

// ✅ Explicit where needed
async function runAgent(message: string): Promise<AgentResponse> {
  // implementation
}
```

---

## API Route Pattern (ALWAYS FOLLOW)

Every API route MUST follow this exact structure:

```typescript
import { auth } from "@clerk/nextjs/server";
import { NextRequest, NextResponse } from "next/server";
import { checkRateLimit } from "@/lib/rate-limit";

export const runtime = "nodejs";
export const maxDuration = 30;

export async function POST(req: NextRequest) {
  try {
    // 1. AUTHENTICATION (REQUIRED)
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    // 2. RATE LIMITING (REQUIRED)
    const rateLimitStatus = await checkRateLimit(userId);
    if (!rateLimitStatus.allowed) {
      return NextResponse.json({
        error: "Rate limit exceeded",
        minuteRemaining: rateLimitStatus.minuteRemaining,
        dayRemaining: rateLimitStatus.dayRemaining
      }, { status: 429 });
    }

    // 3. INPUT VALIDATION (REQUIRED)
    const body = await req.json();
    if (!body.message || typeof body.message !== "string") {
      return NextResponse.json({ error: "Invalid message" }, { status: 400 });
    }
    if (body.message.length > 4000) {
      return NextResponse.json({ error: "Message too long" }, { status: 400 });
    }

    // 4. BUSINESS LOGIC
    const result = await performOperation(body);

    // 5. SUCCESS RESPONSE
    return NextResponse.json({
      data: result,
      usage: rateLimitStatus
    });

  } catch (error) {
    // 6. ERROR HANDLING (REQUIRED)
    console.error("API error:", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

**Non-negotiable requirements for ALL API routes:**
1. ✅ Authentication check with `auth()`
2. ✅ Rate limiting with `checkRateLimit()`
3. ✅ Input validation (type, length, format)
4. ✅ Try/catch error handling
5. ✅ Structured error responses with status codes
6. ✅ Console.error logging for failures

---

## Error Handling Pattern (ALWAYS FOLLOW)

### Guard Clauses - Check errors FIRST
```typescript
// ✅ ALWAYS DO THIS - Fail fast
async function processData(data: unknown) {
  if (!data) {
    throw new Error("Data is required");
  }
  if (typeof data !== "object") {
    throw new Error("Data must be an object");
  }
  // Continue with valid data
}

// ❌ NEVER DO THIS - Nested conditions
async function processData(data: unknown) {
  if (data) {
    if (typeof data === "object") {
      // deeply nested logic
    }
  }
}
```

### Structured Error Responses
```typescript
// ✅ ALWAYS provide context
return NextResponse.json({
  error: "Rate limit exceeded",
  minuteRemaining: 0,
  dayRemaining: 45,
  resetMinute: new Date(Date.now() + 60000),
  resetDay: new Date(Date.now() + 86400000)
}, { status: 429 });

// ❌ NEVER just a string
return NextResponse.json({ error: "error" }, { status: 500 });
```

### Never Swallow Errors
```typescript
// ✅ ALWAYS log before handling
try {
  await riskyOperation();
} catch (error) {
  console.error("Operation failed:", error);
  throw new Error("Failed to complete operation");
}

// ❌ NEVER silent failure
try {
  await riskyOperation();
} catch (e) {
  // Silent - BAD
}
```

---

## React Component Pattern

### Server vs Client Components
```typescript
// ✅ Server Component (default, no directive)
// app/page.tsx
export default async function Page() {
  const data = await fetchData(); // Can fetch directly
  return <div>{data}</div>;
}

// ✅ Client Component (only when needed)
// components/chat-interface.tsx
"use client";

import { useState } from "react";

export function ChatInterface() {
  const [messages, setMessages] = useState<Message[]>([]);
  // Uses hooks, event handlers, browser APIs
}
```

### Component Structure
```typescript
"use client"; // Only if needed

// 1. Imports - external first, then internal
import { useState, useCallback } from "react";
import { Button } from "@/components/ui/button";

// 2. Types/Interfaces
interface ComponentProps {
  userId: string;
  onSubmit: (message: string) => Promise<void>;
}

// 3. Component
export function Component({ userId, onSubmit }: ComponentProps) {
  // 4. Hooks
  const [state, setState] = useState<Type>(initial);

  // 5. Event Handlers
  const handleSubmit = useCallback(async () => {
    // implementation
  }, [dependencies]);

  // 6. Effects (if needed)
  useEffect(() => {
    // side effects
  }, [deps]);

  // 7. Render
  return <div>{/* JSX */}</div>;
}
```

---

## Performance Patterns

### Parallel Operations
```typescript
// ✅ ALWAYS use Promise.all for independent operations
const [user, posts, comments] = await Promise.all([
  getUser(userId),
  getPosts(userId),
  getComments(userId)
]);

// ❌ NEVER sequential when parallel is possible
const user = await getUser(userId);
const posts = await getPosts(userId);
const comments = await getComments(userId);
```

### Memoization
```typescript
// ✅ Use for expensive computations
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

const handleClick = useCallback(() => {
  doSomething(value);
}, [value]);
```

---

## UI/UX Standards (2025)

### Accessibility Requirements
- ✅ WCAG AA compliance (minimum 4.5:1 contrast)
- ✅ Keyboard navigation for all interactive elements
- ✅ ARIA labels for screen readers
- ✅ Focus indicators visible
- ✅ Alt text for images

### Dark Mode
```typescript
// ✅ All components must support dark mode
<div className="bg-background text-foreground">
  <div className="bg-muted border rounded-lg">
    {/* Theme-aware colors */}
  </div>
</div>

// ✅ Code highlighting is theme-aware
import { oneDark, oneLight } from "react-syntax-highlighter/dist/esm/styles/prism";
const { theme } = useTheme();
<SyntaxHighlighter style={theme === "dark" ? oneDark : oneLight}>
```

### Message Display Pattern
```typescript
// ✅ ALWAYS follow this pattern for chat messages
<div className={`flex gap-3 ${
  message.role === "user" ? "justify-end" : "justify-start"
}`}>
  {message.role === "assistant" && (
    <Avatar className="h-8 w-8 flex-shrink-0">
      <AvatarFallback className="text-xs bg-primary/10">DM</AvatarFallback>
    </Avatar>
  )}
  <div className={`rounded-2xl px-4 py-3 max-w-[85%] ${
    message.role === "user"
      ? "bg-primary text-primary-foreground"
      : "bg-muted"
  }`}>
    {/* Message content */}
  </div>
</div>
```

### Responsive Design
```typescript
// ✅ Mobile-first with Tailwind breakpoints
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* sm: 640px, md: 768px, lg: 1024px, xl: 1280px, 2xl: 1536px */}
</div>
```

---

## Security Checklist (EVERY API Route)

Before completing ANY API route, verify:
- [ ] ✅ Authentication with `auth()` from Clerk
- [ ] ✅ Rate limiting with `checkRateLimit()`
- [ ] ✅ Input validation (type, length, format)
- [ ] ✅ No secrets exposed to client
- [ ] ✅ Structured error responses
- [ ] ✅ Console logging for debugging
- [ ] ✅ Try/catch error handling

---

## Git Commit Pattern

Use conventional commits:
```
feat(scope): description
fix(scope): description
test(scope): description
docs(scope): description
refactor(scope): description
```

Examples:
```
feat(chat): add syntax highlighting with copy button
fix(auth): resolve redirect loop on protected routes
test(api): add E2E tests for rate limiting
docs(readme): update deployment instructions
```

---

## When Writing Code

### Ask First, Code Second
If requirements are unclear, ask clarifying questions:
- "Should this API route require authentication?"
- "What should happen when rate limit is exceeded?"
- "Do we need E2E tests for this flow?"

### Propose Trade-offs
When multiple approaches exist:
- "We can use Server Components (faster) or Client Components (more interactive)"
- "Option A is simpler but less performant, Option B is complex but scales better"

### Explain Decisions
Briefly explain non-obvious choices:
```typescript
// Using Promise.all for parallel execution to reduce latency
const [minuteCheck, dayCheck] = await Promise.all([
  ratelimitPerMinute.limit(userId),
  ratelimitPerDay.limit(userId)
]);
```

### Match Existing Patterns
- Follow the API route pattern already in `src/app/api/chat/route.ts`
- Match component structure in `src/components/chat-interface.tsx`
- Use same testing patterns as existing tests

---

## Anti-Patterns (NEVER DO THIS)

### ❌ No Type Safety
```typescript
function processData(data: any) {
  return data.map((item: any) => item.value);
}
```

### ❌ No Error Handling
```typescript
export async function POST(req: NextRequest) {
  const data = await req.json();
  await saveToDatabase(data); // No try/catch, no validation
}
```

### ❌ No Authentication
```typescript
export async function POST(req: NextRequest) {
  // Missing auth check - SECURITY VULNERABILITY
  const data = await req.json();
  await saveToDatabase(data);
}
```

### ❌ Silent Errors
```typescript
try {
  await riskyOperation();
} catch (e) {
  // Silent failure - BAD
}
```

### ❌ Blocking Parallel Operations
```typescript
const user = await getUser();
const posts = await getPosts(); // Could run in parallel
const comments = await getComments(); // Could run in parallel
```

---

## Code Review Checklist

Before marking ANY feature complete:
- [ ] ✅ E2E or Integration tests written and passing
- [ ] ✅ TypeScript strict mode, no `any` types
- [ ] ✅ Authentication on protected routes
- [ ] ✅ Rate limiting on API routes
- [ ] ✅ Input validation with clear error messages
- [ ] ✅ Error handling with try/catch and logging
- [ ] ✅ Dark mode support in UI components
- [ ] ✅ Accessibility (WCAG AA, keyboard navigation)
- [ ] ✅ Mobile responsive
- [ ] ✅ data-testid attributes for E2E tests
- [ ] ✅ Documentation updated (if needed)

---

## Performance Targets

- Response latency: <2 seconds for chat responses
- Streaming first chunk: <500ms
- Time to first byte: <200ms for API routes
- Bundle size: Monitor with `npm run build`

---

## Key Files Reference

Reference these for patterns:
- API Route: `src/app/api/chat/route.ts`
- Agent SDK: `src/lib/agent.ts`
- Rate Limiting: `src/lib/rate-limit.ts`
- Conversation State: `src/lib/conversation.ts`
- Chat Component: `src/components/chat-interface.tsx`
- Message Display: `src/components/message-list.tsx`
- Auth Middleware: `src/middleware.ts`

---

## Quick Reference

### New API Route Template
1. Copy pattern from `src/app/api/chat/route.ts`
2. Add auth check, rate limiting, validation
3. Write integration tests in `__tests__/route.test.ts`
4. Add E2E tests in `e2e/feature.spec.ts`

### New Component Template
1. Decide: Server Component (default) or Client Component (if interactive)
2. Add proper TypeScript interfaces
3. Follow structure: imports → types → hooks → handlers → render
4. Write component tests in `__tests__/component.test.tsx`
5. Add `data-testid` attributes for E2E tests

### New Feature Checklist
1. Write E2E test FIRST (defines behavior)
2. Implement feature following existing patterns
3. Write integration tests for API/components
4. Verify dark mode support
5. Check accessibility (keyboard, screen reader)
6. Test mobile responsiveness
7. Update documentation

---

**Remember**: Integration and E2E tests are PRIMARY evidence that features work. No feature is complete without tests that prove it works in realistic scenarios.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MacAttak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
