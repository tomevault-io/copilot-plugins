## drawingboard-capital

> when working inside apps/frontend


# React TypeScript Coding Conventions

The following conventions MUST be followed when generating or modifying React components in TypeScript. Violating these rules is unacceptable.

---

## 1. COMPONENT DECLARATION

- Use `function` declarations for all React components.
- Do NOT use arrow functions (`const MyComponent = () => {}`).
- Do NOT use `React.FC` or `React.FunctionComponent`.

### Correct:

```ts
function Button(props: ButtonProps) {
  return <button>{props.label}</button>;
}
```

### Incorrect:

```ts
const Button: React.FC<ButtonProps> = (props) => { ... }
const Button = ({ label }: ButtonProps) => { ... }
```

---

## 2. PROP TYPING

- Use `interface` to define prop types.
- Do NOT use `type` for props unless strictly necessary (e.g., union types or mapped types).

### Correct:

```ts
interface ButtonProps {
  label: string;
}
```

### Incorrect:

```ts
type ButtonProps = {
  label: string;
};
```

---

## 3. EXPORT STYLE

- Always use `export default` for components.
- Do NOT use named exports for components.

### Correct:

```ts
export default function Header() {
  return <header />;
}
```

### Incorrect:

```ts
export function Header() { ... }
const Header = () => { ... };
export { Header };
```

---

## 4. CODE STYLE AND STRUCTURE

- Keep components flat and declarative.
- Separate styling and logic where practical.
- Keep prop defaults in function signatures.
- Format using Prettier conventions (e.g., single quotes, trailing commas, 2 spaces).
- Do NOT use overly abstract hooks, factories, or HOCs in demo/mocks.

---

## 5. NAMING

- Components should use PascalCase (e.g., `UserCard`, `LoginForm`).
- Prop interfaces should be suffixed with `Props` (e.g., `UserCardProps`).

---

## 6. DOCUMENTATION REQUIREMENTS

### 6.1 Component Documentation

- **MUST** include JSDoc comments above every component function.
- **MUST** include a brief description of what the component does.
- **MUST** document the component's primary use case or purpose.
- **SHOULD** include usage examples for complex components.
- **MUST** include storybook stories.

### Correct:

```ts
/**
 * A reusable button component that handles user interactions.
 * Supports various styles and sizes for different UI contexts.
 *
 * @example
 * <Button label="Click me" variant="primary" onClick={handleClick} />
 */
function Button(props: ButtonProps) {
  return <button>{props.label}</button>;
}
```

### 6.2 Props Interface Documentation

- **MUST** document every prop with JSDoc comments.
- **MUST** specify if a prop is optional and its default behavior.
- **MUST** include examples for complex prop types (unions, objects, functions).
- **SHOULD** specify acceptable values for string literals or enums.

### Correct:

```ts
interface ButtonProps {
  /** The text content displayed inside the button */
  label: string;

  /**
   * Visual style variant of the button
   * @default 'primary'
   * @example 'primary' | 'secondary' | 'danger'
   */
  variant?: 'primary' | 'secondary' | 'danger';

  /**
   * Click handler function
   * @param event - The mouse click event
   */
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;

  /**
   * Whether the button is disabled
   * @default false
   */
  disabled?: boolean;
}
```

### 6.3 Complex Logic Documentation

- **MUST** document any custom hooks with JSDoc.
- **MUST** document complex state logic or effects.
- **SHOULD** explain business logic or calculations.

### Correct:

```ts
/**
 * Custom hook for managing form validation state
 * @param initialValues - Initial form field values
 * @returns Object containing form state and validation methods
 */
function useFormValidation(initialValues: FormValues) {
  // Implementation...
}

function LoginForm() {
  // Complex state logic should be documented
  const [isSubmitting, setIsSubmitting] = useState(false);

  /**
   * Handles form submission with validation
   * Prevents multiple submissions and shows loading state
   */
  const handleSubmit = async (values: FormValues) => {
    // Implementation...
  };
}
```

### 6.4 Documentation Standards

- Use clear, concise language in documentation.
- Document the "why" not just the "what" for complex logic.
- Keep documentation up-to-date with code changes.
- Use proper JSDoc tags (`@param`, `@returns`, `@example`, `@default`).
- Do NOT document obvious or trivial code.
- Do NOT leave outdated documentation.

### 6.5 File-Level Documentation

- **SHOULD** include a file header comment for complex components.
- **SHOULD** document any external dependencies or side effects.

### Correct:

```ts
/**
 * @fileoverview User authentication components
 * Handles login, logout, and user session management
 *
 * @requires react-router-dom for navigation
 * @requires @/hooks/useAuth for authentication state
 */

import { useAuth } from '@/hooks/useAuth';
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CharviBansall) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
