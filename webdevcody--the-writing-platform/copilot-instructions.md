## tanstack

> In the react components, never try to call a use case directly; instead, please create a server action using the proper validators, middlewares, and handlers.


# Server Functions

In the react components, never try to call a use case directly; instead, please create a server action using the proper validators, middlewares, and handlers.

# UI / UX

Please make sure all async requests show some type of loading indicator. This often means using a useMutation and wrapping the server function so I get access to loading boolean which I Can use to show spinners. Disable the relevant buttons when the async action is running.

# Forms

all forms should be using the default shadcn form approach which would involve using react-hook-form, zod, have client side validation, a toast on errors or success.  See [books_.create.tsx](mdc:app/routes/books_.create.tsx) for an example.

# Loaders

When a page is loading, please remember to add a loader to the route which should invoke a loader server function to fetch data. Do not try to invoke a use case directly from a tanstack loader.

# Loader & Server Functions

Here is a basic example of how to create a loader for a route, invoke a loader server function, fetch some data, and then access it in the route

```
const loaderFn = createServerFn()
  .validator(
      z.object({
        title: z.string(),
      })
  )
  .handler(async () => {
    const segments = await getSegments();
    return { segments };
  });

export const Route = createFileRoute("/")({
  component: Home,
  loader: () => loaderFn(),
});

function Home() {
  const { segments } = Route.useLoaderData();
```

## Validators

You should always add a validator to server functions.

```
const createSegmentFn = createServerFn()
  .middleware([adminMiddleware])
  .validator(
    z.object({
      title: z.string(),
      content: z.string(),
      videoKey: z.string().optional(),
      slug: z.string(),
      moduleTitle: z.string(),
      length: z.string().optional(),
      isPremium: z.boolean(),
    })
  )
  .handler(async ({ data }) => {
```


# File Naming Convention

all files must be lowercase with hyphens.

# Data Access Prefernce

try to use the database.query method over doing manual select statements. here is an example of the query method: `await database.query.modules.findFirst(`

# Link component

never interpolate the to string, instead use props:

```
<Link 
  to="/learn/$slug"
  params={{ slug: continueSlug }}>Start Learning</Link>
```

# Rules

- Never invoke backend code outside of a server function when in a .tsx file
- Never import next.js related code please

---
> Source: [webdevcody/the-writing-platform](https://github.com/webdevcody/the-writing-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
