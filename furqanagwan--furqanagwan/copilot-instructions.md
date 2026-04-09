## furqanagwan

> Type safety, validation, and comprehensive testing strategies


# Validation & Testing

## 1. Validation & Type Safety

### Server Actions & API Routes

**RULE:** Never trust client-provided data. Always validate with runtime schemas.

```typescript
// features/contact/actions.ts
"use server";

import { z } from "zod";

const contactSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  message: z.string().min(10),
});

export async function submitContactForm(formData: FormData) {
  const rawData = {
    name: formData.get("name"),
    email: formData.get("email"),
    message: formData.get("message"),
  };

  const result = contactSchema.safeParse(rawData);

  if (!result.success) {
    return { error: result.error.flatten() };
  }

  // Safe to use result.data
  await saveContact(result.data);
  return { success: true };
}
```

### TypeScript Configuration

Enforce strict mode in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## 2. Error Handling & Resilience

### Error Boundaries

Place at feature boundaries:

```typescript
// features/projects/error.tsx (Next.js)
'use client';

export default function ProjectsError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong loading projects</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

### Loading States

**Route-level:** Use `loading.tsx`  
**Component-level:** Use React Suspense

```typescript
// app/projects/page.tsx
import { Suspense } from 'react';

export default function ProjectsPage() {
  return (
    <Suspense fallback={<ProjectsSkeleton />}>
      <ProjectsList />
    </Suspense>
  );
}
```

### User Feedback

- Toast notifications for async actions
- Inline validation errors for forms
- Empty states for zero-data scenarios

## 3. Testing Strategy

### Coverage Requirements

**MUST have tests:**

- All utility functions (`utils.ts`) - Target: **>90% coverage**
- All server actions/mutations - Target: **>85% coverage**
- Critical user paths (E2E) - Cover checkout, signup, core features

**SHOULD have tests:**

- Complex UI components with logic
- Custom hooks
- API route handlers

### Unit Tests

```typescript
// features/projects/utils.test.ts
import { describe, it, expect } from "vitest";
import { sortProjectsByDate } from "./utils";

describe("sortProjectsByDate", () => {
  it("sorts projects newest first", () => {
    const projects = [
      { id: "1", date: "2023-01-01", title: "A" },
      { id: "2", date: "2024-01-01", title: "B" },
    ];

    const sorted = sortProjectsByDate(projects);
    expect(sorted[0].id).toBe("2");
  });

  it("handles empty array", () => {
    expect(sortProjectsByDate([])).toEqual([]);
  });
});
```

### Integration Tests

```typescript
// features/projects/actions.test.ts
import { expect, test } from "vitest";
import { createProject } from "./actions";

test("createProject validates input", async () => {
  const result = await createProject({ title: "" });
  expect(result.error).toBeDefined();
});

test("createProject saves valid data", async () => {
  const result = await createProject({
    title: "New Project",
    description: "Test description",
  });
  expect(result.success).toBe(true);
});
```

### E2E Tests

```typescript
// e2e/projects.spec.ts
import { test, expect } from "@playwright/test";

test("can view and filter projects", async ({ page }) => {
  await page.goto("/projects");

  // Check projects load
  await expect(page.locator("article")).toHaveCount(5);

  // Filter by category
  await page.click('button:has-text("Web")');
  await expect(page.locator("article")).toHaveCount(3);
});
```

**Recommended Tools:**

- **Unit/Integration:** Vitest, Jest
- **E2E:** Playwright, Cypress
- **Visual:** Chromatic, Percy
- **Mocking:** MSW (Mock Service Worker)

## 4. Performance Considerations

### Bundle Size

- Use dynamic imports for large dependencies
- Code-split features automatically via routing
- Monitor bundle size in CI (`@next/bundle-analyzer`)

```typescript
// Lazy load heavy component
import dynamic from 'next/dynamic';

const ChartComponent = dynamic(() => import('./Chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // Skip SSR if not needed
});
```

### Optimization Checklist

- [ ] Images use `next/image` or framework equivalent
- [ ] Fonts are self-hosted and optimized
- [ ] Third-party scripts use appropriate loading strategies
- [ ] Large lists use virtualization
- [ ] API responses include caching headers

## 5. AI Agent Decision Framework

Before any code modification, AI agents MUST run this checklist:

### 1. Locality Check

**Question:** Is this logic feature-specific or truly shared?

- **Feature-specific →** Place in `src/features/[feature]/`
- **Used by 3+ features →** Consider `src/shared/`
- **Unsure →** Start in feature, refactor later

### 2. Component Boundary Check

**Question:** Does this need client-side interactivity?

- **No interactivity →** Server Component (default)
- **Needs hooks/events →** Client Component (`'use client'`)
- **Mixed →** Compose Server + Client components

### 3. Logic Extraction Check

**Question:** Is there transformation logic in a route or component?

- **Yes →** Extract to `utils.ts` with unit tests
- **Complex →** Consider separate service/repository pattern

### 4. Type Safety Check

**Question:** Does this handle user input or external data?

- **Yes →** Add runtime validation (zod, valibot)
- **API call →** Validate response shape
- **Form →** Validate before submission

### 5. Testing Check

**Question:** Is this business-critical or complex?

- **Yes →** Write tests FIRST (TDD)
- **Utility function →** Must have unit tests
- **User flow →** Add E2E test

### 6. Verification Steps

After changes:

1. Run `npm run build`
2. Run `npm run lint`
3. Run `npm run type-check`
4. Run `npm test`
5. Manual browser smoke test
6. Check for console errors/warnings

**Next:** See `03-migration-workflow-part1.md` for step-by-step migration process.

### Testing with Content Collections

If using `content-collections` or similar content management:

```typescript
// vitest.config.ts - Add alias for module resolution
alias: {
  "content-collections": path.resolve(__dirname, "./.content-collections/generated"),
}
```

This enables proper mocking in feature tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/furqanagwan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/furqanagwan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
