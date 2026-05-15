## coding-standards

> handleSuccess(result)

Rule Name: coding-standards
Description: 
This rule defines the coding standards and formatting guidelines that must be followed for all code changes in this project.

<coding_format>

A. Syntax & Structure
- File names must be dash-case (word-cloud.service.ts) unless an existing pattern differs.
- Group imports: node/standard → npm packages → internal paths. No unused imports.
- Use arrow functions everywhere except inside class bodies, where concise method syntax is allowed.
- Prefer early returns; nested if/else blocks deeper than two levels are disallowed.
- Early returns must use block format with braces (e.g., `if (!value) { return }`) for readability.
- Extract function call results as scope variables before using in conditions (e.g., `const trimmedText = text.trim(); if (!trimmedText) {...}` instead of `if (!text.trim()) {...}`).
- Use async/await—never chain .then().
- No .forEach for side effects; use for (const x of arr) instead.
- Array combinators (map, reduce, filter) are allowed only when you return their result.
- Identifiers must be English.
- No commented code allowed.

B. Functional-Programming Rules
- Each function must:
    * Be ≤ 50 lines (preferably; extract helpers if longer).
    * Take ≤ 4 parameters (optional ones last).
    * Have a single responsibility.
    * Be pure unless it is an intentional I/O wrapper (e.g. DB write); such wrappers must be ≤ 15 lines.
    * Name functions with camelCase imperative verbs (calculateTotals, getUserById).

C. Type Safety & Error Handling
- Explicitly type all function parameters, return types, and exported constants.
- Type all local variables inside a function.
- **Special attention for async operations**: Variables from awaited functions (e.g., `const { userId } = await auth()`) must be explicitly typed, especially in Next.js components where auth results should use proper domain types.
- No any; if an external library forces it, wrap and narrow.
- Error handling in catch blocks:
  - If the error variable is not used, use `catch {}` (no parameter).
  - If the error is used, type it as `unknown` and handle it safely within the catch block.

D. React Component Standards

