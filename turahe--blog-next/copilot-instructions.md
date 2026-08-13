## blog-next

> This repository contains a personal developer blog and portfolio built with:

# AGENTS.md

# Personal Blog & Portfolio — AI Coding Agent Instructions

## 1. Project Overview

This repository contains a personal developer blog and portfolio built with:

* Next.js
* TypeScript
* Tailwind CSS
* MDX/content-based blog
* Scroll World cinematic experience
* Responsive UI
* SEO-focused architecture
* Performance-focused frontend

The project is a **content-first engineering portfolio**, not an animation showcase.

The website must remain useful, accessible, indexable, and performant even when the Scroll World experience is unavailable.

---

# 2. Source of Truth

Before making architectural or implementation decisions, read the relevant documentation.

Required documents:

```text
PRD.md
DESIGN.md
```

Relevant skills:

```text
.cursor/skills/ui-design/SKILL.md
.cursor/skills/scroll-world/SKILL.md
.cursor/skills/seo/SKILL.md
.cursor/skills/performance/SKILL.md
```

Additional documentation may exist under:

```text
docs/
```

Do not invent requirements that contradict these documents.

When requirements conflict, use this priority:

```text
PRD.md
    ↓
DESIGN.md
    ↓
Architecture decisions
    ↓
Skills
    ↓
Implementation details
```

---

# 3. General Agent Behavior

Before changing code:

1. Inspect the repository.
2. Understand the existing architecture.
3. Read relevant documentation.
4. Identify affected components.
5. Determine whether the change affects:

   * UI
   * SEO
   * performance
   * Scroll World
   * accessibility
   * content
6. Make the smallest reasonable change.
7. Run relevant validation.
8. Review the resulting implementation.

Do not rewrite unrelated code.

Do not introduce dependencies without justification.

Do not create abstractions prematurely.

---

# 4. Required Skill Selection

Use the following rules.

## UI changes

Read:

```text
.cursor/skills/ui-design/SKILL.md
```

Examples:

* components
* layouts
* navigation
* buttons
* cards
* typography
* responsive design
* dark mode
* animations

---

## Scroll World changes

Read:

```text
.cursor/skills/scroll-world/SKILL.md
.cursor/skills/ui-design/SKILL.md
.cursor/skills/performance/SKILL.md
```

Examples:

* scene
* camera
* video
* scroll animation
* cinematic hero
* visual transitions
* scene assets

Scroll World changes MUST also consider performance.

---

## SEO changes

Read:

```text
.cursor/skills/seo/SKILL.md
```

Examples:

* metadata
* sitemap
* robots
* canonical
* structured data
* Open Graph
* article metadata
* internal linking

---

## Performance changes

Read:

```text
.cursor/skills/performance/SKILL.md
```

Examples:

* bundle size
* Core Web Vitals
* image optimization
* video optimization
* lazy loading
* dynamic imports
* hydration
* client/server boundaries
* scroll performance

---

## Blog/content changes

Read:

```text
.cursor/skills/seo/SKILL.md
.cursor/skills/ui-design/SKILL.md
```

Examples:

* MDX
* articles
* categories
* tags
* article pages
* article layout
* table of contents

---

# 5. Cross-Skill Rules

Some changes require multiple skills.

## Homepage

Read:

```text
ui-design
scroll-world
performance
seo
```

---

## Blog Article

Read:

```text
ui-design
seo
performance
```

---

## Project Case Study

Read:

```text
ui-design
seo
performance
```

---

## Scroll World Hero

Read:

```text
ui-design
scroll-world
performance
seo
```

---

# 6. Architecture Principles

## Server Components First

Next.js Server Components are the default.

Do not add:

```tsx
"use client"
```

unless client-side behavior is required.

Client Components are appropriate for:

* browser APIs
* interactive UI
* animation
* Scroll World
* client state
* event handlers

Keep Client Component boundaries as small as practical.

---

# 7. Content Architecture

Content must remain independent from presentation.

Preferred:

```text
content
   ↓
data/model
   ↓
page
   ↓
components
```

Do not embed large amounts of blog/project content directly inside reusable UI components.

---

# 8. Blog Architecture

Blog content should preferably be stored as MDX or another static content format.

Example:

```text
content/
└── blog/
    ├── article-one/
    │   └── index.mdx
    ├── article-two/
    │   └── index.mdx
    └── article-three/
        └── index.mdx
```

Each article should have frontmatter.

Example:

```yaml
---
title: "Designing Reliable Go Services"
description: "..."
date: "2026-08-01"
tags:
  - Go
  - Architecture
category: "Software Engineering"
draft: false
---
```

