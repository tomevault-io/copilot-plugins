## fluksos

> description: Repository Context for AI Coding Assistants

---
description: Repository Context for AI Coding Assistants
audience: AI Assistants, LLMs, Autonomous Agents
---

# AGENTS.md — Repository Context for AI Coding Assistants

This document is the authoritative context file for any AI assistant, LLM, or autonomous agent operating within this repository. Read it in full before making any changes.

---

## Repository Identity

**Package name:** `fluksos`  
**npm command:** `npx fluksos@latest`  
**Type:** CLI scaffold generator — not an application, not a library  
**Audience:** Two distinct groups (documented below)

---

## Two Audiences

### End Users (CLI consumers)

End users run `npx fluksos@latest init nextjs my-app ./my-app --tier 3` and expect a production-ready project output. They never see the internals of this repository. Their only interface is the CLI.

From their perspective:
- The CLI is the product.
- The generated project is the output.
- Correctness means: the scaffold runs without errors, all files are present, dependencies install cleanly, and `pnpm dev` starts the app.

### Contributors and Developers

Contributors work inside this repository — they modify scripts, templates, validators, or add new stacks. This is their context document.

From their perspective:
- The CLI is a dispatcher (`bin/cli.js`).
- The stack scripts are the core logic (`stacks/<stack>/scripts/`).
- The templates are the static file sources (`stacks/<stack>/templates/`).
- The tests validate behavior without running the full scaffold.
- Structural changes require a planning workflow (see Slash Commands below).

---

## Repository Structure (What Matters)

The only directories that ship to npm are `bin/` and `stacks/`. All other directories are development tooling.

```
fluksos/
├── bin/
│   └── cli.js                    # CLI entry point — dispatches to stack scripts
├── stacks/
│   └── nextjs/
│       ├── scripts/
│       │   ├── init_project.js            # Full scaffold pipeline (13 steps)
│       │   ├── generate_action.js         # Server Action code generator
│       │   ├── generate_rpc_hook.js       # Hono RPC hook generator
│       │   ├── validate-architecture.js   # AST + filesystem architectural rules
│       │   ├── validate-config.js         # Configuration correctness rules
│       │   ├── validate-ui-state.js       # UI boundary rules
│       │   └── lib/
│       │       ├── ast-parser.js          # Core AST rule engine
│       │       └── report-generator.js    # Violation report formatter
│       ├── templates/
│       │   ├── root/                      # Workspace-level config (biome.json, lefthook.yml, .github/workflows)
│       │   ├── app-common/                # Always applied — base Next.js files
│       │   ├── app-tier-2/                # Applied for --tier 2 and --tier 3
│       │   │   └── src/
│       │   │       └── lib/
│       │   │           ├── safe-action.ts # Rate-limited server actions
│       │   │           └── rate-limit.ts  # Upstash Redis limiter
│       │   ├── app-tier-3/                # Applied for --tier 3 only
│       │   │   ├── Dockerfile             # Standalone production build
│       │   │   └── docker-compose.yml     # Local pgvector database
│       │   ├── observability/             # Docker Compose, OTel, Grafana Faro, Loki, Tempo, Pyroscope
│       │   └── tests/                     # K6 performance test scripts
│       ├── tests/
│       │   ├── mock-project/              # Fixture project for unit tests (not shipped)
│       │   ├── validate-architecture.test.js
│       │   ├── validate-config.test.js
│       │   └── validate-ui-state.test.js
│       └── README.md                      # Stack-specific developer notes
├── package.json                           # npm package — bin entry, files field
├── README.md                              # Public documentation
└── AGENTS.md                              # This file
```
---

## How the CLI Works

`bin/cli.js` is the sole entry point. It:

1. Prints the ASCII logo on every invocation.
2. Parses the first positional argument as a command (`init`, `generate`, `validate`).
3. Looks up the command's target in `STACK_REGISTRY` — an in-file object that maps stack names to their script files, generators, validators, and tiers.
4. Calls `runScript(stack, scriptName, args)`, which resolves the absolute path to the script under `stacks/<stack>/scripts/` and delegates execution via `execSync`.

To add a new stack, create its directory under `stacks/` and add a registration entry to `STACK_REGISTRY` in `bin/cli.js`. No other file requires modification for routing.

