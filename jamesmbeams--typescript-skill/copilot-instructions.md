## typescript-skill

> >


# TypeScript Best Practices — Matt Pocock Style

A comprehensive reference for writing high-quality TypeScript, compiled from Matt Pocock's publicly available teachings at [Total TypeScript](https://www.totaltypescript.com). Not affiliated with or endorsed by Matt Pocock or Total TypeScript.

---

## 1. `type` vs `interface`

**Default to `type`. Use `interface` only for object inheritance.**

### Why `type` by default:
- `type` can express unions, mapped types, conditional types — `interface` cannot
- Interfaces with the same name in the same scope **silently merge** (declaration merging), causing unexpected bugs
- Interfaces lack an implicit index signature, causing friction with `Record` types

### When to use `interface`:
- When you need `extends` for object inheritance — it's faster than `&` intersections
- TypeScript caches interfaces by name internally; intersection types are recomputed

```typescript
// ✅ Default: use type
type User = {
  id: string;
  name: string;
};

type StringOrNumber = string | number;

// ✅ Use interface for object inheritance (faster than &)
interface WithId {
  id: string;
}

interface User extends WithId {
  name: string;
}

// ❌ Avoid: intersection for inheritance (slower)
type User = WithId & {
  name: string;
};

// ❌ Danger: declaration merging surprise
interface Config {
  debug: boolean;
}
interface Config {  // silently merges!
  verbose: boolean;
}
```

---

## 2. Avoid Enums — Use Const Objects or Union Types

Enums are one of TypeScript's few non-type-level features. They emit JavaScript, have surprising behaviors, and are largely unnecessary.

### Problems with enums:
- Numeric enums allow unsafe reverse lookups
- They emit extra JavaScript (an IIFE)
- They can't be used with `satisfies`
- `const enum` has its own issues (inlining across module boundaries)

```typescript
// ❌ Avoid: enum
enum Status {
  Active = "active",
  Inactive = "inactive",
}

// ✅ Option 1: Union type (simplest)
type Status = "active" | "inactive";

// ✅ Option 2: Const object (when you need runtime values + type)
const Status = {
  Active: "active",
  Inactive: "inactive",
} as const;

type Status = (typeof Status)[keyof typeof Status];
// Result: "active" | "inactive"

// Usage — same as an enum but no emitted code surprises
function setStatus(status: Status) {}
setStatus(Status.Active); // works
setStatus("active");      // also works
```

**Rule:** For simple sets of strings, use a union type. When you need a named object for runtime access, use `as const` object + derived type.

---

## 3. The `satisfies` Operator

`satisfies` validates that a value conforms to a type **without widening it**. The value beats the type.

### When to use each:
- **Colon annotation (`:`)** — type beats value. Use for explicit typing, function parameters, variables that will be reassigned
- **`satisfies`** — value beats type. Use when you want validation AND narrow inference
- **`as`** — almost never. It lets you lie to TypeScript
- **No annotation** — let TypeScript infer. This is the default and often the best choice

```typescript
// ❌ Colon widens — loses key information
const routes: Record<string, {}> = {
  "/": {},
  "/users": {},
};
routes.anything; // no error — type is too wide

// ✅ satisfies: validates AND preserves narrow type
const routes = {
  "/": {},
  "/users": {},
} satisfies Record<string, {}>;

routes["/"];       // ✅ works
routes.anything;   // ❌ error — keys are known

// ✅ satisfies for config objects
type ColorConfig = Record<string, string | string[]>;

const palette = {
  red: "#ff0000",
  green: ["#00ff00", "#00ee00"],
} satisfies ColorConfig;

// TypeScript knows palette.red is string, palette.green is string[]
palette.green.map(g => g.toUpperCase()); // ✅ no assertion needed
```

### `as const satisfies` — the power combo

Combining `as const` with `satisfies` gives you **literal inference + type validation** in one expression. This is one of the most useful patterns in modern TypeScript.

