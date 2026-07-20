## imessage-sdk

> This file provides repository context and working rules for AI coding assistants contributing to

# AGENTS.md

This file provides repository context and working rules for AI coding assistants contributing to
`imessage-sdk`.

## Project overview

`imessage-sdk` is a provider-neutral, ESM-only TypeScript conversation layer for iMessage
infrastructure. It normalizes provider messages, conversations, events, capabilities, and errors
while preserving provider-specific functionality on the concrete provider object.

- Repository: https://github.com/jmisilo/imessage-sdk
- License: MIT
- Package manager: pnpm workspaces
- Build tool: tsup
- Test framework: Vitest
- Release tooling: Changesets

The architectural boundary is:

```text
Provider APIs
    -> @imessage-sdk/<provider>
    -> imessage-sdk public provider contract and client
    -> adapters and integrations
```

The core package must not contain provider-specific request logic. Adapters and integrations must
depend on the public `imessage-sdk` interface rather than provider internals.

## Repository structure

| Directory                       | Purpose                                                               |
| ------------------------------- | --------------------------------------------------------------------- |
| `packages/imessage-sdk`         | Provider-neutral core package (`imessage-sdk`)                        |
| `packages/providers/<provider>` | Independently published provider package                              |
| `packages/providers/README.md`  | Cross-provider feature support matrix                                 |
| `packages/chat-adapter`         | Chat SDK integration (`@imessage-sdk/chat-adapter`)                   |
| `packages/cli`                  | Provider-neutral CLI (`imessage-cli`)                                 |
| `examples/basic-blooio`         | Opt-in live example using only published Blooio and core APIs         |
| `test/package-consumer`         | Clean TypeScript consumer used by package smoke tests                 |
| `.changeset`                    | Changesets configuration, prerelease state, and pending release notes |
| `.github/workflows`             | CI and automated release workflows                                    |
| `RELEASING.md`                  | Maintainer release and registry setup guide                           |

The repository does not use Turborepo. Workspace membership is defined only by
`pnpm-workspace.yaml`.

## Package relationships

```text
@imessage-sdk/<provider>     -> imessage-sdk
@imessage-sdk/chat-adapter   -> imessage-sdk
imessage-cli                 -> imessage-sdk + every official provider
```

Workspace packages import package names, never source files from another package:

```ts
import { blooio } from '@imessage-sdk/blooio';
import { createIMessageClient } from 'imessage-sdk';
```

Use `workspace:^` for runtime dependencies on other publishable workspace packages. Do not add
TypeScript path aliases that allow package boundaries to be bypassed.

## Development setup

Requirements:

- Node.js `^20.19.0`, `^22.13.0`, or `>=24`
- pnpm `10.18.3`

