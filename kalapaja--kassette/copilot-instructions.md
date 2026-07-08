## kassette

> Kassette is the payment page SPA for Kalatori. Merchants embed it (or serve it from their Kalatori daemon at `/public/`). It renders an invoice, connects a wallet via Reown AppKit/wagmi, and orchestrates token swaps (Uniswap) or cross-chain bridges (Across) to pay in the invoice's target asset. No backend of its own — all state comes from the Kalatori daemon's HTTP API and on-chain reads.

# Kassette — AI Agent Guide

Kassette is the payment page SPA for Kalatori. Merchants embed it (or serve it from their Kalatori daemon at `/public/`). It renders an invoice, connects a wallet via Reown AppKit/wagmi, and orchestrates token swaps (Uniswap) or cross-chain bridges (Across) to pay in the invoice's target asset. No backend of its own — all state comes from the Kalatori daemon's HTTP API and on-chain reads.

## How We Work

### Development Policy

- **Dagger for all CI**: use `dagger call checks` for fast validation, `dagger call end-to-end` for E2E, or run individual checks. Pipeline defined in TypeScript at `.dagger/src/index.ts`
- **Layer-cached builds**: Dagger caches pnpm install separately from source. Only code changes trigger rebuilds, not dependency re-downloads
- **Git hooks**: lefthook auto-installs via `pnpm install` (the `prepare` script). Pre-commit runs lint + format on staged files; pre-push runs typecheck + tests + tag version check
- **Conventional commits**: enforced by commitlint in the commit-msg hook

### Documentation Policy

- **Done something — write it down.** Every architectural decision, troubleshooting finding, or pattern change gets recorded in `docs/`.
- When a doc contradicts code, ask the user which is correct and update the other.
- See [docs/doc-update-triggers.md](docs/doc-update-triggers.md) for the mandatory update checklist.

### Research Policy

- **Never assume library APIs from memory** — look up in Context7 or Exa first.

### Trust Hierarchy

When sources disagree, trust in this order:

1. **Code, build files, CI workflows** — canonical source of truth
2. **Tests** — verify behavior claims
3. **Specific docs** (e.g., `docs/testing-strategy.md`) override general docs (e.g., `AGENTS.md`)
4. **Docs describe intent and patterns** — not guaranteed implementation truth
5. When unsure, **ask the user** which is correct and update the other

### Editing Strategy

- Prefer minimal, surgical edits. Don't refactor adjacent code opportunistically.
- Add or update tests when behavior changes.
- Check [docs/doc-update-triggers.md](docs/doc-update-triggers.md) after changes.

### Writing Style

- Context-aware, terse, informative, concise.
- No unnecessary abstractions — three similar lines > premature helper.

## Commands

```bash
# Dagger (preferred — reproducible, cached):
dagger call checks                  # lint + format + typecheck + test + audit + build (~60s)
dagger call lint                    # ESLint only
dagger call format-check            # Prettier only
dagger call typecheck               # tsc --noEmit
dagger call test                    # Vitest with coverage thresholds
dagger call audit                   # pnpm audit (blocking on critical)
dagger call audit-advisory          # pnpm audit (high/moderate, advisory)
dagger call build                   # Production Angular build → dist/browser
dagger call end-to-end              # Playwright E2E against production build
dagger call release-zip --version=X.Y.Z  # SRI-patched release ZIP

# Local pnpm (same commands Dagger runs):
pnpm dev                            # Dev server on :3001 with MSW mocks
pnpm build                          # Production build
pnpm test                           # Vitest
pnpm test:coverage                  # Vitest with coverage + thresholds
pnpm lint                           # ESLint (zero warnings)
pnpm lint:fix                       # ESLint autofix
pnpm format                         # Prettier write
pnpm format:check                   # Prettier check
pnpm e2e                            # Playwright (starts dev server automatically)
pnpm e2e:ui                         # Playwright interactive UI mode

# Release:
pnpm release:tag                    # Create signed tag from package.json version
```

## Architecture