```typescript
// ✅ as const satisfies — validates shape AND preserves literals
const routes = {
  home: "/",
  users: "/users",
  admin: "/admin",
} as const satisfies Record<string, string>;

// TypeScript knows: routes.home is "/", not string
// AND it validated that all values are strings

// ✅ Great for config with known shapes
const endpoints = {
  getUser: { method: "GET", path: "/users/:id" },
  createUser: { method: "POST", path: "/users" },
} as const satisfies Record<string, { method: string; path: string }>;

// endpoints.getUser.method is "GET", not string
// endpoints.createUser.path is "/users", not string

// ❌ Without as const — literals are widened
const endpoints = {
  getUser: { method: "GET", path: "/users/:id" },
} satisfies Record<string, { method: string; path: string }>;
// endpoints.getUser.method is string — literals lost

// ❌ Without satisfies — no shape validation
const endpoints = {
  getUser: { method: "GET", path: "/users/:id" },
} as const;
// Typos or missing fields won't be caught
```

**Rule:** Use `as const satisfies` when you want to lock down a value to literal types while also validating it conforms to a shape. Use `satisfies` alone when you don't need literal narrowing. Use `:` when you explicitly want the wider type.

---

## 4. TSConfig Best Practices

Based on Matt Pocock's TSConfig Cheat Sheet. Use `@total-typescript/tsconfig` for a batteries-included base.

### Quickstart with `@total-typescript/tsconfig`:

```jsonc
// Bundler + DOM (e.g. React/Vite)
{ "extends": "@total-typescript/tsconfig/bundler/dom" }

// Bundler + non-DOM (e.g. Node with bundler)
{ "extends": "@total-typescript/tsconfig/bundler/no-dom" }

// tsc + Node
{ "extends": "@total-typescript/tsconfig/tsc/node/dom" }
```

This sets all the options below for you. If you need to customize, here are the manual settings:

### Base options (every project):

```jsonc
{
  "compilerOptions": {
    "esModuleInterop": true,
    "skipLibCheck": true,
    "target": "es2022",
    "allowJs": true,
    "resolveJsonModule": true,
    "moduleDetection": "force",
    "isolatedModules": true,
    "verbatimModuleSyntax": true
  }
}
```

### Strictness (every project):

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true
  }
}
```

**`noUncheckedIndexedAccess`** is critical — it makes array/object index access return `T | undefined`, preventing runtime errors. Matt considers it should be part of `strict`.

### If transpiling with tsc:

```jsonc
{
  "compilerOptions": {
    "module": "NodeNext",
    "outDir": "dist",
    "sourceMap": true
  }
}
```

### If NOT transpiling (using bundler):

```jsonc
{
  "compilerOptions": {
    "module": "preserve",
    "noEmit": true
  }
}
```

### DOM vs non-DOM:

```jsonc
// DOM (browser/frontend)
{ "lib": ["es2022", "dom", "dom.iterable"] }

// Non-DOM (Node, serverless)
{ "lib": ["es2022"] }
```

### Library in monorepo — add:
```jsonc
{
  "declaration": true,
  "composite": true,
  "declarationMap": true,
  "sourceMap": true
}
```

**Avoid** noisy options like `noImplicitReturns`, `noUnusedLocals`, `noUnusedParameters` in tsconfig — use ESLint for those instead.

---

## 5. Return Type Annotations

**Don't annotate return types by default. Let TypeScript infer.**

### When to annotate return types:
1. **Public API boundaries** (library exports) — prevents accidental API changes
2. **Recursive functions** — TypeScript sometimes needs help
3. **When inference produces an unwieldy type** — simplify for readability
4. **When you want to enforce a contract** — the function must return this shape

```typescript
// ✅ Let TypeScript infer (most of the time)
function add(a: number, b: number) {
  return a + b; // inferred: number
}

// ✅ Annotate public API / library exports
export function createUser(name: string): User {
  return { id: crypto.randomUUID(), name };
}

// ✅ Annotate when inference is complex/ugly
function parseConfig(raw: string): AppConfig {
  // ... complex parsing
}
```

**Rule:** Annotate inputs, infer outputs — unless it's a public API boundary.

---

## 6. Generics — Patterns & Constraints

### When to use generics:
- When you need to **relate** input types to output types
- When a function should work across multiple types while preserving type information

### Key patterns:

```typescript
// ✅ Constrain with extends
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// ✅ Use defaults for common cases
function createState<T = string>(initial: T) {
  return { value: initial };
}

// ✅ Multiple constraints with intersection
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}

// ✅ Infer from usage — don't make users specify generics manually
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
  return arr.map(fn);
}
// Users just call map([1,2,3], n => n.toString()) — T and U are inferred

// ❌ Avoid: unnecessary generics (the "useless generic")
function bad<T>(x: T): void { } // T isn't used meaningfully

