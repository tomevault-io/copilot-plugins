## outpost

> This file applies to the entire repository. It is the durable project brief for contributors and coding agents. Keep it aligned with the implementation and CI; do not turn it into a conversation log. Follow the user's current scope and instructions when they differ from these defaults.

# Working on Outpost

This file applies to the entire repository. It is the durable project brief for contributors and coding agents. Keep it aligned with the implementation and CI; do not turn it into a conversation log. Follow the user's current scope and instructions when they differ from these defaults.

## Purpose and philosophy

Outpost is a TypeScript library and CLI for running coding agents in sandboxes, managing their Git workspaces, preserving conversations and composing typed workflows.

- Prioritize reliable, directly usable behavior and an excellent developer experience.
- Make code understandable through names, small responsibilities and explicit contracts.
- Use domain-driven design and ports and adapters pragmatically. Introduce abstractions for real responsibilities and variations, not speculative flexibility.
- Keep agent protocols independent of sandbox backends. Claude Code, Codex, Antigravity (`agy`), GitHub Copilot CLI and Kimi Code have adapters; Kimi supports native capture, resume, fork and response repairs; Copilot supports capture, resume and repairs but rejects automated fork. Antigravity supports resume and repairs only in its existing sandbox, without portable capture or automated fork. Additional agents belong behind the existing ports.
- Harness model requests use the stable `ModelProvider` contract and `createOpenAIModelProvider()` / `createAnthropicModelProvider()` in `src/adapters/models/`, independently of sandbox providers and CLI agent adapters. Providers exchange bounded messages with tool calls and replayable reasoning, with optional streaming. Built-in subagents use `defineHarnessSubagent()` with a borrowed sandbox, intersected declarative permissions, separate histories and cumulative ancestor token budgets.
- Preserve existing features and public contracts during refactoring. Architecture changes must not silently change execution behavior.
- Prefer explicit ownership, predictable failure modes and recoverable state over hidden automation.
- Distinguish released behavior, implemented but unreleased additions, and opt-in research prototypes. Keep remaining work and live-validation prerequisites in the roadmap; do not imply publication from a local implementation.

## Start here

Read `README.md`, `package.json`, `SECURITY.md` and `docs/src/content/docs/guide/integration-ports.md`. Inspect the relevant source, tests and workflows before editing. Check Git status and preserve unrelated work.

Use the repository as the source of truth for versions, supported options and commands. Do not rely on test counts, coverage percentages or publication status remembered from a previous chat.

- Runtime: Node.js 24+, TypeScript, ESM. Bun (version pinned by `packageManager`) installs dependencies and runs scripts, with committed `bun.lock` files; tests and scripts still execute on Node.js. `images/agents/` keeps its npm lockfile because the agent image installs CLIs with npm.
- Public package: `@elie-laloum/outpost`.
- Public facade: `src/index.ts` and the provider subpaths declared in `package.json`.
- Canonical repository: <https://gitlab.elielaloum.com/elielaloum/outpost>.
- GitHub mirror and CI: <https://github.com/elie-laloum/outpost>.
- Documentation: <https://elie-laloum.github.io/outpost/>.

## Architecture and responsibilities

| Location               | Responsibility                                                                               |
| ---------------------- | -------------------------------------------------------------------------------------------- |
| `src/domain/`          | Contracts, validation, prompts, responses, usage, task graphs and workflow rules.            |
| `src/application/`     | Use cases, resource ownership, dispatch, lifecycle orchestration and remote synchronization. |
| `src/adapters/agents/` | One folder per agent: request building, event decoding, descriptor. Plus the catalog.        |
| `src/providers/`       | Sandbox allocation, command execution, file transfer and disposal.                           |
| `src/infrastructure/`  | Processes, binary streams, Git, files, native conversation storage and logging.              |
| `src/cli/`             | Argument handling, project scaffolding and image commands.                                   |

