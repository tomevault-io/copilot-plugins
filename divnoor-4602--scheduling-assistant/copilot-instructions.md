## scheduling-assistant

> Zod Schema Validation for API Requests, Form Data, URL SearchParams, env variables, frontend / backend requests and data validation

# Zod Schema Validation Guidelines

## Core Principles

- ALWAYS validate runtime data with Zod - TypeScript types don't exist at runtime
- NEVER trust external data sources (APIs, user input, environment variables)
- ALWAYS use `z.infer<typeof Schema>` to derive TypeScript types from Zod schemas
- ALWAYS use `safeParse()` instead of `parse()` to handle errors gracefully
- DO define schemas first, then infer types - single source of truth

## Why Zod?

TypeScript provides compile-time type safety, but JavaScript runtime receives untyped data from APIs, forms, and external sources. Zod bridges this gap by validating data at runtime and providing both type safety and runtime guarantees. Without runtime validation, your app will crash when external data doesn't match expected types.

## Basic Schema Patterns

### Schema Definition and Type Inference
**Why**: Avoid duplicate type definitions. Schema acts as single source of truth for both runtime validation and TypeScript types.

```typescript
import { z } from "zod";

// ✅ GOOD: Define schema first, infer type
const UserSchema = z.object({
  id: z.number().positive(),
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().min(0).max(120).optional(),
});

type User = z.infer<typeof UserSchema>;

// ❌ BAD: Separate interface and schema
interface User { id: number; name: string; } // Duplication
const UserSchema = z.object({ id: z.number(), name: z.string() });
```

### Safe Parsing Pattern
**Why**: `parse()` throws errors and can crash your app. `safeParse()` returns a result object that lets you handle validation failures gracefully.

```typescript
// ✅ GOOD: Always use safeParse
const validateUser = (data: unknown) => {
  const result = UserSchema.safeParse(data);
  if (!result.success) {
    console.error("Validation failed:", result.error.issues);
    return null;
  }
  return result.data; // Fully typed User
};

// ❌ BAD: Using parse() can throw
const user = UserSchema.parse(unknownData); // Can crash your app
```

## API Validation

### Request/Response Validation
**Why**: External APIs can change their response format without notice. Validate responses to catch breaking changes early and ensure your app doesn't crash with malformed data.

```typescript
// API Response Schema
const ApiResponseSchema = z.object({
  success: z.boolean(),
  data: z.unknown().optional(),
  error: z.string().optional(),
});

// Validate API responses
const fetchData = async () => {
  const response = await fetch('/api/data');
  const rawData: unknown = await response.json();
  
  const result = ApiResponseSchema.safeParse(rawData);
  if (!result.success) {
    throw new Error('Invalid API response');
  }
  
  return result.data;
};
```

### React Query Integration
**Why**: React Query caches data, but cached data should be validated to ensure consistency. Validate in `queryFn` to catch API changes and provide type safety throughout your component tree.

```typescript
const useUserData = () => {
  return useQuery({
    queryKey: ['user'],
    queryFn: async () => {
      const response = await fetch('/api/user');
      const data: unknown = await response.json();
      
      const result = UserSchema.safeParse(data);
      if (!result.success) {
        throw new Error('Invalid user data');
      }
      
      return result.data;
    },
  });
};
```

## Form Validation

### React Hook Form Integration
**Why**: Forms are the primary source of user input. Validate on both client and server to provide immediate feedback and prevent invalid data submission.

```typescript
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';

const FormSchema = z.object({
  username: z.string().min(3, "Username must be at least 3 characters"),
  email: z.string().email("Invalid email format"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

type FormData = z.infer<typeof FormSchema>;

const MyForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(FormSchema),
  });

  const onSubmit = (data: FormData) => {
    // data is fully validated and typed
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('username')} />
      {errors.username && <span>{errors.username.message}</span>}
      
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

### Server Actions Validation
**Why**: Share schemas between client and server for consistent validation. Server-side validation is essential as client-side validation can be bypassed.

```typescript
// Shared schema between client and server
export const CreateUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

// Server action
export async function createUser(formData: FormData) {
  const rawData = Object.fromEntries(formData.entries());
  
  const result = CreateUserSchema.safeParse(rawData);
  if (!result.success) {
    return { 
      success: false, 
      errors: result.error.flatten().fieldErrors 
    };
  }
  
  // Use validated data
  const user = await saveUser(result.data);
  return { success: true, user };
}
```

### Advanced Form Validation
**Why**: Complex forms often require cross-field validation (password confirmation) or conditional validation (business vs individual accounts).

```typescript
// Cross-field validation
const PasswordFormSchema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string(),
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ["confirmPassword"],
});

// Conditional validation
const UserFormSchema = z.object({
  type: z.enum(["individual", "business"]),
  name: z.string().min(1),
  businessName: z.string().optional(),
}).refine(data => {
  if (data.type === "business") {
    return data.businessName && data.businessName.length > 0;
  }
  return true;
}, {
  message: "Business name is required for business accounts",
  path: ["businessName"],
});
```

## URL Parameters

### Search Params Validation
**Why**: URL parameters are strings by default. Use `z.coerce` to convert to proper types and provide defaults for better UX. Validation prevents crashes from malformed URLs.

```typescript
const SearchParamsSchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  search: z.string().optional(),
  category: z.enum(["all", "active", "archived"]).default("all"),
});

