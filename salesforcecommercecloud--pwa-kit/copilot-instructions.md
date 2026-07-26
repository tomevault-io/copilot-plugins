## pwa-kit

> When accessibility is checked. Check if heading levels should increase sequentially for semantic structure


# Rule: accessibility-heading-order

Heading levels should increase sequentially for semantic structure

## 🔍 Pattern

```regex
<h([3-6])>
```

## 📍 Examples

```tsx
// ❌ Bad
<h1>Main Title</h1>
<h4>Subsection</h4>
// ✅ Good
<h1>Main Title</h1>
<h2>Subsection</h2>
```

---
> Source: [SalesforceCommerceCloud/pwa-kit](https://github.com/SalesforceCommerceCloud/pwa-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