Do not expose draft articles publicly.

---

# 9. Project Architecture

Projects should support case-study content.

Recommended structure:

```text
content/
└── projects/
    ├── project-one/
    ├── project-two/
    └── project-three/
```

A project should be able to contain:

* overview
* problem
* solution
* architecture
* implementation
* technologies
* challenges
* results
* lessons learned

---

# 10. UI Architecture

Prefer:

```text
components/
├── layout/
├── ui/
├── hero/
├── blog/
├── projects/
└── scroll-world/
```

Do not create one giant:

```text
components/HomePage.tsx
```

containing the entire website.

---

# 11. Design System

`DESIGN.md` is the visual source of truth.

Use centralized design tokens.

Do not introduce arbitrary colors, spacing, typography, or radii without a reason.

Tailwind should be used consistently.

Avoid excessive arbitrary values such as:

```text
mt-[37px]
text-[#123456]
w-[913px]
```

when an existing design token can be used.

---

# 12. Semantic HTML

Prefer semantic HTML.

Use:

```html
<header>
<nav>
<main>
<section>
<article>
<aside>
<footer>
```

Use:

```html
<button>
```

for actions.

Use:

```html
<a>
```

for navigation.

Do not use clickable `<div>` elements as substitutes.

---

# 13. Accessibility

Accessibility is mandatory.

Every implementation must consider:

* keyboard navigation
* visible focus states
* semantic HTML
* heading hierarchy
* color contrast
* alt text
* accessible labels
* touch target size
* reduced motion

Do not rely on animation to communicate essential information.

---

# 14. Scroll World Rules

Scroll World is an enhancement layer.

It MUST NOT be the only representation of:

* page title
* developer identity
* project information
* article information
* navigation
* important CTAs

Critical content must exist as semantic HTML.

---

# 15. Scroll World Failure Strategy

The site must remain usable if:

```text
JavaScript fails
```

or:

```text
Scroll World fails
```

or:

```text
video fails
```

or:

```text
user enables reduced motion
```

Fallback hierarchy:

```text
Full Scroll World
       ↓
Reduced animation
       ↓
Static hero
       ↓
Normal semantic HTML
```

---

# 16. Performance Rules

Performance is a first-class requirement.

Do not:

* load all Scroll World assets immediately
* ship unnecessary JavaScript
* turn entire pages into Client Components
* load desktop assets on mobile unnecessarily
* use oversized images
* block rendering on animation initialization
* animate layout properties continuously

Prefer:

```text
Server Components
+
Static content
+
Optimized images
+
Lazy loading
+
Dynamic imports
+
Small Client Components
```

---

# 17. SEO Rules

All public pages must be indexable unless explicitly marked otherwise.

Important information must exist in HTML.

Do not place critical content only inside:

* Canvas
* WebGL
* video
* image
* animation

Every public page should have appropriate:

* title
* description
* canonical
* Open Graph metadata

Blog articles should have appropriate structured data.

---

# 18. Metadata Rules

Do not duplicate generic metadata blindly.

Metadata should reflect the actual page.

For example:

```text
Home
→ Personal developer portfolio

Blog article
→ Specific article title and description

Project
→ Specific project title and description
```

Do not keyword stuff.

---

# 19. Images

Use Next.js image optimization where appropriate.

Images should have:

* appropriate dimensions
* responsive behavior
* meaningful alt text
* optimized format

Avoid layout shifts.

---

# 20. Fonts

Use a limited font system.

Do not load many unnecessary weights.

Prefer Next.js font optimization.

Do not add external font requests without justification.

---

# 21. Dependencies

Before adding a package, evaluate:

1. Is it necessary?
2. Can native browser APIs solve it?
3. Does Next.js already provide the capability?
4. Does it increase client bundle size?
5. Does it affect performance?
6. Is the package actively maintained?

Avoid dependency accumulation.

---

# 22. State Management

Do not introduce global state unless necessary.

Prefer:

```text
server data
+
URL state
+
local component state
```

before adding a global store.

---

# 23. Data Fetching

For static content:

Prefer:

```text
build/server
```

over:

```text
browser fetch
```

Do not fetch MDX/blog content from the client unnecessarily.

---

# 24. API Usage

If an API is required:

* validate input
* handle errors
* define types
* avoid exposing secrets
* implement appropriate caching
* protect write endpoints

Never expose secrets in client-side code.

---

# 25. Security

Never commit:

```text
.env
API keys
tokens
credentials
private certificates
```