Install and build from the workspace root:

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm build
```

Do not replace pnpm with npm, Yarn, or Bun for workspace operations. Keep the shared
`pnpm-lock.yaml` current when dependencies change.

## Development commands

### Root commands

| Command                 | Purpose                                                        |
| ----------------------- | -------------------------------------------------------------- |
| `pnpm build`            | Build every package that defines a build script                |
| `pnpm typecheck`        | Type-check every package that defines a typecheck script       |
| `pnpm test`             | Run unit tests across the workspace; live tests remain skipped |
| `pnpm lint`             | Run ESLint and check Prettier formatting                       |
| `pnpm format`           | Apply Prettier and import sorting                              |
| `pnpm package:check`    | Validate packed packages and a clean TypeScript consumer       |
| `pnpm changeset`        | Create release metadata for changed public packages            |
| `pnpm changeset status` | Inspect pending package version changes                        |

### Package-scoped commands

Use pnpm filters instead of changing unrelated packages:

```bash
pnpm --filter imessage-sdk build
pnpm --filter @imessage-sdk/blooio test
```

Provider packages also expose opt-in live integration tests. These contact real services and may
send messages or mutate provider state. Never run them unless the user explicitly authorizes the
live operation and the required credentials and test targets are already configured.

## Public API and import rules

The public surface of each package is defined by its `package.json` `exports` map.

- Core APIs and types come from `imessage-sdk`.
- Concrete providers come from their own packages, such as `@imessage-sdk/blooio`.
- Do not restore provider subpath exports such as `imessage-sdk/providers/<provider>`.
- Do not expose `src`, `dist` internals, mappers, transport helpers, or test utilities.
- Do not use deep imports across package boundaries.
- Use ESM imports and explicit `.js` extensions for relative imports in TypeScript source.
- Do not add CommonJS builds or `require()` usage without an explicit compatibility decision.

When a public export changes, update package documentation and the clean package-consumer test.

## Core architecture

### One provider per client

Version 0.1 creates one client for one concrete provider connection:

```ts
const client = createIMessageClient({
  provider: blooio(),
});
```

Do not add multi-provider routing, fallback, or provider selection to the core client without an
explicit architecture decision. Applications can create multiple clients when they need multiple
provider connections.

### Provider-aware typing

`defineProvider()` and `createIMessageClient()` must preserve the concrete provider name,
capabilities, and methods through TypeScript generics. Provider-specific APIs are available through
the selected provider:

```ts
client.providers.blooio;
```

Do not widen literal provider names or capability values to `string` or `boolean`. User-defined
providers must remain supported; never replace `IMessageProviderName = string` with a closed union
of built-in providers.

Optional methods in the general provider contract may be required by a concrete provider
interface. Preserve that more specific type so implemented methods do not appear optional to users
of the concrete provider.

### Capabilities

Capabilities describe which normalized client operations are available for the configured
provider and mode. The normalized client keeps a consistent method surface and throws
`UnsupportedCapabilityError` when an operation is unavailable.

- Capability declarations and implementations must agree.
- Keep capabilities static for v0.1.
- A capability indicates runtime availability, not API stability. Normalized webhook verification
  and event parsing are stable in v0.1.
- Provider-specific methods may exist only when intentionally public and documented.
- Keep experimental functionality disabled until it is deliberately included in the normalized
  API and verified in the relevant provider mode.

### Provider encapsulation

All provider behavior belongs inside the concrete provider returned by its factory:

```ts
const provider = blooio(options);
```

Provider packages own authentication, requests, response validation, mapping, webhook parsing,
stream handling, provider-specific errors, and provider extensions. The core client only validates
the contract, dispatches normalized operations, decorates results with connection identity, and
enforces capabilities.

Provider factories should read their standard credentials from documented environment variables
when explicit options are absent. Explicit options take precedence. Never read test-only options in
production provider configuration.

### Messages, attachments, and replies

Keep v0.1 message input focused on text, attachments, and replies.

- A send must have a destination: `conversationId` or `to`.
- A send must have content: non-empty text and/or at least one attachment.
- Attachment kinds are `image`, `video`, and `file`.
- Public URL, `Blob`, and `Uint8Array` sources are represented by the core contract.
- A provider may reject a source it cannot transport; do not silently discard it.
- Replies reference the provider-native target message ID and optional part index.
- Preserve `providerMessageId`, `providerStatus`, and `raw` for diagnostics.

Do not add an attachment-storage abstraction until its API is deliberately designed. Public URLs
are the currently verified cross-provider path.

### Conversation identity

Provider-native conversation IDs are opaque and scoped to a provider connection. Never assume a
phone number alone identifies a conversation.

When a provider message omits its conversation ID, the client may generate a non-routable fallback
with the `imsg-sdk-v1-` prefix. A fallback ID is for stable normalization and diagnostics; it must
not be sent back to a provider as though it were a native conversation ID.

Use the exported `DEFAULT_CONNECTION_ID` instead of duplicating the literal default connection ID
throughout the codebase.

### Events and lifecycle

Providers normalize signed webhooks and persistent streams into `ProviderEvent`. The client
decorates those into `IMessageEvent` with the concrete provider and connection ID.

- Webhook verification must happen before parsing is accepted.
- Preserve the provider payload in `raw`.
- A webhook may produce multiple normalized events, so handlers return arrays.
- Stream implementations must honor `AbortSignal` when supported.
- `close()` is optional on providers and is for transports or resources that require cleanup.
- Client `close()` must remain idempotent.

Do not add initialization requirements for adapters that can connect lazily.

### Errors

Map provider failures to the narrowest public SDK error class and preserve provider metadata,
status, retry information, trace IDs, and raw causes when available.

Use `AmbiguousDeliveryError` when a send may have been accepted despite an uncertain response.
`retryable: true` does not imply that repeating a send is safe. Never add automatic message retries
that can create duplicate delivery without explicit idempotency guarantees.

## Provider implementation standards

Provider response and webhook payloads are untrusted input.

- Validate external data with Zod before mapping it.
- Keep response schemas minimal: parse only fields used by the adapter.
- Allow unknown provider fields unless strict rejection is required by the provider protocol.
- Prefer `safeParse` where malformed optional data can be ignored safely.
- Preserve the original payload in normalized `raw` fields.
- Keep mapping functions deterministic and independently testable.
- Keep transport-specific dependencies in the provider package, not in core.
- Do not put injectable clocks or `fetch` functions in public options solely for tests; mock runtime
  globals or use internal seams instead.

When adding a provider:

1. Create `packages/providers/<provider>` as an independent ESM package.
2. Depend on `imessage-sdk` with `workspace:^`.
3. Implement the public provider contract with `defineProvider()`.
4. Preserve literal name and capability types with `as const` and concrete interfaces.
5. Add unit tests for mapping, errors, webhooks, capabilities, and public typing.
6. Add an opt-in live integration test when real API verification is possible.
7. Add the provider to `packages/providers/README.md` and its support matrix.
8. Add the package to `scripts/check-packages.sh` and `test/package-consumer`.
9. Add the provider to the `imessage-cli` registry; its completeness test must pass.
10. Document install, configuration, verified operations, and known limitations.
11. Add a Changeset for all affected public packages.

## Coding standards

- TypeScript is strict; do not weaken root compiler options to make a change pass.
- Prefer interfaces and functions for public contracts and composition unless a class materially
  improves lifecycle or state encapsulation.
- Keep public inputs immutable with `readonly` where practical.
- Use `unknown` for untrusted or provider-specific raw values; avoid `any`.
- Preserve `exactOptionalPropertyTypes` semantics. Omit an optional property instead of assigning
  `undefined` unless the type explicitly permits it.
- Use exhaustive switches for normalized discriminated unions.
- Source and test filenames use `kebab-case.ts` and `kebab-case.test.ts`.
- Formatting is controlled by Prettier and import sorting in `.prettierrc`.
- Linting is controlled by `eslint.config.ts`.
- Do not manually reformat generated output or commit `dist`, coverage, tarballs, or credentials.

## Testing

Tests should verify behavior and public typing, not private implementation details.

- Framework: Vitest.
- Add regression tests for bug fixes.
- Add capability and unsupported-operation tests when the normalized surface changes.
- Test provider payload parsing with realistic fixtures and malformed input.
- Test webhook signatures, timestamps, and replay tolerance without real credentials.
- Keep live provider tests opt-in and skipped during normal `pnpm test` and CI.
- Update `test/package-consumer` when exports, package relationships, or generic inference change.

Before handing off a code change, run the narrowest relevant checks while iterating, then run from
the workspace root:

```bash
pnpm format
pnpm lint
pnpm build
pnpm typecheck
pnpm test
```

For package metadata or exports changes, also pack the affected package and validate the archive
with Publint, Are the Types Wrong, and a clean TypeScript consumer, matching the package smoke test
in `.github/workflows/ci.yml`.

## Changesets and releases

Every pull request that changes a published package's behavior or public API must include a
committed `.changeset/*.md` file.

```bash
pnpm changeset
```

Use conventional SemVer:

- `patch`: backward-compatible fixes
- `minor`: backward-compatible features
- `major`: breaking changes

Select only public packages actually affected. Tests, internal refactors, documentation-only
changes, and private workspace packages do not need a changeset unless they alter a published
artifact. When Changesets prerelease mode is active, eligible releases receive the configured
prerelease suffix.

Do not manually edit package versions or generated package changelogs during the regular release
flow. The Version Packages pull request performs those updates. Do not publish from an agent session
unless the user explicitly requests the external release action. Follow `RELEASING.md` for registry,
OIDC, npm dist-tag, and GitHub Release procedures.

Build all packages once before packing or publishing. Do not add package-level `prepack` build
scripts: Changesets may publish independent packages concurrently, and declaration builds can race
against workspace package output. The repository publish helper derives the npm dist-tag from
Changesets prerelease state; do not hard-code `latest` or `beta` in package manifests.

## Task completion guidelines

### Bug fixes

A complete bug fix normally includes:

1. A regression test that fails before the fix.
2. The smallest fix at the correct package boundary.
3. Documentation updates when user-visible behavior changes.
4. A Changeset when a published package is affected.
5. Relevant package checks followed by workspace verification.

### New features

A complete public feature normally includes:

1. Core contract changes only when the behavior is provider-neutral.
2. Provider implementation and realistic unit tests.
3. Usage documentation and, when useful, an example.
4. Capability declarations that match verified behavior.
5. Clean-consumer type coverage for new public exports or inference.
6. A Changeset for every affected public package.

### Internal changes

- Preserve public behavior and package boundaries.
- Add tests when behavior or risk changes.
- Do not add a Changeset for changes that cannot affect a published artifact.

Use judgment for trivial documentation and comment changes. Ask before making an API or release
decision that would materially expand the requested scope.

## Do not

- Do not add FaceTime, provider routing, automatic fallback, RCS/SMS-first APIs, provisioning,
  scheduling, rich cards, or other postponed features to v0.1 without explicit direction.
- Do not move provider-specific behavior into `imessage-sdk` core.
- Do not make adapters depend directly on provider request clients.
- Do not close the provider-name type to a list of built-in providers.
- Do not claim a capability based only on an upstream SDK type; verify the configured operating mode.
- Do not expose experimental functionality through normalized v0.1 APIs merely because an
  upstream provider API or dependency contains it.
- Do not silently retry an ambiguous send.
- Do not bypass workspace package exports with source imports or TypeScript aliases.
- Do not commit secrets, `.env` files, generated `dist` output, coverage, or `.tgz` archives.
- Do not run live integration tests, publish npm packages, create GitHub Releases, or mutate npm
  dist-tags without explicit user authorization.

---
> Source: [jmisilo/imessage-sdk](https://github.com/jmisilo/imessage-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