- Always define props with interfaces, never inline types
- Place interfaces directly above component definitions
- Use const arrow functions for component definitions
- Use implicit return syntax when components only return JSX (no logic before return)
- Export components using export default pattern (required for Next.js pages/layouts)
- Handler functions inside components must be ≤ 20 lines and have a single, clear responsibility. Extract helper functions for complex logic.

  **Wrong (~50 lines in one handler):**
  ```tsx
  const handleFormSubmit = async (): Promise<void> => {
    const trimmedName: string = formData.name.trim()
    const trimmedEmail: string = formData.email.trim()
    const trimmedMessage: string = formData.message.trim()
    
    if (!trimmedName) {
      setErrors({ ...errors, name: "Name is required" })
      toast({ title: "Error", description: "Name is required", variant: "destructive" })
      return
    }
    
    if (!trimmedEmail || !trimmedEmail.includes("@")) {
      setErrors({ ...errors, email: "Valid email is required" })
      toast({ title: "Error", description: "Valid email is required", variant: "destructive" })
      return
    }
    
    if (!trimmedMessage || trimmedMessage.length < 10) {
      setErrors({ ...errors, message: "Message must be at least 10 characters" })
      toast({ title: "Error", description: "Message too short", variant: "destructive" })
      return
    }
    
    setIsSubmitting(true)
    setErrors({})
    
    try {
      const payload: FormPayload = {
        name: trimmedName,
        email: trimmedEmail,
        message: trimmedMessage,
        timestamp: new Date().toISOString()
      }
      
      const response: Response = await fetch("/api/contact", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      })
      
      if (!response.ok) {
        throw new Error("Failed to submit")
      }
      
      const result: SubmissionResult = await response.json()
      
      setFormData({ name: "", email: "", message: "" })
      setSubmissionCount(prev => prev + 1)
      
      toast({ title: "Success", description: "Message sent successfully!" })
      
      if (onSuccess) {
        onSuccess(result)
      }
    } catch (error: unknown) {
      const errorMessage: string = error instanceof Error ? error.message : "Unknown error"
      console.error("Submission error:", errorMessage)
      setErrors({ submit: "Failed to send message" })
      toast({ title: "Error", description: "Failed to send message", variant: "destructive" })
    } finally {
      setIsSubmitting(false)
    }
  }
  ```

  **Good (broken into focused helpers ≤ 20 lines each):**
  ```tsx
  const validateForm = (): boolean => {
    const trimmedName: string = formData.name.trim()
    const trimmedEmail: string = formData.email.trim()
    const trimmedMessage: string = formData.message.trim()
    
    if (!trimmedName) {
      setErrors({ ...errors, name: "Name is required" })
      toast({ title: "Error", description: "Name is required", variant: "destructive" })
      return false
    }
    
    if (!trimmedEmail || !trimmedEmail.includes("@")) {
      setErrors({ ...errors, email: "Valid email is required" })
      toast({ title: "Error", description: "Valid email is required", variant: "destructive" })
      return false
    }
    
    if (!trimmedMessage || trimmedMessage.length < 10) {
      setErrors({ ...errors, message: "Message must be at least 10 characters" })
      toast({ title: "Error", description: "Message too short", variant: "destructive" })
      return false
    }
    
    return true
  }
  
  const submitForm = async (): Promise<SubmissionResult> => {
    const payload: FormPayload = {
      name: formData.name.trim(),
      email: formData.email.trim(),
      message: formData.message.trim(),
      timestamp: new Date().toISOString()
    }
    
    const response: Response = await fetch("/api/contact", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    })
    
    if (!response.ok) {
      throw new Error("Failed to submit")
    }
    
    return response.json()
  }
  
  const handleSuccess = (result: SubmissionResult): void => {
    setFormData({ name: "", email: "", message: "" })
    setSubmissionCount((prev: number) => prev + 1)
    toast({ title: "Success", description: "Message sent successfully!" })
    
    if (onSuccess) {
      onSuccess(result)
    }
  }
  
  const handleError = (error: unknown): void => {
    const errorMessage: string = error instanceof Error ? error.message : "Unknown error"
    console.error("Submission error:", errorMessage)
    setErrors({ submit: "Failed to send message" })
    toast({ title: "Error", description: "Failed to send message", variant: "destructive" })
  }
  
  const handleFormSubmit = async (): Promise<void> => {
    const isValid: boolean = validateForm()
    
    if (!isValid) {
      return
    }
    
    setIsSubmitting(true)
    setErrors({})
    
    try {
      const result: SubmissionResult = await submitForm()
      handleSuccess(result)
    } catch (error: unknown) {
      handleError(error)
    } finally {
      setIsSubmitting(false)
    }
  }
  ```
- Example with implicit return:

  ```tsx
  interface MyComponentProps {
    title: string
    children: React.ReactNode
  }

  const MyComponent = ({ title, children }: MyComponentProps) => (
    <div>
      {title}
      {children}
    </div>
  )

  export default MyComponent
  ```

- Example with explicit return (when logic is present):

  ```tsx
  interface MyComponentProps {
    title: string
    children: React.ReactNode
  }

  const MyComponent = ({ title, children }: MyComponentProps) => {
    const processedTitle = title.toUpperCase()
    
    return (
      <div>
        {processedTitle}
        {children}
      </div>
    )
  }

  export default MyComponent
  ```

- Normal components that are not Next.js pages/layouts should be exported
  using export const pattern
