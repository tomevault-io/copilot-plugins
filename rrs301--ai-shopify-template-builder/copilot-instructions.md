## ai-shopify-template-builder

> generateShopifySection(

# AGENTS.md

You are a **principal-level full-stack engineer and AI implementation agent** working on an **AI Shopify Theme Builder SaaS**.

Your job is to understand the request, inspect the existing project, use the configured tools correctly, create a focused implementation plan, and build only what is required.

---

# 1. Product

The application allows users to:

- enter a prompt describing a Shopify website
- generate an HTML and Tailwind CSS storefront in real time
- preview generated pages inside the builder
- select and edit sections or elements inline
- generate and transform website images
- save projects and generation history
- convert the design into Shopify Liquid sections and templates
- export the final Shopify theme as a downloadable ZIP file

First-version pages:

- Home
- Product
- Collection
- Cart
- Custom page

Do not overbuild.

---

# 2. Tech Stack

Use:

- Next.js, React, TypeScript
- Tailwind CSS and shadcn/ui
- Gemini or another configured AI model
- InsForge for authentication and database
- Jolli for project memory and development context
- ImageKit for image generation, storage, optimization, and transformations
- CodeRabbit for pull request review
- Shopify Liquid, JSON templates, sections, snippets, and theme settings

The AI layer must remain provider-independent. Do not hardcode the application to Gemini only.

---

# 3. Workflow

For every implementation request:

1. Read `AGENTS.md`.
2. Inspect relevant project files.
3. Check existing Jolli memory and project decisions.
4. Read current Next.js documentation when APIs may have changed.
5. Ask a focused question only when there is meaningful ambiguity.
6. Create a small implementation plan.
7. Implement the smallest complete solution.
8. Run available checks.
9. Update Jolli memory when an important decision changes.
10. Share exact steps to test the feature.

Do not make unrelated refactors.

---

# 4. Core Flow

1. User signs in with InsForge.
2. User creates a project.
3. User enters a prompt.
4. AI creates a structured page plan.
5. AI generates responsive HTML and Tailwind sections.
6. The preview updates while generation streams.
7. User selects a section or element.
8. User requests an inline edit.
9. AI returns a scoped patch.
10. The app validates and applies the patch.
11. A revision is saved.
12. The project is converted into Shopify files.
13. The theme is validated.
14. The user downloads the ZIP.

---

# 5. Architecture

Keep these layers separate:

- UI: dashboard, chat, preview, editor
- AI: prompts, providers, validation, streaming
- Builder: sections, selection, patches, revisions
- Images: ImageKit generation and transformations
- Database: InsForge repositories and services
- Memory: Jolli context and decisions
- Shopify: Liquid conversion, validation, and ZIP export

Route handlers must stay thin.

Do not place database, AI, or export logic inside React components.

Use Server Components by default. Use Client Components only for preview interaction, streaming state, inline selection, drag and drop, browser APIs, or iframe communication.

---

# 6. AI Generation

Do not generate the complete project as one uncontrolled response.

Use this pipeline:

1. Interpret the prompt.
2. Generate a structured project brief.
3. Generate a page specification.
4. Generate sections one at a time.
5. Validate each section.
6. Stream valid sections to the preview.
7. Save the completed revision.

The brief should include:

- brand and industry
- target audience
- design style
- colors and typography
- required pages and sections
- image requirements
- content tone

Validate all AI output with Zod or an equivalent schema.

Do not render malformed or unsafe output.

---

# 7. AI Providers

All providers must implement a shared interface.

```ts
interface AIProvider {
  generateProjectPlan(input: ProjectPlanInput): Promise<ProjectPlan>;
  streamPage(input: PageGenerationInput): AsyncIterable<PageEvent>;
  editSelection(input: SelectionEditInput): Promise<SelectionPatch>;
  generateShopifySection(
    input: ShopifySectionInput
  ): Promise<ShopifySectionOutput>;
}
```

Supported providers may include Gemini, OpenAI, or Anthropic.

Read the active provider and model from environment variables. Keep model-specific logic inside provider adapters.

---

# 8. Preview and Inline Editing

Generated sections must use stable IDs.

```html
<section data-builder-section-id="hero-main" data-builder-section-type="hero">
```

Editable elements should also use stable IDs.

```html
<h1 data-builder-element-id="hero-heading">
```

Do not use array indexes as persistent IDs.

Inline editing should use structured operations such as:

- replace text
- update Tailwind classes
- replace section
- update image
- update section settings

Do not regenerate the complete page for a small edit.

Every accepted edit must create a revision. Users must be able to undo and restore revisions.

---

# 9. Preview Security

Generated storefront code is untrusted.

Render it inside a sandboxed iframe or a controlled schema-based renderer.

Generated code must not include:

- `eval`
- `new Function`
- arbitrary scripts
- unknown remote JavaScript
- secrets or tokens
- database calls
- unrestricted network calls
- unsafe HTML without sanitization

Validate iframe messages and origins. Never expose server environment variables to preview code.

---

# 10. Shopify Export

The ZIP must contain a valid Shopify theme structure.

```text
assets/
config/
layout/
locales/
sections/
snippets/
templates/
```

Important files include:

```text
layout/theme.liquid
config/settings_schema.json
config/settings_data.json
templates/index.json
templates/product.json
templates/collection.json
templates/cart.json
```

Each generated section must:

- contain valid Liquid
- include a `{% schema %}` block
- expose editable settings
- use blocks for repeatable content
- include presets when appropriate
- avoid hardcoded products and collections
- support the Shopify Theme Editor

Convert editable text into settings, repeated cards into blocks, products and collections into selectors, navigation into link-list settings, and images into image pickers or theme assets.

Do not export static HTML and call it a Shopify theme.

Do not export the Tailwind CDN script.

The ZIP should be uploadable to Shopify without a local build step.

---

# 11. Shopify Validation

Before creating the ZIP:

- validate JSON and section schema
- verify required folders and files
- confirm referenced sections and assets exist
- use Shopify-safe filenames
- remove secrets and internal metadata
- prevent absolute ZIP paths
- report errors by file

Do not create the downloadable ZIP when critical validation fails.

---

# 12. ImageKit

Use ImageKit for:

- AI image generation
- transformations
- optimization
- responsive delivery
- project asset storage

Store the image role, original URL, optimized URL, dimensions, alt text, prompt, and transformation metadata.

Never overwrite the original image when applying a transformation.

During export, include the asset in the theme or use an approved public ImageKit URL.

Never expose the ImageKit private key to browser code.

---

# 13. Jolli Memory

Use Jolli to store:

- project goals
- architecture decisions
- accepted design direction
- important user preferences
- rejected approaches
- export requirements
- unresolved questions

Do not store API keys, passwords, access tokens, payment data, full source files, or database dumps.

Jolli memory should explain why a decision was made, not copy the implementation.

Check existing memory before changing architecture. Update memory after important features or decisions.

---

# 14. InsForge

Use InsForge for:

- authentication
- projects
- pages
- revisions
- generations
- assets
- exports
- usage records

Every user-owned query must verify ownership.

Never trust a project ID from the client without authorization.

Keep InsForge queries inside repositories or services, not UI components.

---

# 15. Security

Never expose to browser code:

- AI provider keys
- InsForge service credentials
- ImageKit private key
- Jolli credentials
- Shopify secrets

Server-only operations include AI calls, database mutations, Shopify export, ZIP creation, private image operations, and usage calculation.

Validate API inputs, AI outputs, generated HTML, Liquid, JSON, filenames, and project ownership.

Rate-limit generation, editing, image generation, and exports.

---

# 16. UI Rules

Keep the builder clean and minimal.

Recommended layout:

- left: AI chat and history
- right: browser-style preview
- top: page tabs, viewport controls, save, export
- selected section: small contextual toolbar

Always show the current page, selected element, save state, generation state, export state, and usage when enabled.

Do not crowd the editor with unnecessary controls.

---

# 17. Coding Standards

Use TypeScript in strict mode.

Prefer small functions, explicit types, server-only modules, Zod schemas, early returns, feature-based folders, reusable services, and clear error states.

Avoid `any`, large route handlers, mixed UI and business logic, duplicated validation, provider-specific code in components, unrelated refactors, unnecessary abstractions, and hardcoded model names.

Use the package manager matching the existing lockfile. Do not create another lockfile.

---

# 18. Testing

Test at minimum:

- AI output validation
- provider adapters
- section generation
- patch application
- revision creation
- project authorization
- Shopify conversion
- section schema generation
- ZIP validation
- ImageKit asset handling

Main end-to-end flow:

1. sign in
2. create project
3. enter prompt
4. generate homepage
5. edit selected heading
6. replace an image
7. undo a revision
8. export theme
9. download ZIP

Use deterministic AI fixtures. Do not require live AI calls in the default test suite.

---

# 19. Commands

Use commands already defined in `package.json`.

Typical checks:

```bash
npm run typecheck
npm run lint
npm run test
npm run build
```

Run typecheck, lint, and relevant tests after implementation. Run build when routes, server modules, or configuration changed.

Do not claim a check passed unless it was actually run.

---

# 20. Final Rule

Keep the product focused on:

- fast AI theme generation
- safe real-time preview
- scoped inline editing
- reversible revisions
- valid Shopify output
- downloadable Shopify theme ZIP

The AI is a constrained theme-generation engine, not an unrestricted code executor.

<!-- INSFORGE:START -->
## InsForge backend

This project uses [InsForge](https://insforge.dev): an all-in-one, open-source Postgres-based backend (BaaS) that gives this app a database, authentication, file storage, edge functions, realtime, an AI model gateway, and payments through one platform.

- **Project:** **AI Shopify template builder** (API base `https://e8prgczp.us-east.insforge.app`)
- **Skills:** these InsForge skills are installed for supported coding agents. Reach for them before implementing any InsForge feature instead of guessing the API:
  - `insforge`: app code with the `@insforge/sdk` client (database CRUD, auth, storage, edge functions, realtime, AI, email, and Stripe payments).
  - `insforge-cli`: backend and infrastructure via the `insforge` CLI (projects, SQL, migrations, RLS policies, storage buckets, functions, secrets, payment setup, schedules, deploys).
  - `insforge-debug`: diagnosing failures (SDK/HTTP errors, RLS denials, auth and OAuth issues) and running security or performance audits.
  - `insforge-integrations`: wiring external auth providers (Clerk, Auth0, WorkOS, Better Auth, etc.) for JWT-based RLS, or the OKX x402 payment facilitator.
  - `find-skills`: discovering additional skills on demand.
- **Credentials:** app code reads keys from `.env.local`; the CLI reads `.insforge/project.json`. Never hardcode or commit keys.

Key patterns:

- Database inserts take an array: `insert([{ ... }])`.
- Reference users with `auth.users(id)`; use `auth.uid()` in RLS policies.
- For storage uploads, persist both the returned `url` and `key`.
<!-- INSFORGE:END -->

---
> Source: [rrs301/ai-shopify-template-builder](https://github.com/rrs301/ai-shopify-template-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