// ✅ Rule of thumb: if a generic appears only once, you don't need it
function good(x: unknown): void { }
```

### `NoInfer<T>` — control inference sites (TS 5.4+)

`NoInfer` prevents a generic parameter from being inferred at a specific position. This forces TypeScript to infer the type from other usage sites.

```typescript
// ❌ Without NoInfer — TypeScript infers T from both arguments, causing a union
function createFSM<T extends string>(initial: T, states: T[]) {}
createFSM("idle", ["idle", "running", "stopped"]);
// T is inferred as "idle" | "running" | "stopped" — initial isn't validated

// ✅ With NoInfer — T is inferred only from `states`, so `initial` is checked against it
function createFSM<T extends string>(initial: NoInfer<T>, states: T[]) {}
createFSM("idle", ["idle", "running", "stopped"]); // ✅
createFSM("invalid", ["idle", "running", "stopped"]); // ❌ error

// ✅ Useful for default values
function getOrDefault<T>(values: T[], fallback: NoInfer<T>): T {
  return values[0] ?? fallback;
}
getOrDefault([1, 2, 3], 0);      // ✅
getOrDefault([1, 2, 3], "nope");  // ❌ error — string is not number
```

### Assign to local type variables for performance:

```typescript
// ✅ Cache complex types in generic slots
type Result<T> = {
  data: T;
  error: null;
} | {
  data: null;
  error: Error;
};
```

---

## 7. Discriminated Unions

The most powerful pattern for modeling state in TypeScript.

```typescript
// ✅ Discriminated union with literal discriminant
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: Error };

function handle(result: Result<User>) {
  if (result.success) {
    console.log(result.data); // ✅ narrowed to { success: true; data: User }
  } else {
    console.error(result.error); // ✅ narrowed to { success: false; error: Error }
  }
}

// ✅ Model state machines
type AuthState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "authenticated"; user: User }
  | { status: "error"; error: string };

// ❌ Avoid: optional properties for mutually exclusive state
type BadAuthState = {
  status: string;
  user?: User;    // when is this defined? unclear
  error?: string; // when is this defined? unclear
};
```

**Rule:** When properties are mutually exclusive or depend on a state, use a discriminated union — never optional properties.

---

## 8. Branded / Nominal Types

TypeScript's type system is structural. Use branded types when you need nominal-like behavior.

```typescript
// ✅ Brand pattern
type UserId = string & { readonly __brand: unique symbol };
type OrderId = string & { readonly __brand: unique symbol };

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId) { /* ... */ }

const userId = createUserId("user_123");
const orderId = "order_456" as OrderId;

getUser(userId);   // ✅
getUser(orderId);  // ❌ Type error — OrderId is not UserId
getUser("raw");    // ❌ Type error — string is not UserId

// ✅ Simpler brand helper
declare const __brand: unique symbol;
type Brand<T, B> = T & { [__brand]: B };

type Email = Brand<string, "Email">;
type URL = Brand<string, "URL">;
```

---

## 9. Template Literal Types

Powerful for string manipulation at the type level.

```typescript
// ✅ Build string types
type EventName = `on${Capitalize<string>}`;
// "onClick", "onHover", etc.

// ✅ Extract parts of strings
type ExtractRouteParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<Rest>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractRouteParams<"/users/:id/posts/:postId">;
// "id" | "postId"

// ✅ Autocomplete helper — allows arbitrary strings but suggests known ones
type Autocomplete<T extends string> = T | (string & {});

type Color = Autocomplete<"red" | "blue" | "green">;
// Suggests "red", "blue", "green" but accepts any string
```

---

## 10. Utility Types — When to Use Them

```typescript
// ✅ Pick / Omit for subsetting
type UserPreview = Pick<User, "id" | "name">;
type UserWithoutEmail = Omit<User, "email">;

// ✅ Partial for optional updates
function updateUser(id: string, updates: Partial<User>) {}

// ✅ Required to make all props required
type CompleteForm = Required<FormData>;

// ✅ Record for dictionaries
type UserMap = Record<string, User>;

// ✅ Extract / Exclude for union manipulation
type SuccessStates = Extract<AuthState, { status: "authenticated" | "idle" }>;
type ErrorStates = Exclude<AuthState, { status: "authenticated" }>;

// ✅ ReturnType / Parameters for function types
type ApiResponse = ReturnType<typeof fetchUser>;
type ApiArgs = Parameters<typeof fetchUser>;