---

## How `init_project.js` Works

The init script executes a **linear, ordered pipeline** of 13 async steps via `runStep()`. Each step is named, logged with `[STEP]`, and will `process.exit(1)` on failure.

| Step | Function | Description |
|------|----------|-------------|
| 1 | `assertPreconditions` | Verifies `pnpm` is available in PATH |
| 2 | `assertRequiredTemplateFiles` | Validates all template files exist before any writes |
| 3 | `createWorkspaceRoot` | Creates directories, writes `package.json`, `turbo.json`, `pnpm-workspace.yaml` |
| 4 | `scaffoldNextApp` | Runs `pnpm dlx create-next-app` with strict flags, then removes generated lock files |
| 5 | `cleanupGeneratedFiles` | Removes ESLint, Prettier, Babel, Tailwind config files from the generated app |
| 6 | `installCommonDependencies` | Installs workspace-level devDeps, then app-level deps (Next.js, React, Biome, etc.) |
| 7 | `installTierDependencies` | Installs tier-specific packages (Zustand/Valibot for Tier 2, Drizzle/Better Auth for Tier 3) |
| 8 | `applyRootTemplates` | Copies `templates/root/` into workspace root (skips already-written `package.json`, `turbo.json`, `pnpm-workspace.yaml`) |
| 9 | `applyCommonAppTemplates` | Copies `templates/app-common/` into `apps/web/` |
| 10 | `applyTierTemplates` | Copies `templates/app-tier-2/` and/or `templates/app-tier-3/` based on tier |
| 11 | `applyObservabilityTemplates` | Copies `templates/observability/` and `templates/tests/` |
| 12 | `setupLefthook` | Runs `git init` (if needed) and `pnpm exec lefthook install` |
| 13 | `validateScaffold` | Asserts required output files exist and banned files do not exist |

All dependency versions are pinned in the `STACK_VERSIONS` constant at the top of the file. When updating versions, only edit that object — do not hardcode versions inline.

---

## How the Generators Work

The `fluksos generate` commands are vital for maintaining the architectural boundaries set by `init_project.js`. AI assistants **MUST** recommend and use these generators instead of manually writing boilerplate.

1. **`generate_action.js`**: Generates a `next-safe-action` wrapped with a `valibot` schema.
   - *AI Rule:* Never manually write a Server Action without input validation in this stack. Always use the generator to ensure the `safe-action.ts` rate limiter is correctly applied.
2. **`generate_rpc_hook.js`**: Generates a React Hook wrapping the Hono RPC Client using `TanStack Query`.
   - *AI Rule:* Never call `rpc.api.*` directly inside a React component (e.g., inside `useEffect`). Always use this generator to create a proper caching layer.

---

## How the Validators Work

Validators are standalone scripts that can be run independently or via the CLI. They import from shared libraries:

- **`lib/ast-parser.js`** — The core rule engine. Reads each TypeScript/JavaScript/CSS file and applies regex-based AST rules. Returns an array of violation objects (`{ file, rule, message, severity }`).
- **`lib/report-generator.js`** — Formats violations into a human-readable report and writes to stdout.

Severity levels:
- `FATAL` — Exits with code `1`. Blocks CI.
- `ERROR` — Reported but does not exit.

Validators exit with code `1` if any violations are present. This makes them suitable for `pre-commit` hooks and CI gates.

**Current validator scripts:**

| Script | Responsibility |
|--------|----------------|
| `validate-architecture.js` | Tooling integrity (Biome required, ESLint banned), App Router strictness, AEO/SEO ecosystem, Tailwind v4 compliance, full AST rule pass via `ast-parser.js` |
| `validate-config.js` | Configuration file correctness — `tsconfig.json`, `biome.json`, `next.config.ts`, `vitest.config.ts` structure |
| `validate-ui-state.js` | UI state boundary rules — Zustand isolation, Server/Client Component boundary integrity |

---

## AST Rules Catalogue

Rules enforced by `ast-parser.js` on every `.ts`, `.tsx`, `.js`, and `.css` file:

