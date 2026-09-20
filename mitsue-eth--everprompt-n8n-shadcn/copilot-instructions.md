## development-workflow

> Development workflow, coding standards, and project organization


# Development Workflow & Standards

## Project Structure

```
everprompt-n8n/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth route group
│   ├── (dashboard)/              # Dashboard route group
│   ├── api/                      # API routes
│   ├── globals.css               # Global styles
│   ├── layout.tsx                # Root layout
│   └── page.tsx                  # Home page
├── components/                   # Reusable components
│   ├── ui/                       # Base UI components
│   ├── features/                 # Feature-specific components
│   └── layout/                   # Layout components
├── lib/                          # Utility functions
│   ├── auth.ts                   # Authentication utilities
│   ├── db.ts                     # Database utilities
│   ├── utils.ts                  # General utilities
│   └── validations.ts            # Zod schemas
├── hooks/                        # Custom React hooks
├── store/                        # Zustand stores
├── types/                        # TypeScript type definitions
├── constants/                    # Application constants
├── styles/                       # Additional styles
└── public/                       # Static assets
```

## Coding Standards

### 1. **TypeScript Configuration**

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true
  }
}
```

### 2. **ESLint Configuration**

```javascript
// eslint.config.mjs
export default [
  {
    rules: {
      "@typescript-eslint/no-unused-vars": "error",
      "@typescript-eslint/no-explicit-any": "warn",
      "react-hooks/exhaustive-deps": "error",
      "prefer-const": "error",
      "no-var": "error",
    },
  },
];
```

### 3. **Prettier Configuration**

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

## Component Standards

### 1. **Component Structure**

```typescript
// components/features/PromptEditor.tsx
import React, { useState, useCallback } from "react";
import { cn } from "@/lib/utils";

interface PromptEditorProps {
  value: string;
  onChange: (value: string) => void;
  onSave?: () => void;
  className?: string;
}

export function PromptEditor({
  value,
  onChange,
  onSave,
  className,
}: PromptEditorProps) {
  const [isSaving, setIsSaving] = useState(false);

  const handleSave = useCallback(async () => {
    if (!onSave) return;

    setIsSaving(true);
    try {
      await onSave();
    } finally {
      setIsSaving(false);
    }
  }, [onSave]);

  return (
    <div className={cn("prompt-editor", className)}>
      <textarea
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="w-full h-full resize-none bg-transparent border-none outline-none"
        placeholder="Start crafting your prompt..."
      />
      {onSave && (
        <button
          onClick={handleSave}
          disabled={isSaving}
          className="save-button"
        >
          {isSaving ? "Saving..." : "Save"}
        </button>
      )}
    </div>
  );
}
```

### 2. **Hook Standards**

```typescript
// hooks/usePromptEditor.ts
import { useState, useCallback, useEffect } from "react";
import { debounce } from "lodash-es";

interface UsePromptEditorOptions {
  initialValue?: string;
  onSave?: (value: string) => Promise<void>;
  debounceMs?: number;
}

export function usePromptEditor({
  initialValue = "",
  onSave,
  debounceMs = 1000,
}: UsePromptEditorOptions = {}) {
  const [value, setValue] = useState(initialValue);
  const [isSaving, setIsSaving] = useState(false);
  const [lastSaved, setLastSaved] = useState<Date | null>(null);

  const debouncedSave = useCallback(
    debounce(async (newValue: string) => {
      if (!onSave) return;

      setIsSaving(true);
      try {
        await onSave(newValue);
        setLastSaved(new Date());
      } catch (error) {
        console.error("Failed to save prompt:", error);
      } finally {
        setIsSaving(false);
      }
    }, debounceMs),
    [onSave, debounceMs]
  );

  const handleChange = useCallback(
    (newValue: string) => {
      setValue(newValue);
      debouncedSave(newValue);
    },
    [debouncedSave]
  );

  return {
    value,
    setValue,
    handleChange,
    isSaving,
    lastSaved,
  };
}
```

### 3. **API Route Standards**

```typescript
// app/api/prompts/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { getServerSession } from "next-auth";
import { db } from "@/lib/db";

