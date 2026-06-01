## project-organization

> Project organization

# Project Organization Rules

## Directory Structure Requirements

### Required: Component Organization
All components must be organized in a dedicated components directory:

```yaml
rule:
  id: component-directory-structure
  pattern: |
    project/
    ├── src/
    │   ├── components/
    │   │   └── *.ts
    │   └── main.ts
    ├── index.html
    └── package.json
  message: |
    Components must be placed in the src/components directory.
    This helps maintain a clean and organized codebase.
  severity: error
```

### Required: Component File Naming
Component files should follow these naming conventions:

```yaml
rule:
  id: component-file-naming
  pattern: |
    src/components/[component-name].ts
  message: |
    Component files should:
    1. Be placed in src/components directory
    2. Use kebab-case for file names
    3. Have a .ts extension for TypeScript
  severity: error
```

### Best Practice: Component Organization
Follow these guidelines for component organization:

```yaml
rule:
  id: component-organization-best-practices
  pattern: |
    src/
    ├── components/     # Reusable UI components
    ├── pages/          # Page components (optional)
    ├── models/         # Models or Stores (optional)
    ├── utils/          # Utility functions
    ├── types/          # TypeScript types/interfaces
    └── main.ts         # Application entry point
  message: |
    Organize your project with clear separation of concerns:
    - components/: for reusable UI components
    - models/: for models or stores (if needed)
    - pages/: for page components (if needed)
    - utils/: for utility functions
    - types/: for TypeScript types and interfaces
  severity: warning
```

## Usage
1. Place all component files in `src/components/`
2. Use kebab-case for component file names
3. Use camel-case for class file names
4. Create additional directories (models, pages, etc.) as needed
5. Keep main.ts as the application entry point 

---
> Source: [alexawilcox1120/TestingCursor](https://github.com/alexawilcox1120/TestingCursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
