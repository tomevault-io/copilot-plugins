## development-guide-compliance

> Development guide compliance and code quality requirements


# Development Guide Compliance Rule

**MANDATORY REQUIREMENT**: All code MUST follow the established Development Guide (`DEVELOPMENT_GUIDE.md`).

## Architecture Standards Enforcement

### **TypeScript Conventions**
- **ALWAYS** use PascalCase for entity interfaces (`Employee`, `Product`, `Transaction`)
- **ALWAYS** use ComponentNameProps for props interfaces (`LoginFormProps`, `VirtualKeyboardProps`)
- **ALWAYS** use ContextNameType for context interfaces (`AuthContextType`, `POSContextType`)
- **NEVER** use `any` types - define proper interfaces

### **Component Structure (MANDATORY ORDER)**
```typescript
const ComponentName: React.FC<Props> = ({ prop1, prop2 }) => {
  // 1. Hooks (useState, useEffect, useContext)
  // 2. Event handlers  
  // 3. Computed values (useMemo)
  // 4. Effects (useEffect)
  // 5. Render
};
```

### **File Organization**
- **PLACE** shared components at `src/components/` root
- **GROUP** feature-specific components in folders (`Auth/`, `Layout/`)
- **FOLLOW** established import order: React hooks → Icons → Local contexts

### **State Management Patterns**
- **USE** `useState` for component-specific UI state
- **USE** `useReducer` for complex state logic
- **USE** Context for global app state (auth, cart)
- **IMPLEMENT** proper error boundaries

### **Performance Requirements**
- **USE** `React.memo` for components with frequent re-renders
- **USE** `useMemo` for expensive calculations
- **USE** `useCallback` for event handlers passed to children
- **AVOID** unnecessary re-renders

### **Error Handling Standards**
- **IMPLEMENT** error boundaries for component trees
- **HANDLE** async operations with try/catch
- **PROVIDE** user-friendly error messages
- **LOG** errors for debugging

## Code Quality Checklist

Before ANY component implementation:
- [ ] TypeScript interfaces properly defined
- [ ] Component structure follows established order
- [ ] Props interfaces use correct naming convention
- [ ] State management pattern appropriate for use case
- [ ] Error handling implemented
- [ ] Performance optimizations applied where needed
- [ ] Follows established file organization

## Testing Requirements
- **WRITE** unit tests for new functionality
- **TEST** touch interactions and accessibility
- **VERIFY** TypeScript compilation without errors
- **ENSURE** components work in isolation

**VIOLATION CONSEQUENCE**: Code not following development guide patterns will be rejected and must be refactored.

---
> Source: [portoyounes01/easysoft](https://github.com/portoyounes01/easysoft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
