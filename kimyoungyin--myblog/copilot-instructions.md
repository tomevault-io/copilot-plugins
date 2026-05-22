## error-handling

> - **ALWAYS** implement proper error boundaries

# Error Handling & User Experience (MANDATORY)

## ⚠️ **Error Handling (MUST IMPLEMENT)**

- **ALWAYS** implement proper error boundaries
- **ALWAYS** provide user-friendly error messages
- **ALWAYS** log errors for debugging purposes
- **NEVER** let errors crash the application
- **NEVER** expose technical error details to end users

## 🎯 **Error Handling Patterns (CRITICAL)**

- **ALWAYS** use try-catch blocks in async operations
- **ALWAYS** validate user inputs before processing
- **ALWAYS** handle network errors gracefully
- **ALWAYS** provide fallback UI for failed operations

```typescript
// ✅ CORRECT - Proper error handling
try {
    const result = await riskyOperation();
    return result;
} catch (error) {
    // Log error for debugging
    console.error('Operation failed:', error);

    // Provide user-friendly message
    if (error instanceof ValidationError) {
        throw new Error('입력 데이터가 올바르지 않습니다.');
    } else if (error instanceof NetworkError) {
        throw new Error('네트워크 연결을 확인해주세요.');
    } else {
        throw new Error('작업 중 오류가 발생했습니다.');
    }
}

// ❌ WRONG - Poor error handling
const result = await riskyOperation(); // No error handling
return result;
```

## 🚫 **FORBIDDEN Error Practices**

- **NEVER** ignore errors silently
- **NEVER** show technical error messages to users
- **NEVER** let unhandled promise rejections occur
- **NEVER** use console.error in production
- **NEVER** create infinite error loops

## ✅ **REQUIRED Error Practices**

- **ALWAYS** implement error boundaries for React components
- **ALWAYS** provide loading states for async operations
- **ALWAYS** implement retry mechanisms for failed operations
- **ALWAYS** use proper HTTP status codes
- **ALWAYS** implement graceful degradation

## 🔍 **Debugging & Logging**

- **ALWAYS** log errors with sufficient context
- **ALWAYS** use structured logging in production
- **ALWAYS** implement proper error tracking
- **NEVER** log sensitive information
- **NEVER** use console.log in production code

## 🎨 **User Experience**

- **ALWAYS** provide clear feedback for user actions
- **ALWAYS** implement proper loading states
- **ALWAYS** use toast notifications for success/error messages
- **ALWAYS** implement proper form validation feedback
- **ALWAYS** provide helpful error recovery suggestions
  description:
  globs:
  alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
