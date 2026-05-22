## security-validation

> - **ALWAYS** use Zod schemas for all user input validation

# Security & Data Validation (CRITICAL)

## 🔒 **Data Validation (MUST ENFORCE)**

- **ALWAYS** use Zod schemas for all user input validation
- **NEVER** trust client-side data without server-side validation
- **MUST** validate data in Server Actions before database operations
- **ALWAYS** use the validation schemas from [src/lib/schemas.ts](mdc:src/lib/schemas.ts)

## 🛡️ **Input Validation Examples**

```typescript
// ✅ CORRECT - Using Zod validation
export const CreatePostSchema = z.object({
    title: z
        .string()
        .min(1, '제목을 입력해주세요.')
        .max(100, '제목은 100글자 이하여야 합니다.')
        .transform((val) => val.trim()),
    content: z
        .string()
        .min(1, '내용을 입력해주세요.')
        .max(50000, '내용은 50,000글자 이하여야 합니다.'),
    hashtags: z
        .array(z.string().min(2).max(20))
        .min(1, '최소 하나의 해시태그가 필요합니다.')
        .max(10, '해시태그는 최대 10개까지 입력할 수 있습니다.'),
});

// ❌ WRONG - No validation or weak validation
export const createPost = async (data: any) => {
    // Direct database operation without validation
    return await supabase.from('posts').insert(data);
};
```

## 🚫 **FORBIDDEN Security Practices**

- **NEVER** use `any` type for user input
- **NEVER** bypass validation for "admin" users
- **NEVER** store sensitive data in client-side state
- **NEVER** use `eval()` or `innerHTML` with user input
- **NEVER** expose database credentials or API keys

## ✅ **REQUIRED Security Practices**

- **ALWAYS** validate all form inputs with Zod schemas
- **ALWAYS** use Server Actions for data mutations
- **ALWAYS** implement proper authentication checks
- **ALWAYS** sanitize user input before rendering
- **ALWAYS** use HTTPS in production
- **ALWAYS** implement rate limiting for API endpoints

## 🔐 **Authentication & Authorization**

- **ALWAYS** check `is_admin` field for admin operations
- **ALWAYS** validate user sessions on the server side
- **NEVER** rely solely on client-side authentication
- **MUST** implement proper session management with Supabase
  description:
  globs:
  alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
