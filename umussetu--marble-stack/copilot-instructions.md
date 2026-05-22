## marble-stack

> This is a Vite Remix project in React using Typescript, Ant Design and Tailwind and tRPC React query.

# Frontend Documentation

This is a Vite Remix project in React using Typescript, Ant Design and Tailwind and tRPC React query.

## Pages

Pages located at "/app/routes/\_logged.**.tsx" are accessible for logged users.
Other pages located at "/app/routes/**.tsx" are public.

# Authorization Policies on Server

Policies are defined using zenstack on zmodels "/models/models.zmodel".
It automatically protect auto-generated CRUD models and the default Prisma client.

You can use the `@@allow` decorator to define policies.

Example for a model "Foo" at "/models/models.zmodel":

```zmodel
model Foo {
  id               String    @id @default(uuid())
  name             String?
  email            String?
  isVerified       Boolean?

  @@allow('all', auth().roles?[name == 'admin'])
  @@allow('read', true)
  @@allow('create', auth().id == this.id)
  @@allow('update', auth().id == this.id)
  @@allow('delete', auth().id == this.id && this.isVerified)
}
```

Explanations:

- `auth()` is the authenticated user, you have access to the same properties as the user model.
- `this` is the current model (here Foo)
- `@@allow('read', [condition])` grant read policy if the condition is true. It will also affect how findMany results are filtered.
- `@@allow('update', [condition])` grant update policy if the condition is true.
- `@@allow('delete', [condition])` grant delete policy if the condition is true.
- `@@allow('all', [condition])` grant all policies if the condition is true
- `auth().roles?[name == 'admin']` - you can access relations. In this example, we checking if a user, has a role with the name 'admin'

# Core Utilities

Core utilities, hooks and business logic are located at "/app/core"

- "app/core/context" is where the user context store is setup. It can be used in the application to access information about the logged user.
- "app/core/helpers" is where small utilities to manipulate dates or typescript operations are stored.
- "app/core/hooks" contains hooks for external libraries and API routes that are not using tRPC.

## Sensitive files

- "app/core/authentication" is where the authentication is setup.
- "app/core/configuration" is where the common environment variables are provided.
- "app/core/database" is where the Prisma database is setup for the backend.
- "app/core/trpc" is where the trpc client and server are setup.

Do not updates these files except if asked to do so.

# Design System

The project uses Ant Design for all UI components. On top of that, you can use tailwind className to style and position the UI components.

The design system is setup in the "/app/designSystem" folder.

The following folders and files can safely be updated:

- "/app/designSystem/layouts" contains re-usable layouts components for pages.
- "/app/designSystem/theme/theme.tsx" is where the Ant Design theme can be customised.
- "/app/designSystem/ui" is where custom components that are not part of Ant Design but useful for the whole application are stored.

## Sensitive files

Except if explicitly asked, you must avoid touching the following folders and files:

- "/app/designSystem/core" contains top level components such as HTML and main tags.
- "/app/designSystem/provider.tsx" is where we apply all design system providers.
- "/app/designSystem/style" contains global scss files.

[CHUNK-SEPARATOR]

## How to create a UI component

1. Create your component in "/app/designSystem/ui/[ComponentName]/index.tsx"
2. Code the component

```tsx
// Typical example

type Props = {
  key1: string
  key2: string
  key3: string
}

export const ComponentName: React.FC<Props> = ({ key1, key2, key3 }) => {
  return <>...</>
}
```

Tips:

- We usually prefer to export the `const` directly so the component has a consistent import in other files `import { ComponentName } from '@/designSystem/ui/ComponentName'`.
- You can use tailwind className to style the UI components.
- To import Title, Text and Paragraph from Ant Design, you need to import \`import { Typography } from 'antd'\` and then do \`const { Title, Text, Paragraph} = Typography\`. There are no other ways.

## Session

The context available from `import { AuthenticationServer } from '@/core/authentication/server'` is an object with the following structure:

```ts
type Context = {
  database: PrismaClient // The default prisma client to use, recommended as there are policies triggered in case the action is not permitted for the user.
  databaseUnprotected: PrismaClient // The prisma client without policies, useful when you don't need the policies.
  session: {
    user?: {
      id: string
      email: string
      name: string
      globalRole: string
    }
  }
}

const context = await AuthenticationServer.getHttpContext({ request })

console.log(context.session.user)
```

## Database Interaction

```ts
const context = await AuthenticationServer.getHttpContext({ request })

const userFetched = await context.database.user.findUnique({
  data: { id: context.session.user.id },
})

const filteredUsers = await context.database.user.findMany() // This automatically apply authorization policies to filter the results