// Next.js usage
const Page = ({ searchParams }: { searchParams: Record<string, string> }) => {
  const result = SearchParamsSchema.safeParse(searchParams);
  if (!result.success) {
    // Handle invalid params - redirect or show error
    return <ErrorPage />;
  }
  
  const { page, limit, search, category } = result.data;
  // Use validated params
};
```

### Dynamic Route Params
**Why**: Route parameters come from URLs and can be malformed. Validate to ensure your API receives expected data types and handle invalid routes gracefully.

```typescript
const RouteParamsSchema = z.object({
  id: z.coerce.number().positive(),
  slug: z.string().min(1),
});

// API route
export async function GET(
  request: Request,
  { params }: { params: Record<string, string> }
) {
  const result = RouteParamsSchema.safeParse(params);
  if (!result.success) {
    return NextResponse.json({ error: "Invalid parameters" }, { status: 400 });
  }
  
  const { id, slug } = result.data;
  // Use validated params
}
```

## Backend Validation

### API Route Handlers
**Why**: Never trust client data. Always validate request bodies on the server to prevent malicious input and provide consistent error responses.

```typescript
// Shared schema
const CreatePostSchema = z.object({
  title: z.string().min(1),
  content: z.string().min(10),
  tags: z.array(z.string()).optional(),
});

// API route
export async function POST(request: Request) {
  const body: unknown = await request.json();
  
  const result = CreatePostSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json(
      { error: "Invalid data", issues: result.error.issues },
      { status: 400 }
    );
  }
  
  const post = await createPost(result.data);
  return NextResponse.json({ post });
}
```

### Environment Variables
**Why**: Environment variables are strings and may be missing or malformed. Validate at startup to fail fast and provide clear error messages during deployment.

```typescript
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  PORT: z.coerce.number().default(3000),
});

// Validate at startup
export const env = EnvSchema.parse(process.env);
```

### Webhook Validation
**Why**: Webhooks come from external services and can have malformed payloads. Validate to ensure your webhook handlers receive expected data structure.

```typescript
const WebhookSchema = z.object({
  event: z.string(),
  data: z.record(z.unknown()),
  timestamp: z.string().datetime(),
});

export async function POST(request: Request) {
  const body: unknown = await request.json();
  
  const result = WebhookSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json({ error: "Invalid webhook" }, { status: 400 });
  }
  
  await processWebhook(result.data);
  return NextResponse.json({ received: true });
}
```

## Common Patterns

### File Upload Validation
**Why**: File uploads need size and type validation for security and performance. Validate file properties before processing to prevent abuse.

```typescript
const FileUploadSchema = z.object({
  file: z.instanceof(File)
    .refine(file => file.size <= 5_000_000, "File too large")
    .refine(file => ["image/jpeg", "image/png"].includes(file.type), "Invalid file type"),
  description: z.string().optional(),
});
```

### Array and Object Validation
**Why**: Complex data structures need validation at multiple levels. Validate array contents, object properties, and nested structures for complete data integrity.

```typescript
// Array validation
const TagsSchema = z.array(z.string().min(1)).min(1).max(10);

// Record validation
const MetadataSchema = z.record(z.string(), z.unknown());

// Nested object validation
const AddressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string().regex(/^\d{5}$/),
});

const UserWithAddressSchema = z.object({
  name: z.string(),
  address: AddressSchema,
});
```

### Schema Composition
**Why**: Reuse schema definitions to maintain consistency. Compose schemas for different use cases (public vs private data, create vs update operations).

```typescript
// Base schemas
const BaseUserSchema = z.object({
  id: z.number(),
  name: z.string(),
});

// Extended schemas
const PublicUserSchema = BaseUserSchema.pick({ id: true, name: true });
const PrivateUserSchema = BaseUserSchema.extend({
  email: z.string().email(),
  role: z.enum(["user", "admin"]),
});

// Partial schemas
const UpdateUserSchema = BaseUserSchema.partial();
```

## Error Handling

### Custom Error Messages
**Why**: Default Zod error messages may not be user-friendly. Provide clear, actionable error messages for better user experience.

```typescript
const UserSchema = z.object({
  age: z.number({
    required_error: "Age is required",
    invalid_type_error: "Age must be a number",
  }).min(18, "Must be at least 18 years old"),
});
```

### Error Formatting
**Why**: Zod errors have a specific structure. Format them consistently for your UI components and API responses.

```typescript
const formatZodError = (error: z.ZodError) => {
  return error.issues.map(issue => ({
    field: issue.path.join('.'),
    message: issue.message,
  }));
};
```

## Performance Tips

- USE `.optional()` instead of `.nullable()` when possible - better performance
- AVOID deeply nested schemas for better performance - flatten when possible
- USE `.transform()` sparingly - prefer data transformation after validation
- CACHE schema instances when used frequently - avoid recreating schemas
- USE `.safeParse()` in production, `.parse()` only in development/testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Divnoor-4602)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Divnoor-4602)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