- **Entry point:** `src/main.ts` — bootstraps Angular app, starts MSW iff `environment.mocks`. `environment.ts` (dev) sets `mocks: true`; `environment.prod.ts`, `environment.e2e.ts`, and `environment.no-mocks.ts` set `mocks: false`. The `no-mocks` build config pairs with `pnpm dev:no-mocks` for manual testing against a real backend without production optimizations. E2E uses `production: true` so the bundle ships without MSW — Playwright handles mocking at the network layer instead.
- **Root component:** `src/app/app.component.ts` — shell with `<router-outlet>`
- **Main page:** `src/app/pages/payment/payment-layout.component.ts` — THE main component with all step rendering via `@switch`
- **Components:** `src/app/components/` — 11 standalone Angular components, prefixed with `kp-` (e.g., `kp-button`, `kp-input`, `kp-webview-escape`)
- **Services:** `src/app/services/` — 14 injectable services (AppKit, WalletState, PaymentState, Invoice, Balance, Payment, Price, Token, Quote, Uniswap, Across, PendingTx, Translation, Layout)
- **Config:** `src/app/config/` — chains, tokens, uniswap, across (plain TypeScript modules)
- **Types:** `src/app/types/` — invoice, payment, payment-step types
- **i18n:** `src/app/i18n/` — custom signal-based translation system (en/es locales)
- **Mocks:** `src/mocks/` — MSW handlers for dev environment

## State Management

`PaymentStateService` is the central state machine with `VALID_TRANSITIONS` map and signal-based context. 12 payment steps, 19 context signals, 6 computed signals. State drives rendering via `@switch(state.currentStep())` in `PaymentLayoutComponent`.

## Theme System

`src/styles/theme.css` defines design tokens as CSS custom properties on `:root`. `src/styles/font.css` contains the Alaska font `@font-face` with base64 WOFF2 blob. Both are global styles registered in `angular.json`.

- **Color space:** OKLCH throughout — brand colors from Figma, semantic tokens following shadcn/ui naming
- **Font:** Alaska is the **only** typeface. Always reference `var(--font-family)` which resolves to `"Alaska", sans-serif`. No exceptions.
- **Light mode only** currently

## Key Conventions

- **Angular components:** Standalone (default), signal-based inputs (`input()`), outputs (`output()`), new control flow (`@if`, `@for`, `@switch`)
- **Zoneless:** No Zone.js — change detection driven by signals. Timer callbacks use `NgZone.run()`.
- **Import alias:** `@/` maps to `./src/`
- **CSS:** ViewEncapsulation.Emulated (Angular default). Component CSS copied verbatim from Lit source with `:host` attribute reflection.
- **Host attributes:** Components reflect signal inputs as host attributes for CSS variant selectors (e.g., `host: { '[attr.weight]': 'weight()' }`)
- **HTTP:** Angular `HttpClient` with `withFetch()` for all HTTP requests
- **Blockchain:** wagmi/core actions + viem for contract interactions, Reown AppKit for wallet modal
- **Tests:** Vitest with `describe`/`it`/`expect` (no Angular TestBed for pure logic services)
- **TypeScript:** Strict mode, ES2022 target. Config in `tsconfig.json`.

## Tech Stack

| Component          | Library / Tool                 | Notes                                                       |
| ------------------ | ------------------------------ | ----------------------------------------------------------- |
| Framework          | Angular 21                     | Standalone components, signal inputs, zoneless (no Zone.js) |
| Build              | Angular CLI + esbuild          | `@angular/build:application` builder                        |
| Package manager    | pnpm 10.32.1                   | Corepack-managed                                            |
| Wallet connection  | Reown AppKit + wagmi/core      | WalletConnect, MetaMask SDK, Coinbase Wallet                |
| Blockchain types   | viem                           | Chain definitions, ABI encoding, contract reads             |
| DEX swaps          | Uniswap v3                     | Via quoter + router contracts                               |
| Cross-chain bridge | Across Protocol                | For cross-chain token transfers                             |
| State management   | Angular signals                | `PaymentStateService` — 12-step state machine               |
| Styling            | Tailwind CSS 4                 | PostCSS plugin, OKLCH color space                           |
| i18n               | Custom signal-based            | `src/app/i18n/` — en/es locales                             |
| HTTP               | Angular HttpClient             | `withFetch()` provider                                      |
| Unit tests         | Vitest 4 + @vitest/coverage-v8 | Coverage thresholds enforced                                |
| Property tests     | fast-check                     | Number formatting invariants                                |
| E2E tests          | Playwright                     | Chromium, network-level route mocking                       |
| Linting            | ESLint (flat config)           | @angular-eslint + typescript-eslint + prettier              |
| Formatting         | Prettier                       | Enforced in pre-commit hook                                 |
| Commit linting     | commitlint                     | @commitlint/config-conventional                             |
| Git hooks          | lefthook                       | Pre-commit, commit-msg, pre-push                            |
| Dev mocking        | MSW (Mock Service Worker)      | Browser service worker for dev server                       |
| CI pipeline        | Dagger (TypeScript SDK)        | Version pinned in `.tool-versions`                          |

## Repository Layout

