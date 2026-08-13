## rebel-app

> Guidance for AI agents and human contributors working on this repository.

# AGENTS – Mindstone Rebel

Guidance for AI agents and human contributors working on this repository.

> **Public-mirror note:** This is the public-mirror version of `AGENTS.md`. It is maintained separately from Mindstone's internal development copy so internal workflow details do not ship in the OSS repo. References to `docs/plans/`, `coding-agent-instructions/`, or `private/` may point to internal-only material; treat them as context, not required public dependencies.


## Project Overview

- **App**: User-friendly agentic-AI Electron app powered by **Rebel Core**, Rebel's native in-process agent runtime (direct provider APIs + MCP, no agent subprocess for the main turn). See [REBEL_CORE](docs/project/REBEL_CORE.md) and [ARCHITECTURE_AGENT_TURN_EXECUTION](docs/project/ARCHITECTURE_AGENT_TURN_EXECUTION.md) for the execution model.
- **Target audience**: **Non-technical knowledge workers**: executives, product managers, sales and marketing teams, customer-success teams, researchers, and other professionals. Product decisions should be evaluated through that lens. Users think in terms of meeting prep, email triage, research synthesis, and document drafting — not implementation details.
- **Stack**: Electron + React + TypeScript + Vite (via `electron-vite`).
- **Platforms**: Cross-platform desktop (macOS, Windows, Linux), with shared cloud and mobile surfaces. Keep code and user-facing copy platform-agnostic. See [WINDOWS_SUPPORT](docs/project/WINDOWS_SUPPORT.md) for path, process, and packaging gotchas.


## Brand Voice & Product Philosophy

See [BRAND_VOICE](docs/project/BRAND_VOICE.md) for the full voice guide.

Quick summary: Rebel is dry, witty, and self-aware — a capable colleague who happens to be amusing. Bias toward clear over clever, calm over exciting, and useful over impressive. Use that voice when touching UI copy, errors, onboarding, notifications, changelogs, and other user-facing surfaces.


## How to Work

### Default operating loop

1. **Understand the task**: read the relevant README/project docs, nearby `AGENTS.md`, and code before editing.
2. **Plan the smallest useful change**: identify files, validation commands, risks, and rollback path.
3. **Implement surgically**: match existing patterns, preserve contracts, and avoid broad rewrites unless the task explicitly calls for one.
4. **Verify**: run the fastest relevant checks during iteration and the project validators before handing off.
5. **Report clearly**: summarize what changed, what passed, and any remaining blockers.

### AI-agent workflows

For non-trivial work, use a structured workflow even if your local tooling names it differently:

- **Feature/refactor work**: create a staged implementation plan, review boundary impacts, implement one coherent stage at a time, and get a second-model or peer review when available. Internal workflow docs may call this `CHIEF_ENGINEER`.
- **Bug fixes**: reproduce or localize the failure, compare plausible root causes, make a minimal fix, add or update regression coverage, and record the learning. Internal workflow docs may call this `CHIEF_BUGFIXER`.
- **Renames**: enumerate all variants and surfaces before editing. Concept renames are broader than file renames; include UI copy, persisted data, prompts, tests, and docs.

Trivial changes (single-file copy edits, small config tweaks, obvious documentation fixes) can be handled directly.


## Coding Principles Digest

### Mandatory stop-and-check gates

When one of these applies, pause and read the linked project guidance before editing:

- **UI work**: use shared UI components from `src/renderer/components/ui`, follow [UI_OVERVIEW](docs/project/UI_OVERVIEW.md), [UI_CSS_ARCHITECTURE](docs/project/UI_CSS_ARCHITECTURE.md), and [BRAND_VOICE](docs/project/BRAND_VOICE.md). Test light and dark themes when changing visuals.
- **E2E test failures**: read [E2E_TEST_FIXING_GUIDELINES](docs/project/E2E_TEST_FIXING_GUIDELINES.md) and [WHY_E2E_TESTS_ARE_HARD_TO_FIX](docs/project/WHY_E2E_TESTS_ARE_HARD_TO_FIX.md). Do not weaken coverage to make a failure disappear.
- **Version bumps**: follow [CHANGELOG_UPDATE_PROCESS](docs/project/CHANGELOG_UPDATE_PROCESS.md) and update the required changelogs before or alongside the bump.
- **LLM prompt changes**: update eval fixtures and run the relevant eval suite. See [WRITING_EVALS](docs/project/WRITING_EVALS.md).
- **New eval harnesses**: read [WRITING_EVALS](docs/project/WRITING_EVALS.md) in full before designing the harness.
- **Model/provider/auth/routing work**: start with [MODEL_AND_PROVIDER_OVERVIEW](docs/project/MODEL_AND_PROVIDER_OVERVIEW.md).
- **IPC changes**: every new handler needs a Zod contract in `src/shared/ipc/contracts.ts`. See [ARCHITECTURE_IPC](docs/project/ARCHITECTURE_IPC.md).
- **Cross-process or connector boundaries**: check [BOUNDARY_REGISTRY](docs/project/BOUNDARY_REGISTRY.md) and run the boundary-hint tooling if contracts, MCP connectors, IPC schemas, or shared boundary interfaces change.
- **Destructive git/file operations**: never discard, reset, or delete work you did not create. Treat untracked files as user-owned.
- **Secrets**: never commit keys, tokens, DSNs, customer data, or private URLs. See [SECRET_SCANNING](docs/project/SECRET_SCANNING.md).

### General principles

- Prefer simple, reversible changes over clever abstractions.
- Keep business logic in `src/core/` whenever it can be platform-agnostic.
- Use boundary interfaces instead of importing Electron APIs in shared/core code.
- Validate at process boundaries with TypeScript and Zod.
- Do not silently swallow unexpected errors. If recovery is intentional, make it observable with structured logs and user-visible state where appropriate.
- Never log secrets or sensitive user data.
- Keep performance proportional to the user value: avoid unnecessary re-renders, heavy hot-path work, and unbounded scans.

### Logging

- Use structured log calls: `log.warn({ data }, 'message')`, not `log.warn('message', { data })`.
- Use scoped loggers from `src/core/logger.ts`.
- Preserve diagnostic context without leaking tokens, file contents, personal data, or provider payloads.

### Agent message content

`AgentAssistantMessage` text lives in content blocks, not a `.text` property. Use `extractAgentAssistantText()` from `@core/agentRuntimeTypes` instead of reaching into message internals.


## Architecture Notes

### Core-first, surface-aware

Rebel runs across three surfaces:

- **Desktop**: Electron app (`src/main`, `src/preload`, `src/renderer`).
- **Cloud**: standalone Node service (`cloud-service`) that reuses core logic.
- **Mobile**: React Native app (`mobile`) with shared client code.

Default to `src/core/` for business logic so behavior can be shared across surfaces. Only put code in `src/main/` when it genuinely needs Electron APIs. Avoid duplicating core behavior in `cloud-service/` or `mobile/`.

Important boundary interfaces:

- `src/core/platform.ts` — platform paths and app metadata.
- `src/core/storeFactory.ts` — store abstraction.
- `src/core/handlerRegistry.ts` — IPC/HTTP handler registration abstraction.
- `src/core/broadcastService.ts` — event broadcast abstraction.
- `src/core/errorReporter.ts` — error-reporting abstraction.
- `src/core/logger.ts` — structured logging.

### Cross-surface parity

When adding features that touch auth, provider routing, synced settings, secure storage, telemetry, or desktop-connected services, verify desktop/cloud/mobile behavior explicitly. See [CROSS_SURFACE_PARITY_CHECKLIST](docs/project/CROSS_SURFACE_PARITY_CHECKLIST.md) and [CROSS_SURFACE_PARITY_TRAP_CATALOGUE](docs/project/CROSS_SURFACE_PARITY_TRAP_CATALOGUE.md).

### App.tsx architecture note

`src/renderer/App.tsx` is intentionally a large orchestration file. Prefer adding feature-specific logic under `src/renderer/features/<feature>/hooks/`, independent UI state under `src/renderer/hooks/`, and only keep cross-cutting orchestration in `App.tsx` when callback injection or a feature hook is not sufficient.


## Documentation

Evergreen developer docs live under `docs/project/`. Planning docs live under `docs/plans/`. User-facing bundled help lives under `rebel-system/help-for-humans/`.