Use environment variables.

Client-side environment variables must never contain secrets.

---

# 26. URL and Routing Rules

Preferred:

```text
/
/about
/projects
/projects/[slug]
/blog
/blog/[slug]
/resume
/contact
```

Use stable, human-readable slugs.

Avoid unnecessary query-based routing.

---

# 27. Error Handling

Provide appropriate:

```text
loading.tsx
error.tsx
not-found.tsx
```

where they improve the user experience.

Errors should not expose internal implementation details.

---

# 28. Loading States

Loading states must not cause layout jumps.

Prefer:

* skeletons
* reserved space
* static placeholders

For Scroll World:

```text
poster image
```

should be used while the experience initializes.

---

# 29. Testing

Before declaring a feature complete, run appropriate checks.

Minimum:

```text
lint
typecheck
build
```

Where configured.

For visual changes:

* desktop
* tablet
* mobile

For Scroll World:

* normal motion
* reduced motion
* JavaScript failure/fallback where practical

For SEO:

* metadata
* sitemap
* robots
* structured data

For performance:

* production build
* Lighthouse or equivalent
* bundle inspection when needed

---

# 30. Validation Commands

Use the project's package manager.

If package manager is unclear, inspect:

```text
package.json
lockfile
```

Possible commands:

```bash
pnpm lint
pnpm typecheck
pnpm build
```

Do not assume these commands exist.

Inspect `package.json` first.

---

# 31. Git Discipline

Keep changes focused.

Prefer:

```text
feature:
fix:
refactor:
perf:
docs:
```

style commits where appropriate.

Do not mix unrelated refactoring into a feature.

Do not modify generated files unless required.

---

# 32. Before Implementation

For non-trivial tasks, create or update a plan.

Example:

```text
Task
 ↓
Affected files
 ↓
Architecture
 ↓
Implementation
 ↓
Validation
```

For major features, update relevant documentation before coding.

---

# 33. Implementation Workflow

Use this workflow for major tasks:

```text
1. Understand request
        ↓
2. Read PRD
        ↓
3. Read DESIGN
        ↓
4. Identify required skills
        ↓
5. Inspect repository
        ↓
6. Create implementation plan
        ↓
7. Implement
        ↓
8. Test
        ↓
9. Review
        ↓
10. Report changes
```

---

# 34. Do Not Hallucinate

Do not assume:

* a library is installed
* a component exists
* an API exists
* a route exists
* a database exists
* an environment variable exists
* a skill exists

Inspect the repository first.

If a required dependency or architecture is missing, state the assumption before implementing it.

---

# 35. Do Not Over-Engineer

Prefer the simplest architecture that satisfies the requirements.

Do not introduce:

* microservices
* databases
* queues
* global state
* complex caching
* search infrastructure
* WebGL

unless the requirements justify them.

A personal blog should remain simple.

---

# 36. Visual Quality Gate

Before completing UI work, verify:

```text
Typography
Spacing
Hierarchy
Responsive layout
Accessibility
Motion
Consistency
```

The page must not look like a generic generated template.

---

# 37. Performance Quality Gate

Before completing performance-sensitive work, verify:

```text
Initial render
Client JS
Images
Fonts
Video
Scroll performance
CLS
LCP
INP
Mobile
```

---

# 38. SEO Quality Gate

Before completing SEO work, verify:

```text
Title
Description
Canonical
Open Graph
Structured Data
Sitemap
Robots
Headings
Internal Links
Indexability
```

---

# 39. Scroll World Quality Gate

Before completing Scroll World work, verify:

```text
Storyboard
Scene continuity
Desktop
Tablet
Mobile fallback
Reduced motion
Static fallback
Asset optimization
Scroll smoothness
SEO-safe HTML
```

---

# 40. Final Review

Before reporting a task as complete, ask:

### Requirements

* Did I satisfy the PRD?

### Design

* Did I follow DESIGN.md?

### Architecture

* Did I preserve the existing architecture?

### UI

* Is the interface responsive and accessible?

### SEO

* Is the content indexable and correctly described?

### Performance

* Did I introduce unnecessary JavaScript or assets?

### Scroll World

* Does the experience enhance rather than replace content?

### Testing

* Did I run the relevant validation?

If any answer is "no", do not claim the task is complete.

---

# 41. Final Response Format

When reporting completed work, provide:

```text
## Implemented

- ...
- ...

## Files Changed

- ...
- ...

## Validation

- ...
- ...

## Notes

- ...
```

Mention known limitations explicitly.

Do not claim tests passed if they were not executed.

