## shadcn-ui

> This project uses **shadcn UI** exclusively for ALL UI elements and components.

# shadcn UI Usage Guide

This project uses **shadcn UI** exclusively for ALL UI elements and components.

## ⚠️ CRITICAL REQUIREMENTS

### 🚫 ZERO CUSTOM UI COMPONENTS POLICY 🚫

**THIS IS AN ABSOLUTE, NON-NEGOTIABLE REQUIREMENT:**

**ABSOLUTELY NO CUSTOM UI COMPONENTS ARE ALLOWED IN THIS PROJECT. PERIOD.**

Every single UI element, no matter how simple or complex, MUST use shadcn UI components. There are NO exceptions.

### What This Means in Practice

#### ❌ PROHIBITED ACTIONS (Never Do These):

- **NEVER** create custom buttons, even if you think it's just a simple button
- **NEVER** create custom inputs, textareas, or any form elements
- **NEVER** create custom cards, containers, or layout components
- **NEVER** create custom dialogs, modals, popovers, or overlays
- **NEVER** create custom dropdowns, selects, or menus
- **NEVER** create custom tabs, accordions, or collapsible sections
- **NEVER** create custom tooltips, toasts, or alerts
- **NEVER** create custom loading spinners or progress indicators
- **NEVER** create custom badges, avatars, or status indicators
- **NEVER** write Tailwind classes directly on `<div>`, `<button>`, `<input>`, etc. to build UI from scratch
- **NEVER** create components in locations other than `src/components/ui/` (except for business logic components that compose shadcn components)
- **NEVER** think "this is too simple for shadcn, I'll just create it myself"
- **NEVER** bypass this rule because "it would be faster to create a custom component"

#### ✅ REQUIRED ACTIONS (Always Do These):

- **ALWAYS** use shadcn UI components for 100% of UI elements
- **ALWAYS** check shadcn UI documentation FIRST before implementing ANY UI feature
- **ALWAYS** install the required shadcn component if it doesn't exist in the project
- **ALWAYS** compose shadcn components together for complex UI patterns
- **ALWAYS** use shadcn's styling system and variants
- **ALWAYS** import from `@/components/ui/` for all UI elements

### The Workflow for ANY UI Implementation:

1. **STOP**: Before writing any UI code, ask yourself "Does shadcn UI have this component?"
2. **CHECK**: Visit https://ui.shadcn.com/docs/components and verify the component exists
3. **INSTALL**: If not already installed, run `pnpm dlx shadcn@latest add <component-name>`
4. **USE**: Import and use the shadcn component directly
5. **NEVER**: Build a custom alternative, no matter the reason

### Examples of Correct Usage

✅ **Correct**: Using shadcn Button
```typescript
import { Button } from "@/components/ui/button";
<Button>Click me</Button>
```

❌ **Wrong**: Creating custom button
```typescript
// NEVER DO THIS
<button className="px-4 py-2 bg-blue-500 rounded">Click me</button>
```

✅ **Correct**: Using shadcn Card
```typescript
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
</Card>
```

❌ **Wrong**: Creating custom card
```typescript
// NEVER DO THIS
<div className="border rounded-lg shadow p-4">
  <h2>Title</h2>
  <p>Content</p>
</div>
```

## Initialization

If shadcn UI is not initialized in the project, use the following command:

```bash
pnpm dlx shadcn@latest init
```

## Adding Components

When a particular component is not installed, use the following command format:

```bash
pnpm dlx shadcn@latest add <component-name>
```

### Example

To install the button component:

```bash
pnpm dlx shadcn@latest add button
```

### Common Components

Frequently used components you should install as needed:

```bash
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add dialog
pnpm dlx shadcn@latest add input
pnpm dlx shadcn@latest add label
pnpm dlx shadcn@latest add select
pnpm dlx shadcn@latest add form
pnpm dlx shadcn@latest add dropdown-menu
pnpm dlx shadcn@latest add avatar
pnpm dlx shadcn@latest add badge
```

## 🔐 Clerk Authentication Integration

### 🚨 MANDATORY Clerk Implementation Pattern 🚨

**THIS IS THE ONLY ACCEPTABLE WAY TO IMPLEMENT CLERK AUTHENTICATION:**

Clerk authentication MUST be implemented using **shadcn UI Button** and **shadcn UI Dialog** components. The Clerk sign-in and sign-up forms must appear in MODAL dialogs, NOT on separate pages or embedded directly.

### ✅ REQUIRED Implementation Pattern (The ONLY Way)

```typescript
import { SignIn, SignUp } from "@clerk/nextjs";
import { Button } from "@/components/ui/button";
import { Dialog, DialogContent, DialogTrigger } from "@/components/ui/dialog";

// Sign In Modal - THIS IS THE ONLY ACCEPTABLE PATTERN
<Dialog>
  <DialogTrigger asChild>
    <Button>Sign In</Button>
  </DialogTrigger>
  <DialogContent>
    <SignIn routing="virtual" />
  </DialogContent>
</Dialog>

// Sign Up Modal - THIS IS THE ONLY ACCEPTABLE PATTERN
<Dialog>
  <DialogTrigger asChild>
    <Button>Sign Up</Button>
  </DialogTrigger>
  <DialogContent>
    <SignUp routing="virtual" />
  </DialogContent>
</Dialog>
```

### 🚫 PROHIBITED Clerk Patterns

❌ **NEVER** use `<SignInButton>` from Clerk
```typescript
// WRONG - DO NOT USE CLERK'S BUTTONS
import { SignInButton } from "@clerk/nextjs";
<SignInButton /> // ❌ FORBIDDEN
```