Before creating new docs, check whether a public doc already covers the topic and link to the single source of truth instead of duplicating it. Focus docs on intent, design decisions, invariants, and signposts to code.

Useful starting points:

- [ARCHITECTURE_OVERVIEW](docs/project/ARCHITECTURE_OVERVIEW.md) — system architecture, processes, and data flows.
- [TESTING_AUTOMATION_OVERVIEW](docs/project/TESTING_AUTOMATION_OVERVIEW.md) — unit/integration tests and validation commands.
- [BUILD_AND_RELEASE_OVERVIEW](docs/project/BUILD_AND_RELEASE_OVERVIEW.md) — packaging, CI, distribution, and release docs.
- [UI_OVERVIEW](docs/project/UI_OVERVIEW.md) — renderer layout, UI patterns, and design-system links.
- [UI_CONVERSATIONS](docs/project/UI_CONVERSATIONS.md) — transcript, message cards, auto-scroll, and turn IDs.
- [SAFETY_SYSTEM_OVERVIEW](docs/project/SAFETY_SYSTEM_OVERVIEW.md) — tool safety, memory safety, shell safety, evals, and tests.
- [WRITING_EVALS](docs/project/WRITING_EVALS.md) — LLM eval harnesses and fixture practices.
- [CODING_PRINCIPLES](docs/project/CODING_PRINCIPLES.md) — project coding principles.
- [SPACES](docs/project/SPACES.md) — spaces, organization grouping, and frontmatter model.
- [MCP_IMPROVEMENT_WORKFLOW](docs/project/MCP_IMPROVEMENT_WORKFLOW.md) — MCP development entry point.
- [MCP_OSS_CATALOG_VERSION_AUDIT](docs/project/MCP_OSS_CATALOG_VERSION_AUDIT.md) and [MCP_OSS_PACKAGE_MANUAL_UPDATE](docs/project/MCP_OSS_PACKAGE_MANUAL_UPDATE.md) — maintaining OSS MCP package versions.
- [SETUP_DEVELOPMENT_ENVIRONMENT](docs/project/SETUP_DEVELOPMENT_ENVIRONMENT.md) — local prerequisites and configuration.
- [PRODUCT_VISION_FEATURES](docs/project/PRODUCT_VISION_FEATURES.md) — product overview and feature summary.


## Build & Run

- **First-time setup (fresh clone)**: `npm run setup` — one-shot, cross-platform bootstrap (submodule init with retry, `npm ci`, super-mcp + bundled-MCP builds, `.env.local` scaffold) with fail-loud prerequisite checks. Then `npm run dev`.
- **Install** (refresh dependencies only): `npm ci`.
- **Dev**: `npm run dev` (full predev cycle + Electron/Vite dev server).
- **Quick restart**: `npm start` (skips predev; use when bundles are already built).
- **Build**: `npm run build`.
- **Lint**: `npm run lint`.
- **Strict TypeScript check**: `npm run lint:ts`.
- **Package for local testing**: `npm run package`.
- **Create installer artifacts**: `npm run make`.

Prefer existing scripts over new tooling unless the task specifically requires a new command.


## Validation

After code or config changes, run the project validators before handing off:

```bash
npm run validate:fast
```

Also run targeted tests for the changed area:

```bash
npm run test
npm run test:e2e
npm run replay:session-trace <trace.json>
```

Use fast scoped checks while iterating, then a full relevant validation pass before committing or reporting completion. If validators fail, fix the cause and rerun them; do not report success with known failures unless the task explicitly asked to skip validation.


## CI/CD Configuration

- GitHub Actions workflows live in `.github/workflows/`.
- The main release workflow builds desktop artifacts, runs validation, and publishes release outputs.
- Node.js bundling is handled by `scripts/bundle-node.mjs`.
- Mobile workflows live in `.github/workflows/mobile-*.yml` and are documented from [CI_PIPELINE](docs/project/CI_PIPELINE.md) and [BUILD_AND_RELEASE_OVERVIEW](docs/project/BUILD_AND_RELEASE_OVERVIEW.md).

Keep workflow changes minimal and validate them locally where practical.


## Code Layout

