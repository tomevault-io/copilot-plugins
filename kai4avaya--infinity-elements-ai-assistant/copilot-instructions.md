## infinity-elements-ai-assistant

> When creating new sample elements in this directory:

# Samples Folder - Development Rules

## Styling

When creating new sample elements in this directory:

- **ALWAYS** create a separate `Element.module.css` file for styles
- **NEVER** use inline styles (style prop) in the element JSX
- Use CSS modules for all styling needs
- Import styles with: `import styles from "./Element.module.css";`
- Apply styles using className: `<div className={styles.container}>`

## Structure

Each sample component should follow this structure:

```
sample-name/
├── Element.tsx           # Main React component
├── Element.module.css    # CSS module for styles
├── index.ts              # Web component registration
├── index.html            # Development testing page
└── README.md             # Documentation (optional)
```

## Example CSS Module Usage

```tsx
import React from "react";
import styles from "./Element.module.css";

export default function Element() {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>Hello</h1>
    </div>
  );
}
```

## Why CSS Modules?

- Better separation of concerns
- Easier to maintain and update styles
- Styles are scoped automatically (no conflicts)
- Better for production builds
- Cleaner element code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kai4avaya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
