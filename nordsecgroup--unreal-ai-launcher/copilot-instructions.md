## unreal-ai-launcher

> **MANDATORY RULE:** Whenever the user requests GUI creation, interface design, or UI components, you MUST use the shadcn/ui component library with Tailwind CSS.

# GUI Development Rules

## When Creating GUIs: Always Use shadcn/ui Framework

**MANDATORY RULE:** Whenever the user requests GUI creation, interface design, or UI components, you MUST use the shadcn/ui component library with Tailwind CSS.

### Required Actions:
1. **Use shadcn/ui components** - Never create custom components from scratch
2. **Install via MCP server** - Use natural language commands like "Add a login form using shadcn components"
3. **Tailwind CSS styling** - All styling must use Tailwind classes
4. **TypeScript/React** - Components must be built with TypeScript and React

### Component Preferences:
- Button: Use shadcn/ui Button component
- Form inputs: Use shadcn/ui Input, Textarea, Select components
- Layout: Use shadcn/ui Card, Dialog, Sheet components
- Navigation: Use shadcn/ui Tabs, Breadcrumb components
- Data display: Use shadcn/ui Table, Badge, Avatar components

### Installation Commands:
- "Add the [component name] component to my project"
- "Create a [feature] using shadcn components"
- "Install components from the shadcn registry"

### Technical Requirements:
- Use TypeScript for all components
- Follow shadcn/ui design patterns and conventions
- Ensure components are accessible and responsive
- Use proper semantic HTML structure
- Implement proper TypeScript interfaces for props

### Exception Handling:
If the user specifically requests a non-web GUI (desktop app, mobile native, etc.), acknowledge the mismatch and suggest:
1. Creating a separate web-based companion interface
2. Using the shadcn/ui design principles in the native framework
3. Building a web version alongside the desktop app

**CRITICAL:** Never build custom GUI components when shadcn/ui equivalents exist. Always prefer the shadcn/ui component library for web interfaces.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nordsecgroup) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
