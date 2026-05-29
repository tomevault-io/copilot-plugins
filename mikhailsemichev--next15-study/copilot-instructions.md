## typescript

> - All new files should use `.ts` extension for pure TypeScript files

# TypeScript Rules

## File Extensions
- All new files should use `.ts` extension for pure TypeScript files
- All React components should use `.tsx` extension
- Avoid using `.js` or `.jsx` extensions

## Type Declarations
- Always declare types for function parameters and return values
- Use interfaces for object types
- Use type for unions, intersections, and mapped types
- Avoid using `any` type unless absolutely necessary
- Use `unknown` instead of `any` when type is truly unknown

## React Components
- Use functional components with TypeScript
- Define prop types using interfaces
- Name interfaces with 'Props' suffix (e.g., `interface ButtonProps`)
- Use React.FC type sparingly (prefer explicit return types)

## State Management
- Define types for all state variables
- Use proper typing for useState hooks
- Define action types for reducers

## Event Handlers
- Use proper event types from React (e.g., `React.ChangeEvent<HTMLInputElement>`)
- Define callback function types explicitly

## API Responses
- Create interfaces for API response types
- Use generics for reusable API handlers

## Best Practices
- Enable strict mode in tsconfig.json
- Use type inference when types are obvious
- Document complex types with JSDoc comments
- Use readonly modifier for immutable properties
- Leverage union types for finite sets of values

## Example Component Structure
```tsx
interface ComponentProps {
  title: string;
  onAction: (id: number) => void;
  items?: string[];
}

export function Component({ title, onAction, items = [] }: ComponentProps) {
  const [value, setValue] = useState<string>('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };

  return (
    // JSX
  );
}
``` 

---
> Source: [MikhailSemichev/next15-study](https://github.com/MikhailSemichev/next15-study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
