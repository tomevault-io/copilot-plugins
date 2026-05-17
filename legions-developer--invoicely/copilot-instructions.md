## nextjs-cursor-rule

> You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.

  You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.
  
  Code Style and Structure
  - Write concise, technical TypeScript code with accurate examples.
  - Use functional and declarative programming patterns; avoid classes.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  - Structure files: exported component, subcomponents, helpers, static content, types.
  - Follow seprate export const component name like don't "export const Button = () => {}" make it do like "const Button = () => {}  export { Button }" 
  - Don't use default exports.
  
  Naming Conventions
  - use name.client.tsx for file having "use client" or its a client component so add (.client.tsx) in end of file name.
  - Use lowercase with dashes for directories (e.g., components/auth-wizard).
  - Favor named exports for components.
  - For all Zod Type must include "Zod" in front of every type of interace example "InvoiceSchema" should be "ZodInvoiceSchema".
  
  TypeScript Usage
  - Use TypeScript for all code; prefer interfaces over types.
  - Avoid enums; use maps instead.
  - Use functional components with TypeScript interfaces.
  
  Syntax and Formatting
  - Use the "function" keyword for pure functions.
  - Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
  - Use declarative JSX.
  
  UI and Styling
  - Use Shadcn UI, Radix, and Tailwind for components and styling.
  - Implement responsive design with Tailwind CSS; use a mobile-first approach.
  
  Performance Optimization
  - Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC).
  - Wrap client components in Suspense with fallback.
  - Use dynamic loading for non-critical components.
  - Optimize images: use WebP format, include size data, implement lazy loading.
  
  Key Conventions
  - Use 'nuqs' for URL search parameter state management.
  - Optimize Web Vitals (LCP, CLS, FID).
  - Limit 'use client':
    - Favor server components and Next.js SSR.
    - Use only for Web API access in small components.
    - Avoid for data fetching or state management.

  Github Commit Syntax
  - use <type>(required scope any related to commit file): single title for commit without description
 
  
  Follow Next.js docs for Data Fetching, Rendering, and Routing.
  

---
> Source: [legions-developer/invoicely](https://github.com/legions-developer/invoicely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
