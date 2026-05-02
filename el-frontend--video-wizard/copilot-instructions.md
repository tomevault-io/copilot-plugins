## video-wizard

> You are working on the Video Wizard project, a full-stack application for AI-powered video content analysis.

# GitHub Copilot Instructions for Video Wizard

You are working on the Video Wizard project, a full-stack application for AI-powered video content analysis.

## Context Files

Before suggesting code, review these files in order:

1. `.copilot/project-instructions.md` - Project overview, tech stack, and coding standards
2. `.copilot/code-patterns.md` - Reusable templates and examples
3. `.copilot/architecture-decisions.md` - Why we made certain technical choices

## Critical Rules

### ALWAYS:
- ✅ Write ALL code, comments, and documentation in **ENGLISH ONLY**
- ✅ Use TypeScript with strict mode (never JavaScript)
- ✅ Follow the separation of concerns pattern (API routes ≠ business logic)
- ✅ Use Zod schemas for validation
- ✅ Include proper error handling with custom error classes
- ✅ Add JSDoc comments for public functions
- ✅ Use the structured logger for logging
- ✅ Return consistent response shapes from APIs

### NEVER:
- ❌ Write comments or code in Spanish (or any language other than English)
- ❌ Put business logic in API routes
- ❌ Use the `any` type
- ❌ Skip input validation
- ❌ Hardcode configuration values
- ❌ Forget error handling

## Project Structure

When suggesting new code, place it in the correct location:

```
apps/web/
├── app/
│   ├── api/            # HTTP handling ONLY
│   └── */page.tsx      # Page components
├── components/         # React components
├── lib/               # Client utilities
└── server/            # Server-side code ⭐
    ├── services/      # Business logic goes here
    ├── types/         # Zod schemas & types
    ├── config/        # Configuration
    ├── prompts/       # AI prompts
    └── lib/           # Server utilities
```

## Code Templates

### Creating a New Feature

1. **Types First** (`server/types/feature-name.ts`)
```typescript
import { z } from 'zod';

export const FeatureSchema = z.object({
  // Define schema
});

export type Feature = z.infer<typeof FeatureSchema>;
```

2. **Service** (`server/services/feature-name-service.ts`)
```typescript
export class FeatureService {
  async processData(input: string): Promise<Result> {
    // Business logic
  }
}

export const featureService = new FeatureService();
```

3. **API Route** (`app/api/feature-name/route.ts`)
```typescript
export async function POST(request: NextRequest) {
  const { input } = await request.json();
  const data = await featureService.processData(input);
  return NextResponse.json({ success: true, data });
}
```

## Type Safety

```typescript
// ✅ DO: Infer types from Zod schemas
export type MyType = z.infer<typeof MySchema>;

// ✅ DO: Use satisfies for type-safe objects
const config = { key: 'value' } satisfies ConfigType;

// ❌ DON'T: Use any
const data: any = someFunction();
```

## Error Handling

```typescript
// ✅ DO: Use custom error classes
throw new ValidationError('Invalid input');
throw new ServiceError('Operation failed', originalError);

// ✅ DO: Log with context
logger.error('Operation failed', error, { userId, operation });

// ❌ DON'T: Throw generic errors
throw new Error('Something went wrong');
```

## AI Integration

```typescript
// ✅ DO: Use structured output with Zod
const { output } = await generateText({
  model: AI_MODELS.contentAnalysis,
  output: Output.object({ schema: MySchema }),
  system: SYSTEM_PROMPT,
  prompt: buildPrompt(input),
});

// ❌ DON'T: Parse AI responses manually
const response = await openai.chat.completions.create({...});
const parsed = JSON.parse(response.choices[0].message.content);
```

## React Components

```typescript
// ✅ DO: Use 'use client' only when needed
'use client'; // Only if using hooks or browser APIs

interface MyComponentProps {
  title: string;
  onAction: (id: string) => void;
}

export function MyComponent({ title, onAction }: MyComponentProps) {
  // Component
}

// ❌ DON'T: Use 'use client' unnecessarily
// Server components are faster
```

## Environment Variables

```typescript
// ✅ DO: Use helper functions
const apiKey = requireEnv('MY_API_KEY');
const url = getEnv('MY_URL', 'https://default.com');

// ❌ DON'T: Access process.env directly
const key = process.env.MY_API_KEY;
```

## Imports

```typescript
// ✅ DO: Use path aliases
import { myService } from '@/server/services/my-service';
import type { MyType } from '@/server/types/my-types';
import { Button } from '@/components/ui/button';

// ✅ DO: Separate type imports
import type { NextRequest } from 'next/server';
import { NextResponse } from 'next/server';
```

## Testing

```typescript
// ✅ DO: Test services independently
describe('MyService', () => {
  it('should process valid input', async () => {
    const result = await myService.process('valid');
    expect(result).toBeDefined();
  });
});
```

## Language Reminder

**CRITICAL**: All suggestions, explanations, comments, and documentation MUST be in English.

Example:
```typescript
// ❌ WRONG - Spanish comment
// Valida que el usuario tenga permisos
function validateUser(user: User) {}

// ✅ CORRECT - English comment
// Validates that the user has permissions
function validateUser(user: User) {}
```

## Response Format

When suggesting code changes:

1. Explain WHAT you're changing (in English)
2. Explain WHY (reference patterns or ADRs)
3. Show the code
4. Mention any implications

Example:
```
I'll create a new service for user authentication following the service pattern 
from code-patterns.md. This separates business logic from the API route and 
makes it testable.

[code here]

This requires adding environment variables for JWT secret.
```

## References

- Full guidelines: `.copilot/project-instructions.md`
- Code templates: `.copilot/code-patterns.md`
- Technical decisions: `.copilot/architecture-decisions.md`
- Server module docs: `apps/web/server/README.md`

---

**Remember**: Quality over speed. Follow the patterns. Write in English.

---
> Source: [el-frontend/video-wizard](https://github.com/el-frontend/video-wizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
