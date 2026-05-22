## code-style

> Based on [.prettierrc](mdc:.prettierrc):

# Code Style & Formatting (MANDATORY)

## 🎨 **Prettier Configuration (MUST FOLLOW)**

Based on [.prettierrc](mdc:.prettierrc):

- **ALWAYS** use semicolons (`semi: true`)
- **ALWAYS** use trailing commas (`trailingComma: "es5"`)
- **ALWAYS** use single quotes (`singleQuote: true`)
- **ALWAYS** limit line length to 80 characters (`printWidth: 80`)
- **ALWAYS** use 4 spaces for indentation (`tabWidth: 4`)
- **NEVER** use tabs (`useTabs: false`)

## 📝 **Code Formatting Examples**

```typescript
// ✅ CORRECT - Following Prettier rules
export const createPostAction = async (formData: FormData) => {
    try {
        const rawData = {
            title: formData.get('title') as string,
            content: formData.get('content') as string,
            hashtags:
                (formData.get('hashtags') as string)
                    ?.split(',')
                    .map((tag: string) => tag.trim())
                    .filter((tag: string) => tag.length > 0) || [],
        };

        // ... rest of the function
    } catch (error) {
        throw error;
    }
};

// ❌ WRONG - Violating Prettier rules
export const createPostAction = async (formData: FormData) => {
    try {
        const rawData = {
            title: formData.get('title') as string,
            content: formData.get('content') as string,
        };
    } catch (error) {
        throw error;
    }
};
```

## 🚫 **FORBIDDEN Formatting Practices**

- **NEVER** mix single and double quotes in the same file
- **NEVER** exceed 80 characters per line
- **NEVER** use inconsistent indentation
- **NEVER** omit semicolons
- **NEVER** omit trailing commas where appropriate

## ✅ **REQUIRED Style Practices**

- **ALWAYS** use consistent spacing around operators
- **ALWAYS** use proper line breaks for readability
- **ALWAYS** group related imports together
- **ALWAYS** use descriptive variable and function names
- **ALWAYS** add proper JSDoc comments for complex functions
  description:
  globs:
  alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