const allUsers = await context.databaseUnprotected.user.findMany() // This bypass all the authorization policies
```

# How to create a model

1. Create a model in the file "models/models.zmodel"
2. Define the model using the Prisma syntax (you can look at an existing model to view an example)

[CHUNK-SEPARATOR]

# How to update a Model

When editing a model, any new field must either be optional or have a default value. Failing to do so will cause the app to crash.

```zmodel
model [Name] {
  ...
  fieldOptional             String?
  fieldWithDefaultValue     String   @default("the default value")
  ...
}
```

[CHUNK-SEPARATOR]

## How to add a OneToMany relation

```zmodel

model Parent {
  ...
  children Child[] @relation("parent")
  ...
}
```

```zmodel

model Child {
  ...
  parentId? String
  parent?   Parent @relation(fields: [parentId], references: [id], name: "parent")
  ...
}
```

Constraints:

- It's important that the name:"parent" in the Child matches the @relation("parent") name
- When adding a relation, make sure to add fields on both sides of the relations

## Enums

```zmodel
enum NameEnum{
  VALUE1
  VALUE2
  VALUE3
}
```

# Models

We are using Zenstack (a Prisma superset) and all the models are located in "/models/models.zmodel".

Do not edit the .prisma, only edit the models.zmodel file to edit the data models.

Zenstack generates also all the tRPC endpoints/routers from the models.

[CHUNK-SEPARATOR]

## Using Prisma Types with React State and tRPC

To work with Prisma models in React, import necessary types from @prisma/client. Use Prisma.[name]GetPayload for accurate typing when state involves relations.

```tsx
// Import Prisma types
import { Prisma, User } from '@prisma/client'
import { useState } from 'react'

// Correct typing with relations
const [value, setValue] = useState<Prisma.UserGetPayload<{ include: { relation1: true; relation2: true } }>>()

// tRPC functions have correct typing by default
const { data } = Api.user.findMany.useQuery({ include: { relation1: true; relation2: true } })
```

Always use Prisma.[name]GetPayload to avoid TypeScript errors when including relations.

# Navigation Layout - Important Information

## Navigation Links

- **File:** `app/designSystem/layouts/NavigationLayout/index.tsx`
- All navigation links for the topbar and leftbar are defined here in variables.
- **AI Instruction:** Only modify this file when adding or displaying a new link for the top or left bar.

## Layout Components

- **Folder:** `app/designSystem/layouts/NavigationLayout/`
- Contains the components for layout elements like the **topbar**, **leftbar**, etc.
- **AI Instruction:** For any changes to the layout's structure or appearance, refer to the components inside this folder, not the navigation tabs.

# tRPC & Zenstack

Zenstack automatically auto-generates routers for all models, meaning **there is no need to create these routers when you add or edit a data model, they already exist**.

---

## Common Operations

Zenstack provides model-level endpoint/functions that handle the following operations:

- `findUnique`
- `findFirst`
- `findMany`
- `create`
- `createMany`
- `update`
- `updateMany`
- `delete`
- `deleteMany`

These CRUD routers are **auto-generated** from your models and available in both the server and frontend contexts.

### Frontend: tRPC Client

To interact with the backend, use the pre-built tRPC / React Query client with `@/core/trpc`. No need to manually create routers for CRUD operations. Instead, import the `Api` object that exposes all necessary hooks.

Here’s how to perform common CRUD operations:

#### Example - Fetch a User

```tsx
import { Api } from '@/core/trpc'

const userId = '667'
const {
  data: user,
  isLoading,
  refetch,
} = Api.user.findFirst.useQuery({ where: { id: userId } })
```

#### Example - Fetch many users

```tsx
import { Api } from '@/core/trpc'

const { data: users, isLoading, refetch } = Api.user.findMany.useQuery()
```

#### Example - Create a User

```tsx
import { Api } from '@/core/trpc'

const { mutateAsync: createUser } = Api.user.create.useMutation()

const handleCreate = async () => {
  await createUser({ data: { email: 'email', name: 'test' } })
}
```

#### Example - Update a User

```tsx
import { Api } from '@/core/trpc'

const userId = '667'
const { mutateAsync: updateUser } = Api.user.update.useMutation()

const handleUpdate = async () => {
  await updateUser({ where: { id: userId }, data: { email: 'new_email' } })
}
```

#### Example - Delete a User

```tsx
import { Api } from '@/core/trpc'

const userId = '667'
const { mutateAsync: deleteUser } = Api.user.delete.useMutation()

