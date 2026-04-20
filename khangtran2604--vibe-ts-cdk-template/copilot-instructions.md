## vibe-ts-cdk-template

> CLI scaffolding tool (like create-t3-app for AWS). Generates a full-stack TypeScript monorepo: CDK + Lambda + API Gateway + Vite React + Cognito. Three presets: minimal, standard, full.

# CLAUDE.md — vibe-ts-cdk-template

## What This Is

CLI scaffolding tool (like create-t3-app for AWS). Generates a full-stack TypeScript monorepo: CDK + Lambda + API Gateway + Vite React + Cognito. Three presets: minimal, standard, full.

## Stack

- **Runtime**: Node 24.x LTS, pnpm
- **CLI framework**: `commander` (args) + `@clack/prompts` (interactive UI)
- **Build**: `tsup` (ESM, shebang banner)
- **Test**: `vitest`
- **Templating**: No engine — `String.replaceAll("{{key}}", value)` + `// @feature:X` conditional lines

## Package Versions — Always Use Latest Stable

**Before adding or updating any dependency, run `npm view {package_name} version` to get the latest stable version.** Never hardcode outdated versions. Never guess — always verify.

### CLI tool dependencies (this repo)

| Package | Latest Stable |
|---------|---------------|
| `commander` | 14.0.3 |
| `@clack/prompts` | 1.0.1 |
| `tsup` | 8.5.1 |
| `vitest` | 4.0.18 |
| `typescript` | 5.9.3 |
| `tsx` | 4.21.0 |

### Generated project dependencies (templates)

| Package | Latest Stable |
|---------|---------------|
| `aws-cdk-lib` | 2.240.0 |
| `constructs` | 10.5.1 |
| `@types/aws-lambda` | 8.10.161 |
| `hono` | 4.12.3 |
| `@hono/node-server` | 1.19.9 |
| `react` | 19.2.4 |
| `vite` | 7.3.1 |
| `@vitejs/plugin-react` | 5.1.4 |
| `react-router` | 7.13.1 |
| `@playwright/test` | 1.58.2 |
| `eslint` | 10.0.2 |
| `prettier` | 3.8.1 |
| `turbo` | 2.8.12 |
| `husky` | 9.1.7 |
| `lint-staged` | 16.3.1 |
| `supertest` | 7.2.2 |

> **These versions are a snapshot (2026-03-02).** Always re-verify with `npm view <pkg> version` before use — they may have updated.

## Project Layout

```
src/                    # CLI source
  index.ts              # Entry: commander + clack + scaffold orchestration
  prompts.ts            # All clack interactive prompts
  scaffolder.ts         # Core: copy templates, apply transforms, git, pnpm
  template-helpers.ts   # Feature flags → template dirs, variable maps, conditionals
  presets.ts            # Preset → FeatureFlags mapping
  types.ts              # ProjectConfig, Preset, FeatureFlags
  module-prompts.ts     # Interactive prompts for `module` subcommand
  module-generator.ts   # Core module generation engine
  module-context.ts     # Project context detection (ports, project name)
  module-helpers.ts     # String transforms, variable maps, injection helpers
  constants.ts          # CLI name, version
  utils/                # fs, git, pnpm, logger helpers
templates/              # Template files copied into generated projects
  base/                 # Root configs (all presets)
  infra/                # CDK stacks
  services/             # health, users microservices
  dev-gateway/          # Local proxy
  packages/             # Shared packages (lambda-utils, types, utils, eslint, tsconfig)
  frontend/             # Vite + React (standard+)
  auth/                 # Cognito authorizer (standard+)
  e2e/                  # Playwright (standard+)
  database/             # DynamoDB + RDS (full)
  cicd/                 # GitHub Actions (full)
  monitoring/           # CloudWatch (full)
  extras/               # Husky + lint-staged (full)
test/                   # CLI tests
```

## Commands

```bash
pnpm install            # Install deps
pnpm build              # Build CLI with tsup
pnpm test               # Run vitest
pnpm dev                # Dev mode (tsup --watch or tsx)

# Module generation (run from inside a scaffolded project)
node dist/index.js module <name>        # Generate a CRUD module interactively
node dist/index.js module orders -y     # Generate with defaults (no prompts)
node dist/index.js module orders --no-install  # Skip pnpm install
```

## Template Conventions

| Convention | Example | Purpose |
|------------|---------|---------|
| `_` prefix for dotfiles | `_gitignore` → `.gitignore` | Avoids tooling issues |
| `.hbs` suffix | `package.json.hbs` → `package.json` | Marks files with `{{variable}}` placeholders |
| `{{variable}}` | `{{projectName}}`, `{{awsRegion}}` | Simple string replacement |
| `// @feature:X` | `// @feature:frontend import {...}` | Line included if feature enabled, removed if not |

## Key Implementation Rules

- **tsup target**: `node24` (match current LTS)
- **Template resolution**: `path.resolve(__dirname, "..", "templates")` — dist/index.js is one level below root
- **package.json `files` field**: Must include `["dist", "templates"]`
- **`pnpm-workspace.yaml`**: Built programmatically in scaffolder (not a template) — workspace entries vary by preset
- **Service ports**: health=3001, users=3002 — auto-assigned by index
- **Hono is devDependency only** — not shipped to Lambda
- **Lambda handlers are the primary code** — Hono dev-servers are wrappers for local dev only

## Do NOT

- Use outdated package versions — always run `npm view {pkg} version` to verify latest stable before adding any dependency
- Use a template engine (handlebars, ejs, etc.) — use simple `replaceAll`
- Add CLI subcommands beyond `module` without owner approval
- Make `pnpm-workspace.yaml` a template — build it programmatically
- Add Hono as a production dependency in service templates
- Create a single monolithic CDK stack — use stack-per-service pattern
- Share API Gateways between services — each service gets its own

## Verification

After any significant change, verify the full loop:

```bash
pnpm build && node dist/index.js --preset minimal -y
pnpm build && node dist/index.js --preset standard -y
pnpm build && node dist/index.js --preset full --rds -y
# Then in generated project: pnpm install && pnpm build && pnpm test && pnpm dev

# Module generation (run from within a generated project directory)
pnpm build && cd <generated-project> && node <cli-path>/dist/index.js module orders -y
```

## Implementation Phases

See PLAN.md for detailed phases. Current phase order:
1. Bootstrap CLI project (package.json, tsup, entry point)
2. Interactive prompts (clack flow → ProjectConfig)
3. Scaffolding engine + base templates
4. Minimal preset templates
5. Standard preset templates (frontend, auth, e2e)
6. Full preset templates (database, cicd, monitoring, hooks)
7. Testing & polish

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khangtran2604) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
