## simple-journal

> - Always reuse existing functions and code

# Cursor Rules for Simple Journal Project

## Code Quality and Organization

### 1. Avoid Code Duplication
- Always reuse existing functions and code
- Merge similar code whenever reasonable
- Extract common functionality into reusable methods
- Prefer composition over duplication

### 2. CSS and Styling
- Never hard-code CSS styles in JavaScript code
- Always put styles in CSS files (main.css, print.css, etc.)
- Use CSS classes and data attributes for styling hooks
- Keep styling concerns separate from logic

### 3. Encapsulation and Boundaries
- Always respect class/module/API boundaries
- Private variables and members should be kept private
- Never access private members externally directly
- Use getters and setters if absolutely necessary
- Group related functionality together
- Don't mix concerns - keep UI logic, data logic, and business logic separate

### 4. Constants and Magic Numbers
- Don't use magic numbers in code
- Use constants at the top of respective files for values with 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahagelberg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