const CreatePromptSchema = z.object({
  title: z.string().min(1).max(255),
  content: z.string().min(1),
  labelIds: z.array(z.string().uuid()).optional(),
  isPublic: z.boolean().optional().default(false),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const data = CreatePromptSchema.parse(body);

    const prompt = await db.prompt.create({
      data: {
        ...data,
        createdBy: session.user.id,
        workspaceId: session.user.workspaceId,
      },
    });

    return NextResponse.json(prompt);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "Invalid input", details: error.errors },
        { status: 400 }
      );
    }

    console.error("Failed to create prompt:", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

## Database Standards

### 1. **Prisma Schema Organization**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Core entities
model Workspace {
  id        String   @id @default(cuid())
  name      String
  slug      String   @unique
  createdAt DateTime @default(now())

  // Relations
  members   WorkspaceMember[]
  prompts   Prompt[]
  labels    Label[]

  @@map("workspaces")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  avatarUrl String?
  createdAt DateTime @default(now())

  // Relations
  workspaces WorkspaceMember[]
  prompts    Prompt[]
  labels     Label[]

  @@map("users")
}

// ... other models
```

### 2. **Database Utilities**

```typescript
// lib/db.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const db = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db;
```

## Testing Standards

### 1. **Test Structure**

```typescript
// __tests__/components/PromptEditor.test.tsx
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { PromptEditor } from "@/components/features/PromptEditor";

describe("PromptEditor", () => {
  it("should render with initial value", () => {
    render(<PromptEditor value="Test prompt" onChange={jest.fn()} />);

    expect(screen.getByDisplayValue("Test prompt")).toBeInTheDocument();
  });

  it("should call onChange when content changes", async () => {
    const user = userEvent.setup();
    const mockOnChange = jest.fn();

    render(<PromptEditor value="" onChange={mockOnChange} />);

    const textarea = screen.getByRole("textbox");
    await user.type(textarea, "New content");

    expect(mockOnChange).toHaveBeenCalledWith("New content");
  });

  it("should show saving state when onSave is called", async () => {
    const mockOnSave = jest.fn().mockResolvedValue(undefined);

    render(
      <PromptEditor value="Test" onChange={jest.fn()} onSave={mockOnSave} />
    );

    const saveButton = screen.getByText("Save");
    await userEvent.click(saveButton);

    expect(screen.getByText("Saving...")).toBeInTheDocument();
    await waitFor(() => {
      expect(screen.getByText("Save")).toBeInTheDocument();
    });
  });
});
```

### 2. **API Testing**

```typescript
// __tests__/api/prompts.test.ts
import { POST } from "@/app/api/prompts/route";
import { NextRequest } from "next/server";

// Mock authentication
jest.mock("next-auth", () => ({
  getServerSession: jest.fn().mockResolvedValue({
    user: { id: "user-1", workspaceId: "workspace-1" },
  }),
}));

describe("/api/prompts", () => {
  it("should create a new prompt", async () => {
    const request = new NextRequest("http://localhost:3000/api/prompts", {
      method: "POST",
      body: JSON.stringify({
        title: "Test Prompt",
        content: "This is a test prompt",
        isPublic: false,
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data.title).toBe("Test Prompt");
    expect(data.content).toBe("This is a test prompt");
  });

  it("should return 401 for unauthenticated requests", async () => {
    // Mock unauthenticated session
    jest.mocked(getServerSession).mockResolvedValue(null);

    const request = new NextRequest("http://localhost:3000/api/prompts", {
      method: "POST",
      body: JSON.stringify({
        title: "Test Prompt",
        content: "This is a test prompt",
      }),
    });

    const response = await POST(request);
    expect(response.status).toBe(401);
  });
});
```

## Git Workflow

### 1. **Branch Strategy**

```bash
# Main branches
main                    # Production-ready code
develop                 # Integration branch

# Feature branches
feature/prompt-editor   # New features
feature/n8n-integration # Feature development

# Bug fix branches
bugfix/save-button      # Bug fixes

# Hotfix branches
hotfix/critical-bug     # Critical production fixes
```

### 2. **Commit Standards**

```bash
# Commit message format
<type>(<scope>): <description>

# Examples
feat(prompt): add autosave functionality
fix(ui): resolve label arc positioning issue
docs(api): update authentication endpoints
test(prompt): add unit tests for editor component
refactor(db): optimize prompt query performance
```

### 3. **Pull Request Template**

```markdown
## Description

Brief description of changes

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist

- [ ] Code follows project standards
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No console.log statements
- [ ] No commented code
```

## Performance Standards

### 1. **Bundle Size Limits**

```javascript
// next.config.js
module.exports = {
  experimental: {
    bundleAnalyzer: {
      enabled: process.env.ANALYZE === "true",
    },
  },
  webpack: (config) => {
    config.optimization.splitChunks = {
      chunks: "all",
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
        },
      },
    };
    return config;
  },
};
```

### 2. **Performance Monitoring**

```typescript
// lib/analytics.ts
export function trackPerformance(name: string, startTime: number) {
  const duration = performance.now() - startTime;

  if (process.env.NODE_ENV === "production") {
    // Send to analytics service
    analytics.track("performance", {
      name,
      duration,
      timestamp: Date.now(),
    });
  }
}

// Usage
const startTime = performance.now();
// ... expensive operation
trackPerformance("prompt-save", startTime);
```

## Security Standards

### 1. **Input Validation**

```typescript
// lib/validations.ts
import { z } from "zod";

export const PromptSchema = z.object({
  title: z
    .string()
    .min(1, "Title is required")
    .max(255, "Title too long")
    .regex(/^[a-zA-Z0-9\s\-_]+$/, "Invalid characters in title"),
  content: z
    .string()
    .min(1, "Content is required")
    .max(10000, "Content too long"),
  isPublic: z.boolean().default(false),
});

export type PromptInput = z.infer<typeof PromptSchema>;
```

### 2. **Rate Limiting**

```typescript
// lib/rate-limit.ts
import { NextRequest } from "next/server";

const rateLimitMap = new Map<string, { count: number; resetTime: number }>();

export function rateLimit(
  request: NextRequest,
  limit: number = 100,
  windowMs: number = 60000
): boolean {
  const ip = request.ip ?? "unknown";
  const now = Date.now();
  const windowStart = now - windowMs;

  const current = rateLimitMap.get(ip);

  if (!current || current.resetTime < windowStart) {
    rateLimitMap.set(ip, { count: 1, resetTime: now });
    return true;
  }

  if (current.count >= limit) {
    return false;
  }

  current.count++;
  return true;
}
```

## Deployment Standards

### 1. **Environment Configuration**

```bash
# .env.local
DATABASE_URL="postgresql://..."
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="http://localhost:3000"
CLERK_PUBLISHABLE_KEY="..."
CLERK_SECRET_KEY="..."
```

### 2. **Build Optimization**

```javascript
// next.config.js
module.exports = {
  output: "standalone",
  images: {
    domains: ["images.unsplash.com"],
    formats: ["image/webp", "image/avif"],
  },
  experimental: {
    optimizeCss: true,
    optimizePackageImports: ["lucide-react"],
  },
};
```

## Documentation Standards

### 1. **Component Documentation**

````typescript
/**
 * PromptEditor - Main component for editing prompts
 *
 * @param value - Current prompt content
 * @param onChange - Callback when content changes
 * @param onSave - Optional save callback
 * @param className - Additional CSS classes
 *
 * @example
 * ```tsx
 * <PromptEditor
 *   value={prompt.content}
 *   onChange={setPromptContent}
 *   onSave={handleSave}
 * />
 * ```
 */
export function PromptEditor({
  value,
  onChange,
  onSave,
  className,
}: PromptEditorProps) {
  // Component implementation
}
````

### 2. **API Documentation**

```typescript
/**
 * @route POST /api/prompts
 * @description Create a new prompt
 * @access Private
 * @param {string} title - Prompt title (required)
 * @param {string} content - Prompt content (required)
 * @param {string[]} labelIds - Array of label IDs (optional)
 * @param {boolean} isPublic - Whether prompt is public (optional)
 * @returns {Prompt} Created prompt object
 */
export async function POST(request: NextRequest) {
  // Implementation
}
```

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