const handleDelete = async () => {
  await deleteUser({ where: { id: userId } })
}
```

#### Refetch

Information about `refetch` method:

- Calling the refetch method from the useQuery client will automatically update the data object
- The refetch method does not take any argument
- The refetch method does not return anything

#### Fetch a specific direclty from a function

- Use the `Api.useUtils()` hook with its provided `fetch` method
- You must declare the `Api.useUtils()` hook at the root of the component and use it inside the function

Example:

```tsx
import { Api } from '@/core/trpc'

const apiUtils = Api.useUtils()

const handleProjectClicked = async (projectId: string) => {
  const project = await apiUtils.project.findUnique.fetch({
    where: { id: projectId },
  })
  // Use the project fetched inside the function (e.g: to redirect the user)
}
```

Important:

- All `useMutation` and `useQuery` hooks must be declared at the top level of the component body, not within nested functions or event handlers. Violating this rule will break the hooks' behavior.

---

## Custom Functions

Most operations are covered out of the box. However, in case you need custom logic, and if the user ask you to do so, you can create custom routers.

In general, it's very rare to create custom router, only create custom routers if the user request asks for it.

### Server: tRPC Custom Routers

Custom routers are defined in `app/server/routers/[name].router.ts`. When creating these routers, you have access to the `ctx` object, which provides:

- `ctx.session.user`: The authenticated user who triggered the request.
- `ctx.database`: The Prisma client with Zenstack policies.
- `ctx.databaseUnprotected`: The Prisma client without Zenstack policies, for unrestricted operations.

#### Example - Custom Mutation

```ts
import { Trpc } from '@/core/trpc/base'
import { z } from 'zod'

export const MyRouter = Trpc.createRouter({
  createCustomUser: Trpc.procedure
    .input(z.object({ name: z.string(), email: z.string() }))
    .mutation(async ({ ctx, input }) => {
      const user = await ctx.database.user.create({ data: input })
      return user
    }),
})
```

To add custom routers, import them in `app/server/index.tsx`:

```ts
import { MyRouter } from './routers/my-router'

const appRouter = Trpc.mergeRouters(Trpc.createRouter({ myRouter: MyRouter }))
```

---

## Common Errors

### Accessing Nested Models

When you need to access foreign relation data in nested models, use the `include` attributes in your fetch query to include the nested models. You can also include sub-level nested models if needed.

### Required Properties in Create Queries

When creating a new entity with the API, ensure that all required properties are included in the create query. If any required properties are missing, the query will fail.

### Do Not Re-Create Routers

The auto-generated routers from Zenstack already exist in the `node_modules`. You cannot see them, but they are always available. Do not attempt to re-create them manually.

### Types & Relations in Frontend

Prisma types do not include relations by default. To avoid TypeScript errors, include relations explicitly in your queries and redeclare types when necessary.

#### Example

```tsx
import { Prisma } from '@prisma'

const [value, setValue] =
  useState<Prisma.UserGetPayload<{ include: { relation1: true } }>>(null)

const { data } = Api.user.findMany.useQuery({ include: { relation1: true } })

useEffect(() => {
  setValue(data[0])
}, [data])

console.log(value.relation1)
```

---

## Key Takeaways

- No router creation is needed when adding or editing data models; Zenstack generates everything automatically.
- Use the `Api` object for frontend CRUD operations with `@/core/trpc`.
- Custom routers are used for non-CRUD operations, and should be placed in `app/server/routers/[name].router.ts`.
- Use `ctx` in the server for accessing session and Prisma clients.

# Typescript conventions

## Prefer Named Exports

In our TypeScript codebase, **named exports** should be preferred over `export default` except in Remix pages.
This makes it easier to understand what is being imported, encourages consistency, and avoids issues when renaming exports during imports.

### Example: Named Export

Instead of using a default export:

```typescript
// Bad Example: default export
export default function myFunction() {
  // implementation
}
```

Use a named export:

```typescript
// Good Example: named export
export function myFunction() {
  // implementation
}
```

Named exports are imported like this:

```typescript
import { myFunction } from './myModule'
```

For Remix pages, it's necessary to use the export default convention. In these specific cases, export default is fine:

```typescript
// Remix page example
export default function HomePage() {
  // implementation
}
```

But for all other cases, avoid export default and stick to named exports.

## Typescript Tips

- When iterating through an array using `.map`, always ensure the array is not null by using the optional chaining operator (`?`) like this: `items?.map`.
- Use `<InputNumber/>` for any input that requires a user to enter a number.
- You will be penalized if you use Moment.js to manipulate dates. Only dayJs is allowed.
- Always use `toString()` when displaying a number or decimal in the HTML of a component, or TypeScript will throw an error.

---
> Source: [umussetu/marble-stack](https://github.com/umussetu/marble-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
