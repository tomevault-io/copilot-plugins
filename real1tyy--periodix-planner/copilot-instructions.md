## modular-utility-separation

> Enforce strict modular separation of utility functions from business logic


# Modular Utility Separation Rule

**CRITICAL: Any function that is NOT directly related to business logic MUST be extracted to [libs/utils](mdc:libs/utils) for maximum code reuse and DRY principles.**

## 🚨 **Zero Tolerance Policy**

- **NO utility functions in business logic classes**
- **NO duplicate implementations across the codebase**
- **NO "convenience" methods that could be utilities**
- **ALWAYS check [libs/utils](mdc:libs/utils) before writing ANY helper function**

## ✅ **MUST Extract to Utils**

### Date/Time Operations
```typescript
// ❌ VIOLATION: In component class
private formatDateTimeForInput(dateString: string): string {
  // Date formatting logic
}

// ✅ CORRECT: In libs/utils/src/date-utils.ts
export const formatDateTimeForInput = (dateString: string): string => {
  // Reusable date formatting logic
}
```

### File Operations
```typescript
// ❌ VIOLATION: Duplicate file generation
private generateUniqueFileName(file: TFile): string {
  // File naming logic
}

// ✅ CORRECT: Use existing utility
import { generateUniqueFilePath } from "../../../../libs/utils/src/file-operations";
const uniquePath = generateUniqueFilePath(app, folder, baseName);
```

### String Manipulation
```typescript
// ❌ VIOLATION: String utils in business class
private sanitizeTitle(title: string): string {
  return title.replace(/[^a-zA-Z0-9]/g, '-');
}

// ✅ CORRECT: In libs/utils/src/string-utils.ts
export const sanitizeForFilename = (input: string): string => {
  return input.replace(/[^a-zA-Z0-9]/g, '-');
}
```

### Validation Functions
```typescript
// ❌ VIOLATION: Validation in component
private isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// ✅ CORRECT: In libs/utils/src/validation-utils.ts
export const isValidEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

### DOM/UI Helpers
```typescript
// ❌ VIOLATION: DOM helpers in business logic
private createSettingItem(container: HTMLElement, label: string): HTMLElement {
  // DOM creation logic
}

// ✅ CORRECT: In libs/utils/src/dom-utils.ts
export const createSettingItem = (container: HTMLElement, label: string): HTMLElement => {
  // Reusable DOM creation
}
```
## 🔍 **Detection Patterns**

### Functions That MUST Be Extracted:
- Any function with `format`, `parse`, `convert` in the name
- Any function that manipulates strings, dates, or arrays
- Any function that creates DOM elements
- Any function that validates input
- Any function that generates file names/paths
- Any function that handles async operations generically
- Any function used in multiple components

### Business Logic That STAYS:
- Event handlers specific to the component
- State management for the component
- API calls specific to the domain
- Workflow orchestration
- Component lifecycle methods

## 🛠 **Implementation Process**

1. **Before Writing ANY Helper Function:**
   - Check if it exists in [libs/utils](mdc:libs/utils)
   - If similar exists, extend it instead of duplicating

2. **When Creating Utility Functions:**
   - Create in appropriate utils file
   - Export from [libs/utils/src/index.ts](mdc:libs/utils/src/index.ts)
   - Use descriptive, generic names
   - Add proper TypeScript types
   - Include JSDoc only if it adds real value

3. **Import Pattern:**
   ```typescript
   // ✅ CORRECT: Use absolute imports with utils alias
   import { formatDateTimeForInput, inputValueToISOString } from "utils/date-utils";
   import { generateUniqueFilePath, getFilenameFromPath } from "utils/file-utils";

   // ❌ WRONG: Fragile relative imports
   import { formatDateTimeForInput } from "../../../../libs/utils/src/date-utils";
   ```

## 🚨 **Enforcement Examples**

### Event Edit Modal - BEFORE (VIOLATION)
```typescript
export class EventEditModal extends Modal {
  // ❌ VIOLATION: Date formatting in business logic
  private formatDateTimeForInput(dateString: string): string {
    // 15 lines of date formatting code
  }

  // ❌ VIOLATION: Manual ISO conversion
  private saveEvent(): void {
    start: new Date(this.startInput.value).toISOString()
  }
}
```

### Event Edit Modal - AFTER (CORRECT)
```typescript
import { formatDateTimeForInput, inputValueToISOString } from "utils/date-utils";

export class EventEditModal extends Modal {
  // ✅ CORRECT: Pure business logic only
  private saveEvent(): void {
    const eventData = {
      start: inputValueToISOString(this.startInput.value),
      // ... other business logic
    };
  }
}
```

## 📋 **Pre-Commit Checklist**

- [ ] No utility functions in component/service classes
- [ ] All date/string/file operations use utils
- [ ] No duplicate implementations
- [ ] New utilities added to [libs/utils/src/index.ts](mdc:libs/utils/src/index.ts)
- [ ] **Imports use absolute `utils/*` paths (NOT relative `../../../` paths)**
- [ ] Business logic classes focus ONLY on domain concerns

## 🚨 **Import Pattern Enforcement**

### ✅ **ALWAYS Use Absolute Imports**
```typescript
// ✅ CORRECT: Absolute imports with utils alias
import { formatDateTimeForInput } from "utils/date-utils";
import { generateUniqueFilePath } from "utils/file-utils";
import { SettingsStore } from "utils/settings-store";
```

### ❌ **NEVER Use Relative Imports for Utils**
```typescript
// ❌ WRONG: Fragile relative imports
import { formatDateTimeForInput } from "../../../../libs/utils/src/date-utils";
import { generateUniqueFilePath } from "../../../libs/utils/src/file-utils";
import { SettingsStore } from "../../../libs/utils/src/settings-store";
```

**Why Absolute Imports?**
- **Maintainable**: No need to count `../` when moving files
- **Refactor-safe**: File moves don't break imports
- **Readable**: Clear indication of external dependencies
- **IDE-friendly**: Better autocomplete and navigation

## 🎯 **Goal: Maximum Reusability**

Every utility function should be:
- **Generic**: Works across different domains
- **Pure**: No side effects when possible
- **Testable**: Easy to unit test in isolation
- **Reusable**: Used by multiple components/apps
- **Well-typed**: Proper TypeScript interfaces

**Remember: If it's not business logic, it belongs in utils. No exceptions.**

---
> Source: [Real1tyy/Periodix-Planner](https://github.com/Real1tyy/Periodix-Planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
