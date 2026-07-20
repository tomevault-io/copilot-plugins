## style-guide-compliance

> Style guide compliance and design consistency requirements


# Style Guide Compliance Rule

**MANDATORY REQUIREMENT**: All UI/design decisions MUST follow the established Style Guide (`STYLE_GUIDE.md`).

## Design Standards Enforcement

### **Color System Compliance**
- **ALWAYS** use role-based color coding for employees:
  - Admin: `from-red-500 to-pink-600`
  - Manager: `from-orange-500 to-amber-600` 
  - Cashier: `from-blue-500 to-purple-600`
- **ALWAYS** use functional colors for actions:
  - Success/Confirm: `bg-green-500 hover:bg-green-600`
  - Delete/Danger: `bg-red-500 hover:bg-red-600`
  - Warning/Clear: `bg-orange-500 hover:bg-orange-600`
- **NEVER** introduce new colors without updating the style guide

### **Touch Screen Requirements (CRITICAL)**
- **MINIMUM** ~60px touch targets at default root (`min-h-touch` = `3.75rem`); menus `min-h-touch-sm`; compact `min-h-touch-xs`
- **LARGE** action buttons (`min-h-20` / ~5rem)
- **EMPLOYEE** cards (`min-h-70` / ~17.5rem)
- **AVOID** dropdowns, hover-dependent interactions, small fonts (<16px)

### **Typography Scale**
- **USE** established text sizes: `text-6xl` (titles), `text-2xl` (buttons), `text-xl` (labels)
- **MAINTAIN** font weight hierarchy: `font-bold` (titles), `font-semibold` (headers), `font-medium` (buttons)

### **Component Patterns**
- **FOLLOW** established button patterns (Primary: gradient, Secondary: gray, Keyboard: white with border)
- **USE** standard card structure with `bg-white rounded-3xl shadow-2xl p-8`
- **APPLY** consistent spacing with `gap-4`, `gap-6`, `gap-8` scale

### **Animation Standards**
- **USE** standard transitions: `duration-200` (buttons), `duration-300` (cards), `duration-150` (keyboard)
- **IMPLEMENT** hover effects: `hover:scale-105` (cards), gradient changes (buttons)

## Compliance Checklist

Before ANY UI component implementation:
- [ ] Touch targets meet 60px minimum
- [ ] Colors follow role-based or functional patterns
- [ ] Typography uses established scale
- [ ] Spacing follows gap system
- [ ] Animations use standard timings
- [ ] Component follows established patterns

**VIOLATION CONSEQUENCE**: Any deviation from style guide must be explicitly justified and documented.

---
> Source: [portoyounes01/easysoft](https://github.com/portoyounes01/easysoft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