```
src/                                # Angular source
├── main.ts                         # Bootstrap, conditional MSW start
├── app/
│   ├── app.component.ts            # Root shell with <router-outlet>
│   ├── app.config.ts               # Provider configuration (HttpClient, Router)
│   ├── app.routes.ts               # Route definitions with invoice guard
│   ├── components/                 # 11 standalone components (kp-* prefix)
│   ├── pages/payment/
│   │   ├── payment-layout.component.ts   # THE main component (12-step @switch)
│   │   ├── payment-layout.component.html # Template
│   │   ├── payment-layout.component.css  # Styles
│   │   ├── payment-layout.execution.spec.ts  # Execution flow tests
│   │   ├── payment-layout.recovery.spec.ts   # Recovery flow tests
│   │   └── guards/invoice.guard.ts       # Route guard (requires invoice_id)
│   ├── services/                   # 14 injectable services
│   ├── config/                     # Chain, token, DEX, bridge configuration
│   ├── types/                      # Invoice, payment, swap types
│   ├── i18n/                       # Translation system + locales
│   ├── testing/test-harness.ts     # Shared createComponentHarness() helper
│   ├── pipes/translate.pipe.ts     # Translation pipe
│   └── utils/                      # Utility functions
├── environments/                   # Build-time environment configs
│   ├── environment.ts              # Dev (MSW enabled)
│   ├── environment.prod.ts         # Production
│   └── environment.e2e.ts          # E2E (production-like, no MSW)
├── mocks/                          # MSW handlers for dev server
└── styles/                         # Global CSS (theme tokens, font)

tests/e2e/                          # Playwright E2E specs
.dagger/src/index.ts                # Dagger CI pipeline (TypeScript)
.githooks/pre-push-tag-check.sh     # Tag/version validation hook
.github/
├── actions/setup-dagger/           # Composite action for CI
├── workflows/
│   ├── ci.yml                      # 6-job parallel matrix via Dagger
│   ├── release.yml                 # Build ZIP + upload to GitHub release
│   ├── version-bump.yml            # Auto version-bump PR on tag push
│   ├── codeql.yml                  # CodeQL security analysis
│   ├── semgrep.yml                 # Semgrep SAST
│   └── gitleaks.yml                # Secret scanning
└── dependabot.yml                  # Dependency update automation
```

## Documentation Map

| Doc                            | Covers                                                            | When to consult                                 |
| ------------------------------ | ----------------------------------------------------------------- | ----------------------------------------------- |
| `docs/testing-strategy.md`     | Test tiers, coverage thresholds, test harness, adding tests       | Adding tests, understanding test infrastructure |
| `docs/release-strategy.md`     | Tag-first release model, signed tags, ZIP artifact, version bumps | Releasing, version management                   |
| `docs/dagger-ci-guide.md`      | Dagger commands, caching strategy, CI architecture, debugging     | CI changes, pipeline debugging                  |
| `docs/doc-update-triggers.md`  | Mandatory doc update checklist                                    | After any code change                           |
| `docs/testing-payment-flow.md` | MSW-mocked manual testing scenarios                               | Manual QA, dev server testing                   |

## MCP Tooling Summary

| Tool                    | Use for                                                                            |
| ----------------------- | ---------------------------------------------------------------------------------- |
| **Ripgrep**             | Fast code search (preferred over grep via Bash)                                    |
| **Context7**            | Library docs (Angular, wagmi, viem, Playwright). Always check before assuming APIs |
| **Exa**                 | Online search, code samples, best practices                                        |
| **mcp-server-git**      | Git read ops. Bash for complex git                                                 |
| **Playwright**          | Browser automation for local testing                                               |
| **Sequential Thinking** | Step-by-step reasoning for complex problems                                        |

**IMPORTANT**: Ripgrep MCP is extremely efficient; use it as a find/grep replacement whenever possible.
**IMPORTANT**: Use `dagger call checks` for fast checks. Use Dagger MCP tools (`dagger-tools`) for programmatic access when available.

## Key Links

- [V2 API spec](https://github.com/Kalapaja/kalatori-api/blob/master/kalatori.yaml)
- [Kalatori Matrix](https://matrix.to/#/#Kalatori-support:matrix.zymologia.fi)
- [GitHub Discussions](https://github.com/Kalapaja/Kalatori/discussions)
- [Roadmap](https://github.com/orgs/Kalapaja/projects/2) and [Milestones](https://github.com/Kalapaja/Kalatori/milestones)

---
> Source: [Kalapaja/Kassette](https://github.com/Kalapaja/Kassette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