| Rule ID | Severity | Description |
|---------|----------|-------------|
| `missing-server-only` | FATAL | `src/services/`, `src/actions/`, `src/db/`, `fetcher.ts` must import `server-only` |
| `deprecated-unstable-cache` | FATAL | `unstable_cache` is banned — use `"use cache"` directive (Next.js 16) |
| `deprecated-force-cache` | FATAL | `fetch({ cache: "force-cache" })` is banned |
| `banned-ts-ignore` | FATAL | `// @ts-ignore` is globally banned |
| `banned-react-context` | FATAL | `createContext` from React is banned — use Zustand |
| `banned-redux` | FATAL | `react-redux` and `@reduxjs/toolkit` are banned |
| `banned-md5` | FATAL | `MD5` is banned — use SCRAM-SHA-256 or secure hashing |
| `db-in-controller` | FATAL | Controllers/Pages must not import `db` directly — use Repositories |
| `banned-axios` | FATAL | `axios` is banned — use `fetcher.ts` or Hono RPC |
| `use-client-in-page` | FATAL | `page.tsx` must be a React Server Component |
| `business-logic-in-page` | ERROR | Route handler mutations in `page.tsx` must use Server Actions |
| `db-in-page` | FATAL | Direct `db` import in `page.tsx` is banned |
| `static-metadata-in-dynamic-route` | FATAL | Dynamic routes must use `generateMetadata()`, not `export const metadata` |
| `json-ld-in-client` | FATAL | JSON-LD must be in Server Components (LLM crawlers cannot execute JS) |
| `db-in-client-component` | FATAL | `db` import in `"use client"` components is banned |
| `banned-csr-fetch` | FATAL | `fetch()` or `axios()` inside `useEffect` is banned — use TanStack Query |
| `remote-data-in-zustand` | FATAL | Zustand stores must only contain UI state — no `fetch`, `useQuery`, or `db` |
| `insecure-auth-config` | FATAL | Better Auth must have `requireEmailVerification: true` |
| `fake-sitemap-date` | FATAL | `lastModified: new Date()` in `sitemap.ts` is banned — use real `updatedAt` from the database |
| `css-import-url` | FATAL | `@import url()` in `globals.css` is banned — use `next/font` |
| `missing-alt-text` | FATAL | `<img>` and `<Image>` must have meaningful `alt` text |
| `banned-custom-primitive-ui` | FATAL | Do not build Button/Input/Card/Modal/Dialog primitives — use Shadcn UI |
| `legacy-uuid` | WARNING | `uuidv4()` used where `uuidv7()` is expected for PKs |

**File-level rules (in `validate-architecture.js`):**

| Rule ID | Severity | Description |
|---------|----------|-------------|
| `no-eslint` | FATAL | `.eslintrc.*` presence is banned |
| `missing-biome` | FATAL | `biome.json` must exist at workspace root |
| `deprecated-middleware` | FATAL | `middleware.ts` is banned — use `proxy.ts` |
| `banned-pages-router` | FATAL | `src/pages/` directory is banned |
| `missing-llms-txt` | FATAL | `/llms.txt` route must exist (AEO) |
| `missing-llms-full-txt` | FATAL | `/llms-full.txt` route must exist |
| `static-robots-txt` | FATAL | `public/robots.txt` is banned — use `src/app/robots.ts` |
| `static-sitemap-xml` | FATAL | `public/sitemap.xml` is banned — use `src/app/sitemap.ts` |
| `legacy-tailwind-config` | FATAL | `tailwind.config.*` is banned — configure via `globals.css` |
| `legacy-postcss-config` | FATAL | `postcss.config.js` is banned — use `.mjs` |
| `missing-opengraph-image` | ERROR | Marketing `page.tsx` must have a co-located `opengraph-image.*` |

---

## Testing (Developer Context)

Tests live at `stacks/nextjs/tests/` and use a shared fixture project at `stacks/nextjs/tests/mock-project/`.

The `mock-project/` directory simulates a real project output with intentional violations for negative testing. It is **not a generated output and not representative of what Fluksos produces** — it exists to provide test fixtures for the validator scripts.

To run tests:

```bash
cd stacks/nextjs/scripts
pnpm test
```

Tests import the validator functions directly and run them against the `mock-project/` fixture, asserting the presence or absence of specific rule violations.

---

## Constraints for AI Assistants

