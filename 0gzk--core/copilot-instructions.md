## no-decorative-comment-separators

> description: Never use decorative dashed separators in code comments

---
description: Never use decorative dashed separators in code comments
alwaysApply: true
---

# No decorative comment separators

Never use long runs of dashes (or any other character) as decorative
separators inside code comments. No human writes code that way; they read
as visual noise and add zero information.

## Bad

```js
// --- Config -----------------------------------------------------------------
const PORT = 3000;
```

```py
# ---------- helpers ----------
def foo(): ...
```

```ts
// ====================================
// Section heading
// ====================================
```

## Good

```js
// Config
const PORT = 3000;
```

```py
// Helpers
def foo(): ...
```

If a section needs a comment header, just write it as a normal sentence on
one line. If multiple sections need to be visually distinct, that is what
**blank lines** and **function/file boundaries** are for.

This applies to every language and every file the agent writes or edits.

---
> Source: [0gzk/core](https://github.com/0gzk/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
