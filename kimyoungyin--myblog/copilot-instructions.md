## server-actions

> - **ALWAYS** use Server Actions for data mutations (create, update, delete)

# Server Actions & Data Processing (MANDATORY)

## 🚀 **Server Actions (MUST USE)**

- **ALWAYS** use Server Actions for data mutations (create, update, delete)
- **ALWAYS** implement proper error handling and validation
- **ALWAYS** use `revalidatePath` for cache invalidation
- **NEVER** create API routes when Server Actions can handle the logic

## 📝 **Server Action Patterns (CRITICAL)**

Based on [src/lib/actions.ts](mdc:src/lib/actions.ts):

- **ALWAYS** use Zod schemas for input validation
- **ALWAYS** implement proper error handling with try-catch
- **ALWAYS** use `revalidatePath` to update related pages
- **ALWAYS** redirect after successful operations

```typescript
// ✅ CORRECT - Proper Server Action implementation
export async function createPostAction(formData: FormData) {
    try {
        // Extract and validate form data
        const rawData = {
            title: formData.get('title') as string,
            content: formData.get('content') as string,
            hashtags:
                (formData.get('hashtags') as string)
                    ?.split(',')
                    .map((tag: string) => tag.trim())
                    .filter((tag: string) => tag.length > 0) || [],
        };

        // Validate with Zod schema
        const validationResult = CreatePostSchema.safeParse(rawData);
        if (!validationResult.success) {
            const errorMessage = validationResult.error.issues
                .map((issue) => `${issue.path.join('.')}: ${issue.message}`)
                .join(', ');
            throw new Error(`데이터 검증 실패: ${errorMessage}`);
        }

        // Process data and redirect
        const post = await createPost(validationResult.data);
        revalidatePath('/admin/posts');
        revalidatePath('/posts');
        redirect('/admin/posts');
    } catch (error) {
        throw error;
    }
}

// ❌ WRONG - Poor Server Action implementation
export async function createPostAction(formData: FormData) {
    const title = formData.get('title');
    const content = formData.get('content');

    // No validation, no error handling, no cache invalidation
    await supabase.from('posts').insert({ title, content });
}
```

## 🚫 **FORBIDDEN Server Action Practices**

- **NEVER** skip input validation
- **NEVER** omit error handling
- **NEVER** forget to invalidate related caches
- **NEVER** expose sensitive information in error messages
- **NEVER** use client-side validation only

## ✅ **REQUIRED Server Action Practices**

- **ALWAYS** validate all inputs with Zod schemas
- **ALWAYS** implement comprehensive error handling
- **ALWAYS** use proper TypeScript types
- **ALWAYS** implement proper authentication checks
- **ALWAYS** use transactions for multi-table operations
- **ALWAYS** sanitize user inputs before database operations

## 🔄 **Cache Management**

- **ALWAYS** use `revalidatePath` for related pages
- **ALWAYS** invalidate caches after data mutations
- **ALWAYS** consider the scope of cache invalidation
- **ALWAYS** use React Query for client-side caching
  description:
  globs:
  alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