`scripts/check-architecture.mjs` enforces these internal dependency directions:

- Domain depends only on domain modules.
- Infrastructure may depend on domain and infrastructure.
- Adapters may depend on domain, infrastructure and adapters.
- Providers may depend on domain, infrastructure, adapters and providers.
- Application may depend on every non-CLI layer.
- CLI composes the other layers.

These are allowed boundaries, not a reason to add unnecessary dependencies. Keep vendor SDKs and protocol details out of the domain. Optional cloud SDKs must remain optional and load through their provider entry points.

Apply SRP throughout the codebase: allocation, request building, event decoding, process supervision, transfer, storage and cleanup have different reasons to change. Split them accordingly. Do not centralize Claude and Codex implementations in a provider file. Compatibility facades such as `providers/agents.ts` re-export; internal services import their owning modules directly.

Compose agents with `createAgent({ harness, model })`; `createCodexHarness()`, `createClaudeHarness()`, `createAntigravityHarness()`, `createCopilotHarness()` and `createKimiHarness()` are CLI presets. `createHarness({ modelProvider, instructions, tools, limits, toolExecution })` configures the built-in Outpost loop; tools come from `defineHarnessTool()`/`defineHarnessToolset()` and never bypass the borrowed sandbox. Models are names or `{ name, reasoning, maxOutputTokens }` objects; the executing harness or model provider validates them when the agent is composed and rejects unsupported settings. Each built-in CLI agent owns `src/adapters/agents/<agent>/`, including a descriptor registered once in `catalog.ts` and named in `BuiltInAgentName`; CLI choices, init authentication, doctor, bootstrap, versions and the image recipe derive from the catalog, so do not add per-agent lists elsewhere. Native conversation formats belong to the agent folder and are built from `createTranscriptConversations()` or `createSessionBundleConversations()`; infrastructure must not branch on format names, and persisted format names must stay stable. Use `AgentAdapter` only for CLI protocol behavior, `SandboxProvider`/`SandboxLease` and explicit `create*SandboxProvider()` factories for execution environments and `ConversationStore` for transcript persistence. Prefer composition and injected capabilities to inheritance or branching on provider names throughout the application. Extend the relevant adapter or strategy when introducing a variant.

## Workflow projects and repositories

`outpost init` creates a standalone workflow project directly in `--directory`, including `run.ts` (`run.mts` for explicit CommonJS manifests), a brief, environment declarations and provider files. Preserve existing package manifests and ignore rules. `--repository` selects a local Git checkout independently of the workflow directory; generated scripts resolve relative paths from their own directory and pass `repository` explicitly to dispatch.

