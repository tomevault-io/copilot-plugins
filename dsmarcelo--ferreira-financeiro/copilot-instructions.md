## ferreira-financeiro

> This rule describes how to implement loading states and streaming UI in Next.js App Router projects, following best practices for user experience and performance.

# Loading Components in Next.js

## Purpose
This rule describes how to implement loading states and streaming UI in Next.js App Router projects, following best practices for user experience and performance.

## Key Principles
- Use a `loading.tsx` file in the c folder as your `page.tsx` or `layout.tsx` to provide an instant loading state for the entire route segment.
- For more granular loading states, use React's `<Suspense>` component to wrap parts of your UI that depend on asynchronous data.
- Always provide a meaningful fallback UI (e.g., skeletons, spinners, or placeholder text) to indicate loading.
- Loading components should be simple, accessible, and styled with Tailwind CSS and/or Shadcn UI.

## Example: Route Segment Loading

Create a `loading.tsx` file in your route folder:

```tsx
// [your-route]/loading.tsx
export default function Loading() {
  return (
    <div className="flex flex-col gap-4">
      <p className="animate-pulse bg-gray-200 font-semibold">Carregando...</p>
    </div>
  );
}
```

- This component will be shown automatically while the route's data is loading.
- The loading UI is replaced with the actual content once data fetching completes.

## Example: Granular Loading with Suspense (Most cases)

For finer control, wrap async components in `<Suspense>`:

```tsx
import { Suspense } from "react";
import BlogList from "@/components/BlogList";
import BlogListSkeleton from "@/components/BlogListSkeleton";
import { getBlogPost } from '@/db/queries'

export default async function BlogPage() {
  const blogPosts = getBlogPosts() //This is a async function, but it should not have a await, beause it's going to be awaited in the component.

  return (
    <div>
      <header>
        <h1>Welcome to the Blog</h1>
      </header>
      <main>
        <Suspense fallback={<BlogListSkeleton />}>
          <BlogList blogPosts={blogPosts} />
        </Suspense>
      </main>
    </div>
  );
}
```

### In the list component, use react's use hook:
```tsx
import {use} from 'react'

export default BlogList({blogPosts} : { blogPosts: Promise<BlogPosts[] }) {
  const allBlogPosts = use(BlogPosts)

  return (
    <div>
      {allBlogPosts.map((post) => (
        <p>{post.title}</p>
      ))}
    </div>
  )
}

```

- The fallback UI (`BlogListSkeleton`) is shown while `BlogList` loads.

## Best Practices

- Keep loading components small and focused.
- Use clear, accessible language and ARIA attributes where appropriate.
- Prefer skeletons or meaningful placeholders over generic spinners.
- Test loading states for accessibility and responsiveness.

## References

- [Next.js Docs: Fetching Data & Streaming](mdc:https:/nextjs.org/docs/app/getting-started/fetching-data#with-the-use-hook)
- https://nextjs.org/docs/app/getting-started/fetching-data#with-the-use-hook

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dsmarcelo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