// ✅ Awaited for unwrapping promises
type UserData = Awaited<ReturnType<typeof fetchUser>>;

// ✅ NonNullable to strip null/undefined
type DefinitelyUser = NonNullable<User | null | undefined>;

// ✅ Readonly for immutable data
type FrozenConfig = Readonly<Config>;
// Deep readonly: use as const or a DeepReadonly utility
```

---

## 11. Method Signatures — Function Properties vs Method Shorthand

**Prefer function property syntax over method shorthand in interfaces/types.**

```typescript
// ✅ Function property — stricter, works with strict function types
type Logger = {
  log: (message: string) => void;
  warn: (message: string) => void;
};

// ❌ Method shorthand — bivariant (less safe) parameter checking
type Logger = {
  log(message: string): void;
  warn(message: string): void;
};
```

Method shorthand allows **bivariant** parameter checking (both covariant and contravariant), which can hide type errors. Function properties enforce **strict** (contravariant) parameter checking when `strictFunctionTypes` is on.

---

## 12. Module Augmentation & `declare global`

```typescript
// ✅ Augment existing modules
declare module "express" {
  interface Request {
    user?: User;
  }
}

// ✅ Add global types (use sparingly)
declare global {
  interface Window {
    analytics: AnalyticsLib;
  }
}

// ✅ Force file to be a module (needed for declare global)
export {};
```

**Note:** This is one of the legitimate uses of `interface` — declaration merging is the feature, not a bug, here.

---

## 13. `as const` — Extracting Literal Types

```typescript
// ✅ as const for literal inference
const ROLES = ["admin", "user", "guest"] as const;
type Role = (typeof ROLES)[number]; // "admin" | "user" | "guest"

// ✅ Object with as const
const HTTP_STATUS = {
  OK: 200,
  NOT_FOUND: 404,
  SERVER_ERROR: 500,
} as const;

type StatusCode = (typeof HTTP_STATUS)[keyof typeof HTTP_STATUS];
// 200 | 404 | 500
```

---

## 14. Type Predicates & Assertion Functions

```typescript
// ✅ Type predicate for narrowing
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value
  );
}

// ✅ Assertion function
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error("Not a user");
  }
}

// ✅ Filter with type predicates
const users = items.filter((item): item is User => isUser(item));
```

---

## 15. Common Pitfalls & Anti-Patterns

### ❌ Don't use `any` — use `unknown`
```typescript
// ❌ any disables all type checking
function parse(input: any) { return input.foo.bar; }

// ✅ unknown forces you to narrow first
function parse(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    // now it's safe
  }
}
```

### ❌ Don't use `as` for everyday typing
```typescript
// ❌ Lying to TypeScript
const user = {} as User;

// ✅ Actually construct the object properly
const user: User = { id: "1", name: "Matt" };
```

### ❌ Don't use `Function` type
```typescript
// ❌ Too loose
const fn: Function = () => {};

// ✅ Be specific
const fn: () => void = () => {};
const fn: (...args: unknown[]) => unknown = () => {};
```

### Understand the `{}` type
```typescript
// ⚠️ {} means "any non-nullish value" — not "an empty object"
const obj: {} = "string"; // this is valid!
const num: {} = 42;       // also valid!

// ✅ {} IS useful as a constraint meaning "non-nullish"
function nonNullable<T extends {}>(value: T): T { return value; }

// ❌ Don't use {} when you mean "an object with properties"
// ✅ Use Record<string, unknown> for generic objects
const obj: Record<string, unknown> = { key: "value" };
```

### ❌ Don't use non-null assertion (`!`) casually
```typescript
// ❌ Hiding potential null errors
const el = document.getElementById("app")!;

// ✅ Handle the null case
const el = document.getElementById("app");
if (!el) throw new Error("Element not found");
```

### ❌ Don't use `object` type
```typescript
// ❌ Too vague — matches any non-primitive
function process(obj: object) {}

// ✅ Be specific about the shape
function process(obj: { id: string }) {}
// or use a generic
function process<T extends Record<string, unknown>>(obj: T) {}
```

---

## 16. Performance Tips

### Prefer interfaces for heavily-reused object shapes in extends chains
TypeScript caches interfaces by name; intersection types are recomputed.

### Avoid deep type instantiation
```typescript
// ❌ Recursive types that go too deep
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
// Fine for moderate depth, but can cause "Type instantiation is excessively deep" on complex objects

