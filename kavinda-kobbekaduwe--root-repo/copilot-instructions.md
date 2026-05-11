## root-repo

> This is a root template — clients duplicate it and build their site on top of it with Claude as their primary developer. That means every decision here sets a precedent. Architecture, patterns, and conventions established in this repo will be inherited and extended by Claude across every client project. Get it right here, and it propagates everywhere. Cut corners here, and those corners get copied.

# CLAUDE.md

## What this is

This is a root template — clients duplicate it and build their site on top of it with Claude as their primary developer. That means every decision here sets a precedent. Architecture, patterns, and conventions established in this repo will be inherited and extended by Claude across every client project. Get it right here, and it propagates everywhere. Cut corners here, and those corners get copied.

The stack: Astro 6 static site with Directus as the headless CMS. Content is fetched at build time via the Directus REST API. This file is your operating manual — read it before touching anything.

## Mental model

**Directus holds content. Astro renders it. Never the other way around.**

- Text, images, page structure → Directus
- Layout, components, routing logic → Astro/code
- If you're about to hardcode a string or image URL in a component, stop. It belongs in Directus.
- If you're about to fetch data inside a component's markup, stop. It belongs in frontmatter.

The single question to ask before any change: *does this belong in content or code?*

## Running locally

```bash
npm install
cp .env.example .env   # fill in DIRECTUS_URL and DIRECTUS_TOKEN
npm run dev
```

## Architecture

```
src/lib/types.ts      → All Directus collection types + Schema
src/lib/client.ts     → Directus SDK client instance (rarely touched)
src/lib/queries.ts    → All data-fetching helpers
src/lib/assets.ts     → getAssetUrl() + AssetTransforms type
src/layouts/Base.astro → HTML shell, SEO meta, global data (globals + nav)
src/components/       → Reusable UI pieces
src/pages/            → Routes. [slug].astro handles CMS-driven pages.
src/styles/global.css → Tailwind import + CSS custom properties (design tokens)
```

When adding a new Directus collection: add the type + update `Schema` in `types.ts`, then add a query helper in `queries.ts`.

## How to add a page

Most pages are CMS-driven and handled automatically by `[slug].astro`. You only need a new `.astro` file for pages with custom logic (e.g. a page that aggregates multiple collections, or has a non-standard layout). If you do create one:

1. Create `src/pages/your-page.astro`
2. Fetch in frontmatter using helpers from `@/lib/queries`
3. Wrap in `<Base>` layout, pass SEO props
4. Follow the pattern in `index.astro`

## How to add a component

1. Create `src/components/YourComponent.astro`
2. Define a typed `Props` interface at the top
3. Style with Tailwind utility classes only
4. Use `<DirectusImage>` for any image coming from Directus — never a raw `<img>` with a Directus asset URL
5. If the component needs client-side interactivity, use an Astro island (`client:load`, `client:visible`, etc.)

## How to extend Directus types

When the client's Directus schema has a new collection:

1. Add the type to `src/lib/types.ts`
2. Add it to the `Schema` type in the same file
3. Add a typed helper function in `src/lib/queries.ts` (`getCollection<T>()` already handles simple cases)

## Data fetching

- Always fetch in frontmatter (`---` block), never inside markup
- Use helpers from `@/lib/queries` — never call the Directus client directly from a page or component
- Images from Directus are file UUIDs — always pass through `getAssetUrl()` from `@/lib/assets` with appropriate transform options
- Use `<DirectusImage>` component for all Directus images

## Quality standards

Every piece of work — new component, new page, new section type — must meet all of these:

**Performance**
- Images must have explicit `width` and `height` to prevent layout shift
- Use `loading="lazy"` on all images below the fold; `loading="eager"` + `fetchpriority="high"` on hero/above-fold images
- No render-blocking resources
- Prefer `client:visible` over `client:load` for non-critical islands
- Keep JavaScript to zero unless the feature genuinely requires it
- Proactively recommend performance improvements when you spot an opportunity — even if not asked. Examples: self-hosting fonts instead of loading from Google Fonts, adding `dns-prefetch`/`preconnect` for third-party origins, replacing a JS library with a CSS-only solution, converting a client island to a static component. State the recommendation and the reason briefly alongside your other work.

**Accessibility**
- Semantic HTML: use `<nav>`, `<main>`, `<article>`, `<section>`, `<header>`, `<footer>` correctly
- Every image needs a meaningful `alt` attribute — describe the image, don't leave it empty unless decorative
- All interactive elements must be keyboard-navigable and have visible focus states
- Colour contrast must meet WCAG AA (4.5:1 for text, 3:1 for UI components)
- Forms need associated `<label>` elements and clear error states

**Design**
- Match the Figma design exactly — use the Figma MCP to read structure, spacing, and tokens. Never guess from a screenshot.
- Consistent spacing: use the Tailwind scale, don't invent arbitrary values
- Typography: maintain the established hierarchy (h1 → h2 → h3, prose for body)
- Responsive: mobile-first. Test every breakpoint. Nothing should overflow or collapse unexpectedly.
- Animations/transitions should be subtle and respect `prefers-reduced-motion`

**Security**
- Never expose `DIRECTUS_TOKEN` client-side — it's build-time only
- Rich text from Directus is rendered with `set:html`. **Directus does not sanitise HTML on output** — whatever an editor types is what gets rendered. The trust model here assumes Directus editors are trusted (client staff, not the public). Enforce this by: (1) locking down Directus roles so only trusted users can edit rich-text fields, (2) never rendering rich text that originated from untrusted input (form submissions, external APIs, user comments) through `set:html`, and (3) if an untrusted source is ever introduced, sanitise it with a library like `isomorphic-dompurify` in `queries.ts` before it reaches a component.
- No `dangerouslySetInnerHTML`, no `eval`, no dynamic script injection
- External links must have `rel="noopener noreferrer"`
- Forms must not submit sensitive data to third-party endpoints without explicit client approval

**UX**
- Every interactive element needs a hover and focus state
- Error states must be visible and descriptive — never silently fail
- Loading/empty states should be handled gracefully
- Never break the back button or cause unexpected navigation
- Contact forms need success and error feedback

## Styling rules

- Tailwind utility classes only
- No custom CSS unless there is genuinely no Tailwind equivalent
- Design tokens (colours, fonts, radii) are set as CSS custom properties in `src/styles/global.css` — use them for client-specific branding, don't scatter raw values through components
- Mobile-first: `base` → `md:` → `lg:`

## File conventions

- Components: `PascalCase.astro` in `src/components/`
- Pages: `kebab-case.astro` in `src/pages/`
- Directus types: `src/lib/types.ts`
- Directus query helpers: `src/lib/queries.ts`
- Layouts: `src/layouts/`

## What NOT to touch

- `src/lib/client.ts` — do not modify
- `astro.config.mjs` — only modify to add/configure Astro integrations
- `tsconfig.json` — do not change

## Figma workflow

When given a Figma URL or frame reference, use the Figma MCP to read the design. Extract exact spacing, typography, colours, and component structure from the design tokens. Do not guess values from screenshots.

## Deploy

Push to `main`. Railway auto-deploys. Write commit messages that describe *what changed and why*, not just *what file was edited*.

---
> Source: [kavinda-kobbekaduwe/root-repo](https://github.com/kavinda-kobbekaduwe/root-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
