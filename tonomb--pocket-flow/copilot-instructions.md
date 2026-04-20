## pocket-flow

> How to add new windsurf rules to the project


# Windsurf Rules Location

How to add new windsurf rules to the project

1. Always place rule files in PROJECT_ROOT/.windsurf/rules/:

   ```
   .windsurf/rules/
   ├── your-rule-name.mdc
   ├── another-rule.mdc
   └── ...
   ```

2. Follow the naming convention:

   - Use kebab-case for filenames
   - Always use .md extension
   - Make names descriptive of the rule's purpose

3. Directory structure:

   ```
   PROJECT_ROOT/
   ├── .windsurf/
   │   └── rules/
   │       ├── your-rule-name.md
   │       └── ...
   └── ...
   ```

4. Never place rule files:

   - In the project root
   - In subdirectories outside .windsurf/rules
   - In any other location

5. Windsurf rules have the following structure:

````
---
description: Short description of the rule's purpose
globs: optional/path/pattern/**/*
trigger: false
---
# Rule Title

Main content explaining the rule with markdown formatting.
md
1. Step-by-step instructions
2. Code examples
3. Guidelines

Example:
```typescript
// Good example
function goodExample() {
  // Implementation following guidelines
}

// Bad example
function badExample() {
  // Implementation not following guidelines
}
```
````

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tonomb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