❌ **NEVER** use `<SignUpButton>` from Clerk
```typescript
// WRONG - DO NOT USE CLERK'S BUTTONS
import { SignUpButton } from "@clerk/nextjs";
<SignUpButton /> // ❌ FORBIDDEN
```

❌ **NEVER** create custom buttons for authentication
```typescript
// WRONG - NO CUSTOM BUTTONS
<button onClick={...}>Sign In</button> // ❌ FORBIDDEN
<div className="..." onClick={...}>Sign Up</div> // ❌ FORBIDDEN
```

❌ **NEVER** use redirect routing for authentication
```typescript
// WRONG - MUST USE MODAL/VIRTUAL ROUTING
<SignIn routing="path" /> // ❌ FORBIDDEN
<SignUp routing="hash" /> // ❌ FORBIDDEN
```

### ✅ Key Requirements for Clerk Integration

**MANDATORY REQUIREMENTS:**

1. **Button Component**: Use ONLY `<Button>` from `@/components/ui/button` for all authentication triggers
2. **Dialog Component**: Use ONLY `<Dialog>` from `@/components/ui/dialog` to display authentication forms in modals
3. **Virtual Routing**: Set `routing="virtual"` on ALL Clerk `<SignIn>` and `<SignUp>` components
4. **Modal Presentation**: Authentication forms MUST appear in modal dialogs, not as standalone pages
5. **No Clerk Buttons**: NEVER import or use `SignInButton`, `SignUpButton`, or any pre-styled Clerk button components
6. **No Custom Auth UI**: NEVER create custom buttons, forms, or UI elements for authentication

### Complete Working Example

```typescript
import { SignIn, SignUp } from "@clerk/nextjs";
import { Button } from "@/components/ui/button";
import { Dialog, DialogContent, DialogTrigger } from "@/components/ui/dialog";

export function AuthButtons() {
  return (
    <div className="flex gap-2">
      {/* Sign In Modal */}
      <Dialog>
        <DialogTrigger asChild>
          <Button variant="outline">Sign In</Button>
        </DialogTrigger>
        <DialogContent className="sm:max-w-md">
          <SignIn routing="virtual" />
        </DialogContent>
      </Dialog>

      {/* Sign Up Modal */}
      <Dialog>
        <DialogTrigger asChild>
          <Button>Sign Up</Button>
        </DialogTrigger>
        <DialogContent className="sm:max-w-md">
          <SignUp routing="virtual" />
        </DialogContent>
      </Dialog>
    </div>
  );
}
```

### Why These Requirements Exist

- **Consistency**: All UI elements use shadcn UI, including authentication
- **Customization**: shadcn Button allows full control over styling and variants
- **User Experience**: Modal dialogs provide better UX than redirecting to separate pages
- **Maintainability**: Single source of truth for all UI components

## Component Location

- All shadcn UI components are installed in `src/components/ui/` directory
- Import components from `@/components/ui/component-name`
- **NEVER** modify the installed components directly (they may be updated)
- If customization is needed, compose or wrap the shadcn component, don't recreate it

## ⚖️ ENFORCEMENT & COMPLIANCE

### 🔒 This Rule is ALWAYS APPLIED and ABSOLUTELY NON-NEGOTIABLE

**THIS IS NOT A SUGGESTION. THIS IS A MANDATORY REQUIREMENT.**

- This rule applies to **EVERY SINGLE UI ELEMENT** in the project
- This rule applies to **EVERY DEVELOPER** working on the project
- This rule applies **100% OF THE TIME** with **ZERO EXCEPTIONS**
- There is **NO CIRCUMSTANCE** where custom UI components are acceptable
- There is **NO JUSTIFICATION** for bypassing this rule

### 🛑 If You Find Yourself Writing Custom UI Code:

1. **STOP IMMEDIATELY**
2. **DELETE THE CUSTOM CODE**
3. **CHECK** shadcn UI documentation for the appropriate component
4. **INSTALL** the shadcn component if needed
5. **USE** the shadcn component instead

### 🚨 Red Flags to Watch For:

If you see ANY of these patterns in the code, they are VIOLATIONS:

- `<button className="...">` ← Should use `<Button>` from shadcn UI
- `<input className="...">` ← Should use `<Input>` from shadcn UI
- `<div className="border rounded-lg ...">` ← Probably should use `<Card>` from shadcn UI
- Custom component files for basic UI elements
- Styling HTML elements directly instead of using shadcn components
- Importing `SignInButton` or `SignUpButton` from Clerk

### ✅ Code Review Checklist:

Before committing any UI code, verify:

- [ ] Are you using ONLY shadcn UI components for all UI elements?
- [ ] Have you avoided creating ANY custom UI components?
- [ ] Are all buttons using `<Button>` from `@/components/ui/button`?
- [ ] Are all dialogs/modals using `<Dialog>` from `@/components/ui/dialog`?
- [ ] Are Clerk authentication forms displayed in shadcn Dialog modals?
- [ ] Are you using shadcn Button components (NOT Clerk buttons) for auth triggers?
- [ ] Have you checked shadcn UI documentation before implementing any UI feature?

### ⚠️ Remember:

**"Can I create a quick custom button for this?"** → **NO. Use shadcn Button.**

**"This is too simple to need shadcn UI..."** → **NO. Use shadcn UI anyway.**

**"It would be faster to just style a div..."** → **NO. Use the appropriate shadcn component.**

**"Clerk has a built-in button component..."** → **NO. Use shadcn Button + Dialog.**

---

**IF YOU VIOLATE THIS RULE, THE CODE WILL BE REJECTED AND YOU WILL NEED TO REFACTOR IT TO USE SHADCN UI COMPONENTS.**

---
> Source: [Rivexo/FlashyCardyCourse](https://github.com/Rivexo/FlashyCardyCourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
