## metrolink-mcp

> This rule guides how to build forms using NextJS and React Hook Form


# Form Handling & Validation Guide

## Form Handling with React Hook Form

### Mandatory Usage

- **ONLY use React Hook Form** for all form components
- **NEVER** hook into events individually (no direct `onChange`, `onSubmit`, `onBlur` handlers on form inputs)
- All form state management must go through React Hook Form's `useForm` hook

### Basic Form Pattern

- Use `useForm` hook from `react-hook-form`
- Define form schema/types for type safety
- Use `register` to connect inputs to the form
- Use `handleSubmit` for form submission
- Example:
  ```tsx
  import { useForm } from 'react-hook-form';
  import type { FC } from 'react';
  
  interface FormData {
    email: string;
    password: string;
  }
  
  interface LoginFormProps {
    onSubmit: (data: FormData) => void;
  }
  
  const LoginForm: FC<LoginFormProps> = ({ onSubmit }) => {
    const { register, handleSubmit, formState: { errors } } = useForm<FormData>();
  
    return (
      <form onSubmit={handleSubmit(onSubmit)}>
        <input
          {...register('email', { required: 'Email is required' })}
          type="email"
        />
        {errors.email && <span>{errors.email.message}</span>}
  
        <input
          {...register('password', { required: 'Password is required' })}
          type="password"
        />
        {errors.password && <span>{errors.password.message}</span>}
  
        <button type="submit">Submit</button>
      </form>
    );
  };
  
  export default LoginForm;
  ```

### Form Validation

- Use React Hook Form's built-in validation rules
- For complex validation, use schema validation libraries (e.g., Zod, Yup) with `@hookform/resolvers`
- **Always use `z.infer` to infer form data types from Zod schemas** (never manually define types)
- Example with Zod:
  ```tsx
  import { useForm } from 'react-hook-form';
  import { zodResolver } from '@hookform/resolvers/zod';
  import { z } from 'zod';
  
  const formSchema = z.object({
    email: z.string().email('Invalid email address'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
  });
  
  // Infer type from schema
  type FormData = z.infer<typeof formSchema>;
  
  const LoginForm: FC<LoginFormProps> = ({ onSubmit }) => {
    const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
      resolver: zodResolver(formSchema),
    });
  
    // ... rest of component
  };
  ```

### Form Field Registration

- Always use the spread operator with `register()`: `{...register('fieldName')}`
- Never manually manage `value`, `onChange`, or `onBlur` props
- For controlled components, use `Controller` or `useController` from React Hook Form

### Form Submission

- Always use `handleSubmit` wrapper for form submission
- `handleSubmit` takes two callbacks: `onValid` and `onInvalid` (optional)
- Example:
  ```tsx
  const onSubmit = (data: FormData) => {
    // Handle valid form submission
    console.log(data);
  };
  
  const onError = (errors: FieldErrors<FormData>) => {
    // Handle validation errors
    console.error(errors);
  };
  
  <form onSubmit={handleSubmit(onSubmit, onError)}>
    {/* form fields */}
  </form>
  ```

### Prohibited Patterns

- ❌ **DO NOT** use direct event handlers:
  ```tsx
  // WRONG - Never do this
  <input
    value={email}
    onChange={(e) => setEmail(e.target.value)}
    onBlur={handleBlur}
  />
  ```

- ❌ **DO NOT** manually manage form state:
  ```tsx
  // WRONG - Never do this
  const [formData, setFormData] = useState({ email: '', password: '' });
  ```

- ✅ **DO** use React Hook Form:
  ```tsx
  // CORRECT
  <input {...register('email')} />
  ```

### Advanced Patterns

- Use `Controller` component for complex inputs (date pickers, rich text editors, etc.)
- Use `useWatch` for watching specific field values
- Use `setValue`, `getValues`, `reset` for programmatic form control
- Use `formState` for accessing form state (isDirty, isValid, isSubmitting, etc.)

## Zod Type Inference

### Prefer Zod-Inferred Types

- **ALWAYS** use `z.infer<typeof schema>` to infer types from Zod schemas
- Never manually define types that duplicate Zod schema definitions
- This ensures type safety and keeps types in sync with validation schemas
- Example:
  ```ts
  import { z } from 'zod';
  
  // Define schema
  const userSchema = z.object({
    email: z.string().email(),
    name: z.string().min(1),
    age: z.number().int().positive(),
  });
  
  // ✅ CORRECT - Infer type from schema
  type User = z.infer<typeof userSchema>;
  
  // ❌ WRONG - Don't manually define types
  // type User = {
  //   email: string;
  //   name: string;
  //   age: number;
  // };
  ```

### Type Inference Best Practices

- Use inferred types for function parameters, return types, and component props
- Export inferred types alongside schemas for reuse
- Example:
  ```ts
  const createUserSchema = z.object({
    email: z.string().email('Invalid email address'),
    name: z.string().min(1, 'Name is required'),
    age: z.number().int().positive('Age must be a positive integer'),
  });
  
  // Export both schema and inferred type
  export { createUserSchema };
  export type CreateUserInput = z.infer<typeof createUserSchema>;
  ```

## Server Action Validation

### Zod Validation for Parameters

- **ALWAYS** validate Server Action parameters using Zod schemas
- Define Zod schema for action parameters
- **Use `z.infer` to infer types from schemas** (never manually define types)
- Parse and validate parameters at the start of the action
- Return typed, validated data
- Example:
  ```tsx
  'use server';
  
  import { z } from 'zod';
  
  const createUserSchema = z.object({
    email: z.string().email('Invalid email address'),
    name: z.string().min(1, 'Name is required'),
    age: z.number().int().positive('Age must be a positive integer'),
  });
  
  // Infer type from schema
  type CreateUserInput = z.infer<typeof createUserSchema>;
  
  export const createUser = async (params: unknown) => {
    // Validate parameters
    const validatedParams = createUserSchema.parse(params);
  
    // Use validatedParams (now type-safe)
    const { email, name, age } = validatedParams;
  
    // Server action implementation
    // ...
  };
  ```

### Error Handling in Server Actions

- Use Zod's `safeParse` for error handling when needed
- Return consistent error response format
- Example:
  ```tsx
  export const createUser = async (params: unknown) => {
    const result = createUserSchema.safeParse(params);
  
    if (!result.success) {
      return {
        success: false,
        error: result.error.format(),
      };
    }
  
    const { email, name, age } = result.data;
    // ... implementation
    return { success: true, data: user };
  };
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimdrury) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