- [`src/core`](src/core/AGENTS.md) — platform-agnostic business logic and boundary interfaces. No Electron imports.
- [`src/main`](src/main/AGENTS.md) — Electron main process and desktop-only services.
- `src/preload` — preload scripts bridging main and renderer.
- [`src/renderer`](src/renderer/AGENTS.md) — React UI and agent-session orchestration.
- `src/shared` — shared types, IPC contracts, schemas, and channel policies.
- [`cloud-service`](cloud-service/AGENTS.md) — Node HTTP service reusing core handlers.
- `cloud-client` — shared client library for cloud/mobile consumers.
- [`mobile`](mobile/AGENTS.md) — React Native app; keep platform-specific UI here and shared logic elsewhere.
- `scripts` — build, CI, validation, release, and developer tooling.
- [`evals`](evals/AGENTS.md) — LLM eval harnesses, fixtures, and benchmarks.
- `rebel-system` — bundled system skills and user-facing help content.
- `resources` — connector catalog and packaged resources.

Always check for a closer `AGENTS.md` before editing within a subtree.


## UI Components

Use shared components from `@renderer/components/ui`:

```ts
import { Button, Dialog, Input } from '@renderer/components/ui';
```

Rules of thumb:

- Avoid raw `<button>` elements when a shared component exists.
- Avoid one-off CSS and additions to deprecated stylesheets.
- Use design tokens and test both light/dark themes.
- Use `lucide-react` for icons unless the design system already provides a more specific primitive.
- Preserve accessibility: keyboard flow, focus states, labels, reduced-motion behavior, and screen-reader text.


## Debugging & Development

- **Type errors**: run `npm run lint:ts`. The project uses a ratcheted type-error baseline; do not increase it.
- **Runtime issues**: use Electron devtools for renderer behavior and `src/core/logger.ts` for structured logs.
- **Renderer console logs**: renderer warnings/errors are captured in app logs with a `[Renderer]` prefix. See [DEBUGGING](docs/project/DEBUGGING.md).
- **Agent behavior**: prefer adjusting existing agent/session utilities before adding abstraction layers.
- **Conversation diagnosis**: for `rebel://conversation/{id}` issues, start with [DIAGNOSE_ONE_OF_MY_CONVERSATIONS](docs/project/DIAGNOSE_ONE_OF_MY_CONVERSATIONS.md).
- **Submodules**: if `rebel-system` or `super-mcp` are empty or stale, initialize/update submodules before validating.


## Git & Change Management

- Work on a feature branch or worktree for substantial changes.
- Keep commits atomic and scoped to the current task.
- Review `git diff` before committing and check for secrets, private URLs, and unrelated files.
- Do not push unless explicitly instructed.
- Do not use destructive commands (`git reset --hard`, broad restores, deleting untracked files) unless the owner explicitly asks for that exact cleanup.
- Treat untracked files as user-owned work.
- If a merge is in progress, verify staged incoming changes before committing.

Suggested commit shape:

```text
<type>(<scope>): <Summary sentence. Optional second sentence.>

- Optional details
- Validation performed

AI-Workflow: <workflow-or-direct>
AI-Implementer: <agent/model>
AI-Review-Mode: <none|light|medium|heavy>
```

Submodule notes:

- Check submodule status before committing if the change touches `rebel-system`, `super-mcp`, or other submodules.
- Ensure submodule commits are on an attached branch before committing within the submodule.
- Stage submodule pointer changes intentionally; do not capture them accidentally.


## Security & Privacy

- Protect user data, provider credentials, OAuth tokens, telemetry DSNs, and workspace file contents.
- Prefer opt-in integrations and env-configured credentials for OSS-friendly defaults.
- Keep failure modes explicit and observable.
- Run TruffleHog or the configured secret scan before commits that touch config, docs, fixtures, or auth paths.
- Corporate contact inboxes such as `hello@mindstone.com` or `contact@mindstone.com` are public contact strings and may remain where intentionally used.


## Recovery Playbook

If work drifts from the plan or validation exposes a larger issue:

1. Stop broadening the change.
2. Preserve useful artifacts: failing command output, logs, repro steps, and diffs.
3. Identify the smallest safe next step.
4. Report blockers and risks clearly.

Update this file only when repository-wide build, validation, architecture, or contributor workflow guidance changes.

---
> Source: [mindstone/rebel-app](https://github.com/mindstone/rebel-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