<!-- NEXT-AGENTS-MD-START -->[Next.js Docs Index]|root: ./node_modules/next/dist/docs|STOP. What you remember about Next.js is WRONG for this project. Always search docs and read before any task.|If docs missing, run this command first: npx @next/codemod agents-md --output AGENTS.md|01-app:{04-glossary.md}|01-app/01-getting-started:{01-installation.md,02-project-structure.md,03-layouts-and-pages.md,04-linking-and-navigating.md,05-server-and-client-components.md,06-fetching-data.md,07-mutating-data.md,08-caching.md,09-revalidating.md,10-error-handling.md,11-css.md,12-images.md,13-fonts.md,14-metadata-and-og-images.md,15-route-handlers.md,16-proxy.md,17-deploying.md,18-upgrading.md}|01-app/02-guides:{ai-agents.md,analytics.md,authentication.md,backend-for-frontend.md,caching-without-cache-components.md,ci-build-caching.md,content-security-policy.md,css-in-js.md,custom-server.md,data-security.md,debugging.md,draft-mode.md,environment-variables.md,forms.md,incremental-static-regeneration.md,instant-navigation.md,instrumentation.md,internationalization.md,json-ld.md,lazy-loading.md,local-development.md,mcp.md,mdx.md,memory-usage.md,migrating-to-cache-components.md,multi-tenant.md,multi-zones.md,open-telemetry.md,package-bundling.md,prefetching.md,preserving-ui-state.md,production-checklist.md,progressive-web-apps.md,public-static-pages.md,redirecting.md,sass.md,scripts.md,self-hosting.md,single-page-applications.md,static-exports.md,streaming.md,tailwind-v3-css.md,third-party-libraries.md,videos.md}|01-app/02-guides/migrating:{app-router-migration.md,from-create-react-app.md,from-vite.md}|01-app/02-guides/testing:{cypress.md,jest.md,playwright.md,vitest.md}|01-app/02-guides/upgrading:{codemods.md,version-14.md,version-15.md,version-16.md}|01-app/03-api-reference:{07-edge.md,08-turbopack.md}|01-app/03-api-reference/01-directives:{use-cache-private.md,use-cache-remote.md,use-cache.md,use-client.md,use-server.md}|01-app/03-api-reference/02-components:{font.md,form.md,image.md,link.md,script.md}|01-app/03-api-reference/03-file-conventions/01-metadata:{app-icons.md,manifest.md,opengraph-image.md,robots.md,sitemap.md}|01-app/03-api-reference/03-file-conventions/02-route-segment-config:{dynamicParams.md,instant.md,maxDuration.md,preferredRegion.md,runtime.md}|01-app/03-api-reference/03-file-conventions:{default.md,dynamic-routes.md,error.md,forbidden.md,instrumentation-client.md,instrumentation.md,intercepting-routes.md,layout.md,loading.md,mdx-components.md,not-found.md,page.md,parallel-routes.md,proxy.md,public-folder.md,route-groups.md,route.md,src-folder.md,template.md,unauthorized.md}|01-app/03-api-reference/04-functions:{after.md,cacheLife.md,cacheTag.md,catchError.md,connection.md,cookies.md,draft-mode.md,fetch.md,forbidden.md,generate-image-metadata.md,generate-metadata.md,generate-sitemaps.md,generate-static-params.md,generate-viewport.md,headers.md,image-response.md,next-request.md,next-response.md,not-found.md,permanentRedirect.md,redirect.md,refresh.md,revalidatePath.md,revalidateTag.md,unauthorized.md,unstable_cache.md,unstable_noStore.md,unstable_rethrow.md,updateTag.md,use-link-status.md,use-params.md,use-pathname.md,use-report-web-vitals.md,use-router.md,use-search-params.md,use-selected-layout-segment.md,use-selected-layout-segments.md,userAgent.md}|01-app/03-api-reference/05-config/01-next-config-js:{adapterPath.md,allowedDevOrigins.md,appDir.md,assetPrefix.md,authInterrupts.md,basePath.md,cacheComponents.md,cacheHandlers.md,cacheLife.md,compress.md,crossOrigin.md,cssChunking.md,deploymentId.md,devIndicators.md,distDir.md,env.md,expireTime.md,exportPathMap.md,generateBuildId.md,generateEtags.md,headers.md,htmlLimitedBots.md,httpAgentOptions.md,images.md,incrementalCacheHandlerPath.md,inlineCss.md,logging.md,mdxRs.md,onDemandEntries.md,optimizePackageImports.md,output.md,pageExtensions.md,poweredByHeader.md,productionBrowserSourceMaps.md,proxyClientMaxBodySize.md,reactCompiler.md,reactMaxHeadersLength.md,reactStrictMode.md,redirects.md,rewrites.md,sassOptions.md,serverActions.md,serverComponentsHmrCache.md,serverExternalPackages.md,staleTimes.md,staticGeneration.md,taint.md,trailingSlash.md,transpilePackages.md,turbopack.md,turbopackFileSystemCache.md,turbopackIgnoreIssue.md,typedRoutes.md,typescript.md,urlImports.md,useLightningcss.md,viewTransition.md,webVitalsAttribution.md,webpack.md}|01-app/03-api-reference/05-config:{02-typescript.md,03-eslint.md}|01-app/03-api-reference/06-cli:{create-next-app.md,next.md}|02-pages/01-getting-started:{01-installation.md,02-project-structure.md,04-images.md,05-fonts.md,06-css.md,11-deploying.md}|02-pages/02-guides:{analytics.md,authentication.md,babel.md,ci-build-caching.md,content-security-policy.md,css-in-js.md,custom-server.md,debugging.md,draft-mode.md,environment-variables.md,forms.md,incremental-static-regeneration.md,instrumentation.md,internationalization.md,lazy-loading.md,mdx.md,multi-zones.md,open-telemetry.md,package-bundling.md,post-css.md,preview-mode.md,production-checklist.md,redirecting.md,sass.md,scripts.md,self-hosting.md,static-exports.md,tailwind-v3-css.md,third-party-libraries.md}|02-pages/02-guides/migrating:{app-router-migration.md,from-create-react-app.md,from-vite.md}|02-pages/02-guides/testing:{cypress.md,jest.md,playwright.md,vitest.md}|02-pages/02-guides/upgrading:{codemods.md,version-10.md,version-11.md,version-12.md,version-13.md,version-14.md,version-9.md}|02-pages/03-building-your-application/01-routing:{01-pages-and-layouts.md,02-dynamic-routes.md,03-linking-and-navigating.md,05-custom-app.md,06-custom-document.md,07-api-routes.md,08-custom-error.md}|02-pages/03-building-your-application/02-rendering:{01-server-side-rendering.md,02-static-site-generation.md,04-automatic-static-optimization.md,05-client-side-rendering.md}|02-pages/03-building-your-application/03-data-fetching:{01-get-static-props.md,02-get-static-paths.md,03-forms-and-mutations.md,03-get-server-side-props.md,05-client-side.md}|02-pages/03-building-your-application/06-configuring:{12-error-handling.md}|02-pages/04-api-reference:{06-edge.md,08-turbopack.md}|02-pages/04-api-reference/01-components:{font.md,form.md,head.md,image-legacy.md,image.md,link.md,script.md}|02-pages/04-api-reference/02-file-conventions:{instrumentation.md,proxy.md,public-folder.md,src-folder.md}|02-pages/04-api-reference/03-functions:{get-initial-props.md,get-server-side-props.md,get-static-paths.md,get-static-props.md,next-request.md,next-response.md,use-params.md,use-report-web-vitals.md,use-router.md,use-search-params.md,userAgent.md}|02-pages/04-api-reference/04-config/01-next-config-js:{adapterPath.md,allowedDevOrigins.md,assetPrefix.md,basePath.md,bundlePagesRouterDependencies.md,compress.md,crossOrigin.md,deploymentId.md,devIndicators.md,distDir.md,env.md,exportPathMap.md,generateBuildId.md,generateEtags.md,headers.md,httpAgentOptions.md,images.md,logging.md,onDemandEntries.md,optimizePackageImports.md,output.md,pageExtensions.md,poweredByHeader.md,productionBrowserSourceMaps.md,proxyClientMaxBodySize.md,reactStrictMode.md,redirects.md,rewrites.md,serverExternalPackages.md,trailingSlash.md,transpilePackages.md,turbopack.md,typescript.md,urlImports.md,useLightningcss.md,webVitalsAttribution.md,webpack.md}|02-pages/04-api-reference/04-config:{01-typescript.md,02-eslint.md}|02-pages/04-api-reference/05-cli:{create-next-app.md,next.md}|03-architecture:{accessibility.md,fast-refresh.md,nextjs-compiler.md,supported-browsers.md}|04-community:{01-contribution-guide.md,02-rspack.md}<!-- NEXT-AGENTS-MD-END -->

---
> Source: [turahe/blog-next](https://github.com/turahe/blog-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