// ✅ Limit recursion depth or use a library (ts-toolbelt)
```

### Use `skipLibCheck: true`
Always. Checking node_modules `.d.ts` files is slow and rarely catches real bugs.

### Assign generics to type variables
Instead of repeating complex generic expressions, assign them to intermediate types.

---

## 17. `infer` in Conditional Types

The `infer` keyword lets you **extract types** from within other types. It's the backbone of many advanced patterns.

```typescript
// ✅ Extract return type manually (this is how ReturnType works)
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Result = MyReturnType<() => string>; // string

// ✅ Extract array element type
type ElementOf<T> = T extends (infer E)[] ? E : never;

type Item = ElementOf<string[]>; // string

// ✅ Extract promise value
type Unwrap<T> = T extends Promise<infer V> ? V : T;

type Data = Unwrap<Promise<User>>; // User
type Plain = Unwrap<string>;       // string (passthrough)

// ✅ Extract first argument of a function
type FirstArg<T> = T extends (first: infer A, ...rest: any[]) => any ? A : never;

type Arg = FirstArg<(name: string, age: number) => void>; // string

// ✅ Combine with template literals (see section 9)
type ExtractRouteParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<Rest>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

// ✅ infer with constraints (TS 5.0+)
type GetString<T> = T extends [infer S extends string, ...unknown[]] ? S : never;

type First = GetString<["hello", 42]>; // "hello"
type Nope = GetString<[42, "hello"]>;  // never (42 doesn't extend string)
```

**Rule:** Use `infer` when you need to extract a type that's buried inside another type. If a generic appears only in the `infer` position, it doesn't need to be declared as a type parameter.

---

## 18. Runtime Validation at System Boundaries (Zod)

TypeScript types disappear at runtime. At **system boundaries** (API responses, user input, env vars, file reads), you need runtime validation. Matt Pocock recommends Zod as the standard approach.

```typescript
import { z } from "zod";

// ✅ Define schema — single source of truth for runtime + type
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
  role: z.enum(["admin", "user", "guest"]),
});

// ✅ Derive the TypeScript type from the schema
type User = z.infer<typeof UserSchema>;
// { id: string; name: string; email: string; role: "admin" | "user" | "guest" }

// ✅ Validate at API boundaries
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const data: unknown = await response.json();
  return UserSchema.parse(data); // throws ZodError if invalid
}

// ✅ Safe parse for error handling without exceptions
const result = UserSchema.safeParse(data);
if (result.success) {
  console.log(result.data); // fully typed User
} else {
  console.error(result.error.issues);
}

// ✅ Validate environment variables
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(["development", "production", "test"]),
});

export const env = EnvSchema.parse(process.env);

// ✅ Compose schemas
const CreateUserSchema = UserSchema.omit({ id: true });
const UpdateUserSchema = UserSchema.partial().required({ id: true });
```

**Rule:** Never trust data from outside your program's boundary. Define Zod schemas at entry points (API handlers, CLI args, env vars) and derive TypeScript types from them — not the other way around. The schema is the source of truth.

---

## Quick Reference — Decision Table

| Situation | Do This |
|---|---|
| Declaring an object type | `type X = { ... }` |
| Object inheritance | `interface X extends Y` |
| Union or intersection type | `type X = A \| B` |
| Set of string constants | `type X = "a" \| "b"` or `as const` object |
| Enum-like with runtime values | `as const` object + derived type |
| Config object validation | `satisfies` |
| Function return type | Let TypeScript infer (usually) |
| Public library export | Annotate return type explicitly |
| Mutually exclusive state | Discriminated union |
| Preventing type confusion | Branded types |
| Generic parameter used once | Remove it, use `unknown` |
| Typing callbacks/functions | Function property syntax `fn: () => void` |
| `any` needed | Use `unknown` + narrowing instead |
| Module extension | `declare module` with interface merging |
| Config with literal types + validation | `as const satisfies Type` |
| Extracting types from other types | Conditional type with `infer` |
| Controlling generic inference site | `NoInfer<T>` |
| External data (API, env, user input) | Zod schema → derive type with `z.infer` |
| `{}` type needed | OK for "non-nullish" constraint; use `Record<string, unknown>` for objects |

---
> Source: [jamesmbeams/typescript-skill](https://github.com/jamesmbeams/typescript-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
