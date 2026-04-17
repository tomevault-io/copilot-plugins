## th-tip-top

> Develop TypeScript, Node.js, and Next.js applications following provided conventions and guidelines.

Develop TypeScript, Node.js, and Next.js applications following provided conventions and guidelines.

**Code Style and Structure**

- Write concise, technical TypeScript code using functional and declarative programming patterns.
- Avoid classes; prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`).
- Structure files into: exported component, subcomponents, helpers, static content, and types.

**Naming Conventions**

- Use lowercase with dashes for directories (e.g., `components/auth-wizard`).
- Favor named exports for components.

**TypeScript Usage**

- Use TypeScript for all code; prefer types over interfaces.
- Avoid enums; use maps instead.
- Use functional components with TypeScript types.

**Syntax and Formatting**

- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
- Write declarative JSX.

**UI and Styling**

- Use Shadcn UI, Radix, and Tailwind for components and styling.
- Implement responsive design with Tailwind CSS using a mobile-first approach.

**Performance Optimization**

- Minimize `use client`, `useEffect`, and `setState`; favor React Server Components (RSC).
- Wrap client components in `Suspense` with fallback.
- Use dynamic loading for non-critical components.
- Optimize images: use WebP format, include size data, and implement lazy loading.

**Database Querying & Data Model Creation**

- Use Prisma SDK to query the database.
- For data models, read the `.prisma` files.

**Key Conventions**

- Use 'nuqs' for URL search parameter state management.
- Optimize Web Vitals (LCP, CLS, FID).
- Limit 'use client': Favor server components and Next.js SSR for data fetching or state management.
- Use 'use client' only for Web API access in small components.

**PostgresQL**

- Use valid PostgresQL syntax with guillemets for table and column names.

**React 19**

- Utilize React 19 with Server Components. Implement Prisma queries and backend logic inside `page` or `layout` files like this:

\`\`\`tsx
// Use "async" for server components
export default async function Page() {
// Use "await" for async operations
const result = await prisma.user.findMany();
return (

<div>
{result.map((user) => (
<p>{user.name}</p>
))}
</div>
);
}
\`\`\`

- Avoid using React hooks within server components.

**Creating a component**

- You always use `export default function` for components.
- You always use an object "props" as the first argument of your component, and add type directly in the object.

Example :

\`\`\`tsx
export default function MyComponent(props: { prop1: string; prop2: number }) {
return <div>{props.prop1}</div>;
}
\`\`\`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaye2512) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
