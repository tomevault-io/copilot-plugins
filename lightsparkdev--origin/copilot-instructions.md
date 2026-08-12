## boundary

> Origin is the design system - keep it product-agnostic


# Origin Boundary

Origin is the design system. Products consume it as a package dependency.

## This Repo

**This is the workspace you build in.** All components and tokens are created here.

- Generic, reusable components
- Design tokens and typography
- Before adding logic to a component, pause and ask: should this logic live in the primitive or in the product? Check with the user before proceeding.

## Product Reference

If a consuming product repo exists in the workspace, use it as read-only reference:

- **Never modify** product files from this workspace
- Use as reference for how Origin components are consumed in practice
- Learn from real usage patterns when designing component APIs

## Decision Guide

| If you're adding... | It belongs in... |
|---------------------|------------------|
| Reusable UI pattern | Origin |
| Product-specific layout | Consuming product |
| Generic interaction (button, input) | Origin |
| Business logic | Consuming product |
| Design tokens | Origin |
| App-specific tokens | Consuming product |

---
> Source: [lightsparkdev/origin](https://github.com/lightsparkdev/origin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