- Example with implicit return:

  ```tsx
  interface NotAPageOrLayoutComponentProps {
    title: string
    children: React.ReactNode
  }

  export const NotAPageOrLayoutComponent = ({
    title,
    children
  }: NotAPageOrLayoutComponentProps) => (
    <div>
      {title}
      {children}
    </div>
  )

E. Component Granularity & Organization
- Break down large components into smaller, focused components for better maintainability.
- When a component contains multiple logical sections (e.g., Card with CardHeader + CardContent), extract each section into separate components.
- Create dedicated folders for related component groups:
  * Use kebab-case folder names matching the main component concept
  * Place related sub-components within the same folder using kebab-case file names
  * Example structure: `component-name/component-name-header.tsx`, `component-name/component-name-content.tsx`
- Each sub-component should have a single, clear responsibility.
- Maintain the parent component as a composition wrapper that orchestrates child components.
- Follow this pattern when refactoring existing components or creating new feature components.

F. Advanced Component Architecture Patterns

F.1. Pure Functions and Constants Organization
- **Pure functions** (no side effects, deterministic output) must be extracted outside components:
  * Place above the component definition
  * Examples: `getGreeting()`, `getMembershipBadgeColor()`, `formatDate()`
- **Constants and static data** must be moved outside components:
  * Place after imports and interfaces, before pure functions
  * Use SCREAMING_SNAKE_CASE for naming (e.g., `TEMPLATE_FEATURES`, `TECH_STACK`)
  * **Always explicitly type constants** with appropriate type annotations
  * Examples: `const API_URL: string = "..."`, `const MAX_RETRIES: number = 3`
  * Use `as const` for immutable values when type inference is sufficient
  * Group related constants together

F.1.1. Custom Hooks Organization
- **Custom hooks** must be extracted to separate files in the `/hooks/` directory:
  * Use kebab-case file naming: `use-scroll-detection.ts`, `use-local-storage.ts`
  * Start hook names with `use` prefix following React conventions
  * Place hooks in `/hooks/` folder at project root level
  * Export hooks using named exports: `export const useScrollDetection = () => {}`
  * **Always explicitly type hook return values** and parameters
  * Examples: `useScrollDetection(): boolean`, `useLocalStorage<T>(key: string): [T, (value: T) => void]`
  * Group related hooks in the same file only if they're tightly coupled

F.1.2. Whitespace and Formatting Rules
- **Component variable organization**: Maintain consistent whitespace between different types of declarations:
  * Add a blank line between React state declarations and custom hook calls
  * Add a blank line between custom hook calls and other variable declarations
  * Example:
    ```tsx
    const [isMenuOpen, setIsMenuOpen] = useState<boolean>(false)
    const [isVisible, setIsVisible] = useState<boolean>(true)
    
    const isScrolled: boolean = useScrollDetection()
    const userData: UserData = useUserData()
    
    const processedData: ProcessedData = processUserData(userData)
    ```

F.2. Nested Component Structure for Complex Components
When a component has multiple distinct sections, create nested folder structure:

```
dashboard-welcome/
├── dashboard-welcome.tsx           // Main orchestrator component  
├── greeting.tsx                    // Self-contained greeting section
├── whats-included/                 // Folder for multi-part section
│   ├── whats-included.tsx         // Section orchestrator
│   ├── whats-included-title.tsx   // Title sub-component  
│   └── whats-included-features.tsx // Features list sub-component
├── core-technologies/              // Folder for multi-part section
│   ├── core-technologies.tsx      // Section orchestrator
│   ├── core-technologies-title.tsx // Title sub-component
│   └── core-technologies-list.tsx  // Tech list sub-component
└── get-started/                    // Folder for multi-part section
    ├── get-started.tsx             // Section orchestrator  
    ├── get-started-title.tsx       // Title sub-component
    ├── get-started-features.tsx    // Features grid sub-component
    └── get-started-feature-2.tsx   // Individual feature card
```

F.3. Component Organization Rules
1. **Main orchestrator**: Composition only, minimal logic, imports and renders sub-components
2. **Section orchestrators**: Handle section-specific logic, render related sub-components  
3. **Leaf components**: Single responsibility, pure presentation, accept props only
4. **Shared constants**: Extract to file level, use proper naming conventions
5. **Pure functions**: Extract above component definitions, properly typed
6. **File structure**: Mirror logical component hierarchy in folder structure

</coding_format>

<usage_guidelines>

1. Apply these standards to all new code and when refactoring existing code.
2. When making any code changes, ensure they conform to these guidelines.
3. If existing code doesn't follow these standards, update it to comply when modifying those files.
4. Use these standards as a checklist when reviewing code changes.
5. Prefer extracting helper functions over writing long, complex functions.
6. Always prioritize code readability and maintainability.

</usage_guidelines>

---
> Source: [sportiz91/vibe-template](https://github.com/sportiz91/vibe-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