The CLI uses Commander for command-specific parsing/help and Clack for interactive setup. Preserve headless execution, JSON output and cancellation. `init` builds Docker/Podman images by default; `--no-build` opts out. Authentication is selected explicitly on each CLI harness: `account` (the CLI's own host session file, `account.file`, or a token `key`/`variable`) or `usage` (API keys). There is no automatic mode. Per-agent strategies plan credentials without disk access; the application reads host files and installs them in the private sandbox home, and the local provider receives variables only. Never read a system keychain, write credential files on the host, or forward undeclared secrets. Custom Codex model providers require Responses API compatibility; do not imply Chat Completions support.

Each sandbox owns one repository. Compose multiple repositories with `defineIsolatedTask` and dependency edges; no shared Git transaction or automatic push spans them. Runtime worktrees, local ownership locks, default logs and custom harness transcripts belong under the target repository's `.outpost`. Artifacts, checkpoints, journals, reservations and resource activity persist through Transport only; default runtime objects use createLocalTransport under .outpost/storage. There are no file-store compatibility factories or logging.file option. Journals expose logReference. Checkpoint ownership and abandoned reservations require explicit recovery locally and remotely. Native conversations, Git, locks and execution staging still require filesystems; explicit transports can archive conversations and recovery data. Keep bilingual repository guidance and standalone/multi-repository regression tests aligned with these contracts.

## Code conventions

- Keep source identifiers, errors and project guidance in English.
- Put interfaces, type aliases and object type declarations in adjacent `*.types.ts` modules. Use type-only imports; these files must have no runtime initialization.
- Put configuration defaults, supported values, shared limits and reusable fixed recipes in adjacent `*.constants.ts` modules. Keep local variables, computed values and closures with their operation; do not create a global constants dumping ground.
- Use guard clauses for validation and early exits. Use strategies or handler registries for behavioral alternatives. Do not introduce `if / else if / else` chains or disguise them as nested ternaries.
- Keep simple conditions simple. Avoid abstraction layers that make a single operation harder to read.
- Respect strict TypeScript settings. Narrow unknown external data at boundaries; do not silence errors with unchecked casts or `any`.
- Follow the existing relative `.ts` import convention and public export layout.
- Name public functions by when their work happens: `create*` builds a runtime object (agent, harness, provider, transport, store, queue, observer, reporter); `define*` declares what an engine runs later (workflows, tasks, response and artifact contracts, harness tools, hooks, skills); a verb acts immediately (`dispatch`, `speculate`, `readJournal`). Give each constructor a name distinct from every type.
- When renaming a public function, keep the previous name as a `@deprecated` alias (in `src/deprecated.ts`, or beside a subpath export) until the next major version. Reference generation skips `@deprecated` exports and guide snippets reject them; redirect the retired reference page to its replacement.
- Comments are exceptional: prefer one line, never more than two lines per comment. Explain a non-obvious constraint or reason, not what the code says. Do not split a long explanation into adjacent comments to bypass this rule.
- Put longer explanations in Markdown documentation. The comment limit does not limit documentation prose.
- Use the repository's Prettier configuration. Format changed files without unrelated churn.
- Avoid new dependencies unless they materially simplify a requirement. Preserve the lightweight core and optional provider integrations.

## Behavioral invariants

Treat these as review and regression-test obligations when changing the affected code:

- Workspaces and sandboxes have separate lifetimes. Preserve cold execution, warm reuse, exclusive operation ownership and idempotent disposal.
- A command completes when its process completes. Preserve exit status even when stdout/stderr close early. Cancellation and deadlines must terminate the intended process group and descendants without destroying an otherwise reusable sandbox.
- Non-TTY container commands use `setsid --wait`. Real interactive terminals use the session supplied by the container runtime. Do not apply the non-TTY wrapper blindly to TTY execution.
- Container transfers must see the live mounted filesystem, including tmpfs. Do not replace the streamed archive implementation with `docker cp`/`podman cp` without proving equivalent behavior for these mounts.
- Transfer binary data without text decoding or output-retention truncation. Preserve supported file permissions, symlinks and directory-content semantics. Reject unsafe destination traversal and clean temporary staging on success, failure and cancellation.
- Keep the agent home coherent. The default private home remains ephemeral; do not solve authentication by persisting only a fragment of it. Generated images must create the home with correct ownership and permissions.
- Preserve conversation capture, restore, continuation, fork and transcript relocation. Agent authentication and conversation storage are separate concerns.
- Preserve hook ordering, structured-response validation, retries, usage aggregation and observer isolation. Observer failures must not change execution outcomes. Propagate observation scope explicitly through `ObservationHub`; keep historical agent/workflow callbacks compatible. Sink delivery is bounded and may report loss; it is not a durable state registry. Keep usage accounting synchronous and independent of user sink delivery.
- Interactive tasks persist human questions and accepted answers between completed agent turns. Require portable conversation capture/resume, retain the named worktree, release each sandbox before publishing a question, and require explicit replay after an interrupted turn. Input waits are distinct from approval gates; never replay completed dialogue turns during normal answer submission.
- Task caches restore only a lossless JSON value keyed by workflow, task, version and a caller key; never replay side effects or treat unauthenticated entries as trusted identity. Store failures and invalid entries degrade to a miss without changing task outcomes; hits consume no attempts or usage.
- Quota classification belongs to adapters: only terminal usage/rate-limit signals on a failed turn become `OutpostError` code `quota`; transient retry notices must not. `onQuota` pauses are opt-in settled checkpoint states that do not consume task retries, stay paused when a wait is cancelled and never wait beyond `maxWaitMs`. Resumed attempts continue only captured conversations; queued resumes publish a new job while keeping the handler's idempotency key; durable speculation reruns only quota-stopped candidates.
- Steering never cancels a dispatch. Instructions reach the running turn only through the built-in harness loop (the active subagent while one runs, or the run or main loop named by `send(text, { subagent })`) or an adapter's `liveInput` session on a lease with `liveInput`; otherwise Outpost stops the process once its conversation is known and resumes it, keeping the sandbox. Close live stdin only after a final event once every written message was consumed, and answer protocol requests so they cannot block the agent. Vercel and Daytona deliver live input through an append-only sandbox file read by a wrapper that must run in the killed process group. Undelivered instructions reject with code `steering`; a controller serves one dispatch at a time, each pass emits one summary with aggregated turn usage, and replays split recorded passes at `resumed` steer events.
- Triggers never run workflows inside the request or timer that fires them: they publish deterministic queue jobs (`schedule:<name>:<slot>`, `trigger:<path>:<delivery>`) so replicas, restarts and redeliveries converge, and `defineWorkflowJob()` runs each under its `runId` checkpoint with an input-derived version. Sources verify signatures before parsing and fail closed; a verified sender identity is not an Outpost gate actor. Cron slots are wall-clock times: skipped DST times do not fire, repeated ones fire once, and only the latest slot within `maxLateMs` is caught up.
- Fallback agents are explicit: `createFallbackAgent()` requires an `on` list and moves to the next candidate only on a covered `quota` or `unavailable` fault, never on cancellation, deadlines or other failures. Candidates share the sandbox and workspace without reset, restart from the original brief and keep captured conversations. Outage classification belongs to adapters and model providers, ignores retry notices and keeps the original fault code.
- Durable workflows persist lossless JSON outputs, explicit replay authorization and cumulative usage. Gates use trusted actor metadata by default; opt-in signed gates bind decisions to verified approver keys and preserve their authentication requirement in checkpoint identity. HTTP queue credentials can rotate per request. Stable task/worker idempotency keys require persistent deduplication at the effect service; queues fence stale leases but do not guarantee exactly-once effects. Artifact digests and lineage provide integrity, not authentication.
- Resource activity is an observation, and storage reservations coordinate cooperating writers rather than imposing physical quotas. Transport-backed activity must not infer remote liveness from a PID. Transport checkpoint ownership and abandoned reservations require explicit recovery; conditional mutations must fence stale writers. Opt-in research providers and policies must reject unsupported capabilities explicitly.
- Protect concurrent host edits during remote synchronization. Validate and back up before applying incoming changes. Preserve recovery artifacts whenever cleanup would discard recoverable work.
- Keep branch integration explicit and correctly ordered. Never discard dirty or detached worktrees as routine cleanup.
- Durable speculation requires a provider recovery capability and resource registration before allocation. Fence checkpoint writes by revision, require explicit recovery after stopping an abandoned coordinator, and authorize interrupted-attempt replay separately. Preserve cumulative budgets and old worktrees; a cleanup timeout leaves resources pending. Merge preflight records exact commits but does not authorize integration or prevent later host edits.
- MCP server configuration references secrets by variable name only, never by value, in arguments or files. CLI home configuration is merged by section, never overwritten wholesale, including the user's home with the local provider. Built-in harness MCP servers and their HTTP bridge run inside the borrowed sandbox and require a lease with `liveInput`. Host MCP OAuth logins are copied only for declared `oauth: "login"` servers, with the credential file safeguards and never on the local provider; OAuth client credentials are requested by the in-sandbox bridge. MCP options a CLI cannot apply are refused when the agent is composed.
- Do not silently fall back from an isolated provider to host execution. `createLocalSandboxProvider()` is explicitly unisolated; mounted Git metadata is not an adversarial security boundary.

## Tests and coverage

Use `node:test` and `node:assert/strict`, following the existing suite. Test observable contracts and failures rather than mirroring implementation details.

- `test/unit/`: domain rules, protocol adapters, boundaries and isolated infrastructure behavior.
- `test/functional/`: lifecycle, Git, synchronization, recovery, CLI and workflow behavior using temporary resources.
- `test/redis.test.ts`: real standalone Redis 7/8 queue behavior, including concurrent claims, stale leases, cancellation and interrupted publication/finalization. Run with `bun run test:redis`; `OUTPOST_REDIS_PORT` selects a dedicated local test server.
- `test/container.test.ts`: real Docker/Podman behavior. Mocks alone cannot validate process sessions, tmpfs, mounts, ownership or archive transfer.
- `test/fixtures/container-terminal.ts`: real PTY input, exit status, cancellation and warm reuse.
- `scripts/package-smoke.mjs`: the packed package as a consumer sees it, including exports and declarations.

For a bug fix, add a regression that fails for the actual defect. For execution changes, checking stdout alone is insufficient: test nonzero exit status, completion after output closes, cancellation and reuse as relevant. For transfer changes, verify files through a process inside the sandbox, not just an upload/download round trip.

`bun run coverage` enforces **at least 80% lines, branches and functions**. Preserve or improve meaningful coverage. Do not lower thresholds, add exclusions or write trivial tests to make a metric pass. Existing exclusions cover erased type modules and the CLI process entry wrapper; command handlers remain covered.

Keep routine tests deterministic and independent of real account credentials or paid model calls. Clearly distinguish mock tests, real container tests and live provider/model tests when reporting results. Never report an unavailable or skipped check as passed.

### Validation commands

Install root dependencies with `bun install --frozen-lockfile`; install documentation dependencies with `bun install --cwd docs --frozen-lockfile` when needed.

| Change                                                                | Checks                                                                                                         |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Source behavior or architecture                                       | `bun run check` (architecture, typecheck, unit/functional tests, build), then `bun run coverage`.              |
| Public API, exports, packaging or dependencies                        | Also `bun run test:package`.                                                                                   |
| Container commands, transfers, mounts, lifecycle or image scaffolding | Also real Docker and Podman tests and the PTY fixture using the setup in `.github/workflows/ci.yml`.           |
| BullMQ queue state, distribution or connection ownership              | Also `bun run test:redis` against standalone Redis using the setup in `.github/workflows/ci.yml`.              |
| API documentation or changelog sources                                | `bun run docs:sync`, then inspect generated changes.                                                           |
| Documentation content or site configuration                           | `bun run build`, `bun run docs:check`, `bun run docs:build`, `bun run docs:test`, `bun run docs:test:browser`. |
| Any changed tracked content                                           | Prettier check on changed files; full `bun run format:check` before release.                                   |

CI checks Windows, macOS and Linux, real Docker/Podman execution, package consumption, coverage, formatting, documentation and dependency audits. Match relevant CI checks locally where possible. For a guidance-only Markdown edit, formatting and factual/link review are sufficient; do not rerun runtime suites without a reason.

## Documentation

- Use Astro Starlight in `docs/`, with `.md` content and English as the default language.
- English content lives in `docs/src/content/docs/`; French equivalents live under `fr/` with matching relative paths. Update both languages for user-facing changes.
- The site has two navigation spaces in one shared frame (header, per-space sidebar, breadcrumbs): Guide and Reference, labelled “API” in both languages while its URLs stay under `reference/`. Guide owns practical learning, detailed behavior, CLI and configuration pages under `guide/`; Reference contains symbol pages, a bilingual Overview (Vue d’ensemble) introducing each family, and a generated home that maps sections and families without listing every symbol. The bilingual changelog keeps its `project/changelog/` URLs and is reached only from the header’s version button, which stays visible on mobile; exclude it from sidebars, pagination, homepage links and search. The roadmap lives only in the repository’s root `roadmap.md`, without a documentation page. Each overview explains the concept, its boundaries and how to choose its entry points, with a link to its practical Guide; it is reached from the reference home, not the sidebar. Within each reference family, list functions first, other values/types next and interfaces last. Symbols use distinct SVG icons for functions, interfaces, type aliases, classes and constants. Callable values use the function icon; callable type aliases keep the type icon. Firecracker and its options belong to Providers. Preserve existing symbol URLs and redirect moved guide routes; give case-colliding symbols distinct pages.
- The documentation home uses the Guide’s Markdown bays, code tabs, illustrated cards and canvas in `index.md` and `fr/index.md`. Keep its learning sequence focused, link setup to the Guide and typecheck its snippets with the Guide examples.
- Guide pages use snippets of at most 20 lines and direct explanations, following the editorial structure of Better Auth. The recommended setup builds the agent image in a dedicated directory, then imports Outpost in user-written TypeScript scripts; keep generated `init` workflow projects optional. Setup is explained once and linked where needed; do not restore collapsible workshops, a parallel detailed-behavior tree or a root `examples/` directory. TypeScript examples use `.ts` filenames and imports; installation explains the ESM package setting once. Longer examples are split by responsibility into named files shown in tabs, with explicit imports and clear execution instructions. Diagrams use the pannable, zoomable canvas component; do not restore flow blocks. Markdown snippets are typechecked against the package; selected offline snippets are executed. `docs/scripts/rehype-bays.mjs` lays every Guide, Reference and changelog page out as bays on the 12-track frame: code, contracts or entries on the right of their prose, code-free sections full width. Every Guide snippet or tab group needs a relevant explanation beside it; a title or API link alone is insufficient. Keep retired URLs in `docs/audit/guide-redirects.json`.
- Reference signatures and property tables are generated; bilingual explanations live in `docs/reference-content/`. Normalize interfaces to Import, Parameters and properties, Signature, Related contracts, omitting empty sections, generic introductions and Purpose and behavior. Describe each function’s own behavior, ownership, result and relevant differences from neighboring functions. Every parameter/property needs an accurate description in its declaring contract’s context in both languages, including inherited and union fields; never substitute a generic referral or family-level boilerplate. Audit every reference and make generation reject missing descriptions. The migration inventory in `docs/audit/` preserves traceability and legacy routes.
- The Reference sidebar is flat: one alphabetical list of every public symbol with its kind icon, without sections, families or overviews. The reference home maps families under fixed section titles; section and family labels stay in English in both locales, while page content and Overview labels are localized. The section order and short family labels live in `docs/scripts/reference-navigation.mjs`.
- Organize by user task. Prefer focused, navigable pages to large catch-all documents. Maintain sidebar order, cross-links and useful prerequisites.
- Guide pages show working examples and observable behavior; property definitions, option catalogs and return-field dictionaries belong in API reference pages, linked to their declaring contracts. CLI flags remain in the CLI command documentation. Notes and warnings span the content pane, stopping before outer gutters and the table-of-contents column, with distinct colors in both themes; their text stays aligned with the article.
- Examples must match the public API and be usable in their stated environment. Explain expected results, resource ownership, authentication and failure behavior where relevant.
- Organize the guide in the reading order defined by `docs/scripts/navigation.mjs`: get started (tutorials and how Outpost works), use cases, agent tasks, agents, the Outpost harness, sandboxes, workflows, durable runs, automation, storage and observability, operations, extensions. Follow the page types, templates and exclusions in `docs/audit/guide-style.md`. Distinguish instructions to an agent from enforced workflow gates.
- Keep account/subscription login and API-key authentication clearly separated, including billing implications and host-versus-sandbox credential locations. Check current official vendor documentation when modifying authentication guidance.
- Experimental reference symbols are marked in `docs/scripts/api-groups.mjs`; their bilingual warning explanations live in `docs/reference-content/experimental.json`. Regeneration must preserve a warning before the API content for every marked symbol.
- The API reference and both documentation changelogs are synchronized by `docs/scripts/sync-reference.mjs`. Update their sources and regenerate; do not patch generated output as the source of truth. Changelog sources are root `CHANGELOG.md` and `docs/translations/changelog.fr.md`.
- Keep `CHANGELOG.md` at the root and in the documentation. Keep the roadmap in the root `roadmap.md`. Do not reintroduce documentation roadmap pages, a root French README, `CONTRIBUTING.md` or migration guides without a new requirement.
- When API is updated and docs is updated as well always update docs/references with changes.
- Every guide card includes a decorative SVG icon, including informational cards and diagram branches. Page navigation actions divide the available width equally, without empty cells for missing links.
- Documentation checks validate language parity, links, generated references, rendered output, search, card icons and examples. Documentation builds and dev starts force content regeneration so Markdown renderer changes cannot reuse stale HTML. Browser tests cover the home in both development and built-preview modes. Do not treat a successful Astro build alone as complete validation.

## Git, CI and releases

- GitLab is the source repository; GitHub is its mirror and runs GitHub Actions. Avoid independent changes on GitHub that diverge from GitLab.
- Main and pull requests are validated. There is one public documentation site, deployed after the latest eligible stable release; main does not deploy a separate preview site.
- A release tag is `v<package version>`. Keep `package.json`, the lockfile and changelogs consistent. Use the release workflow, including its verification gates and latest-release guard.
- For every release, update the version history in `CHANGELOG.md` and `docs/translations/changelog.fr.md`, and review and update `roadmap.md` at the repository root. Do this before creating the release tag.
- Reconcile roadmap entries with the actual release: move shipped capabilities into the available section, remove obsolete plans, record removed capabilities where relevant, and revise stale version milestones. Keep unimplemented work explicitly planned; do not imply that a major version delivers every previously associated roadmap item.
- Regenerate the documentation changelogs with `bun run docs:sync`, inspect the generated changes, and run the required documentation checks. Include the changelog and roadmap updates in the release commit and verify the changelog’s English/French consistency before tagging.
- npm publication uses provenance and public access. The workflow packs with `bun pm pack`, sets `repository.url` to the GitHub mirror immediately before packing, and uploads the archive with `npm publish` because Bun cannot generate provenance; the source manifest retains the canonical GitLab URL.
- Do not manually bypass failed release checks, move published tags or republish an existing package version. Verify remote workflow, package and documentation status before claiming publication succeeded.
- A local change does not itself authorize a release. Follow the requested delivery scope; do not bump versions or create tags for every edit.

## Working discipline

Reproduce and explain failures before choosing a fix. Consider adjacent contracts and side effects, then make the smallest coherent change that preserves the architecture. Complete the relevant tests and documentation together.

Never commit credentials, private transcripts, runtime artifacts or sensitive logs. Use environment variables and ignored configuration; keep secrets out of command output and remote URLs. Follow `SECURITY.md` for execution and data boundaries.

Preserve unrelated user changes. Avoid destructive resets, broad cleanup and unrelated refactors. Inspect the final diff for accidental generated files, secrets and scope creep.

Report what changed, what was verified, and concrete limitations or remaining work. Do not promise zero side effects or claim feature completeness based only on unit tests. Keep this file current when a lasting project convention changes.

---
> Source: [elie-laloum/outpost](https://github.com/elie-laloum/outpost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
