## core-rules

> Core rules - always apply


# Core Rules

## Language

- Commit messages must be in English
- Documentation and comments must be in English

## Comments

- All comments MUST be in English - no Chinese or other languages allowed
- No obvious comments
- Comments explain "why" not "what"
- Do not explain code changes in comments

### Examples

```typescript
// Bad - Chinese comment
// 初始化用户数据

// Good - English comment explaining why
// Pre-load user data to avoid waterfall requests on dashboard mount

// Bad - obvious comment
// Increment counter by 1
counter++

// Good - explains the business reason
// Track daily limit to enforce free tier restrictions
counter++
```

---
> Source: [xswitch-cn/CherryPBX](https://github.com/xswitch-cn/CherryPBX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