1. **Do not modify `STACK_VERSIONS` in `init_project.js` without explicit instruction.** These versions are pinned intentionally and have been tested together. A version bump requires running the full scaffold integration test before merging.

2. **Do not add `console.log` to validator scripts.** All output must go through `report-generator.js`. Validators are consumed programmatically in tests.

3. **Do not commit directly to `main`.** All changes must be on a feature branch opened via pull request.

4. **Structural changes require a `/implementation-plan` cycle.** A structural change is any modification that adds a new command, a new stack, a new validator, or changes the pipeline step order in `init_project.js`.

5. **Templates are static files, not code generators.** When editing files under `stacks/nextjs/templates/`, you are editing what the end user will receive verbatim. Do not introduce template variables or placeholder strings. What is in the template is what gets copied.

6. **The `mock-project/` fixture is intentionally broken.** It contains architectural violations by design. Do not "fix" it — the tests depend on those violations being present.

7. **Do not remove or weaken Security Headers in `next.config.ts`.** The scaffold is designed to ship with a strict Content-Security-Policy (CSP) and HSTS. If asked to add external scripts or styles, append them to the existing CSP directives instead of removing the headers.

8. **K6 Test Generation:** The Fluksos ecosystem uses k6 v2.0+ which natively supports MCP. When generating or editing test scripts for the `tests/` directory, always use the new Assertion API `expect()` instead of the legacy `check()` function.

---

## Adding a New Stack

1. Create `stacks/<stack-name>/scripts/init_project.js`.
2. Create `stacks/<stack-name>/templates/` with at minimum a `root/` subdirectory.
3. Add a registration entry to `STACK_REGISTRY` in `bin/cli.js`.
4. Add at least one validator script and register it in the stack's `validators` array.
5. Create `stacks/<stack-name>/tests/` with fixtures and test files.
6. Update `README.md` — move the stack from the Roadmap table to the Available Stacks table.

---

## Key Design Decisions

**Why not a template repository / GitHub template?**  
Templates go stale. Fluksos regenerates from pinned versions on every run, ensuring the output is always current.

**Why `pnpm` only?**  
Turborepo workspace support and strict dependency isolation. The `pnpm-workspace.yaml` and `ignoredBuiltDependencies` configuration require `pnpm`. No other package manager is tested or supported.

**Why `proxy.ts` instead of `middleware.ts`?**  
Next.js 16.2+ removed the `middleware.ts` convention. `proxy.ts` is the architectural replacement for edge-level routing. The validator enforces this.

**Why regex-based AST analysis instead of a real AST parser?**  
Speed and zero dependencies. The validator runs on pre-commit hooks where latency matters. The current rules are precise enough to catch the patterns they target without parsing a full AST tree.

**Why are all dependency versions pinned?**  
Determinism. Two engineers running `npx fluksos@latest init` on different dates must get identical lockfiles. Version ranges defeat this guarantee.

---

## Internal Governance & CD (Dogfooding)

Fluksos uses **Biome**, **Lefthook**, and **GitHub Actions** to enforce its own source code quality, and **Changesets** for Semantic Versioning and Continuous Deployment. As an AI assistant, you must respect these boundaries:

1. The `biome.json` at the repository root strictly ignores `stacks/*/templates/**` and `stacks/*/tests/mock-project/**`. **Do not attempt to format templates**. Templates are the literal string output sent to the user and may contain intentional intermediate states.
2. If you modify `bin/` or `stacks/*/scripts/`, your code will be checked by Biome on `pre-commit` hooks via Lefthook. Ensure your generated code is clean and follows A.N.T (Aesthetics, Naming, Types) aesthetics.
3. The Vitest architecture tests run on `pre-push` and in the CI pipeline. Never bypass the CI validation.
4. **Publishing**: Fluksos uses a CI/CD pipeline with NPM Provenance. If you are instructed to bump a version or release a feature, you MUST use `pnpm exec changeset` to generate a changelog file. Do not manually edit the `version` field in `package.json`. The GitHub Action `.github/workflows/publish.yml` handles the actual publication to NPM when the "Version Packages" PR is merged.

---
> Source: [pasqualinigui/fluksos](https://github.com/pasqualinigui/fluksos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
