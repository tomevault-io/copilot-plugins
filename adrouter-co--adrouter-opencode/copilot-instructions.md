## adrouter-opencode

> This independent repository owns `@adrouter/opencode` and uses Bun. Its GitHub repository is

# AdRouter OpenCode repository instructions

## Current local candidate and repository boundary

This independent repository owns `@adrouter/opencode` and uses Bun. Its GitHub repository is
`adrouter-co/adrouter-opencode`; never combine its lockfile, Git history, or release state with CLI or
Desktop.

Public beta.8 remains on `beta`/`latest`. The substantial user-owned, unreleased beta.5
installation-auth work remains parked in `stash@{0}` as `clean-slate-2026-08-02`; do not apply,
drop, rewrite, publish, or treat that snapshot as accepted.

Beta.6 is an immutable rejected version: its plugin target was correct, but its unversioned
provider registration made OpenCode execute public beta.4 and call machine auth. Never promote,
rebuild, or retag beta.6. Accepted beta.8 binds the provider to its own exact package version; the
active beta.9 source changes only bounded footer presentation/coverage and release metadata.
Release mutations require explicit user authorization.

## Source map and toolchain

- `src/server.ts` — OpenCode provider/model registration.
- `src/provider.ts`, `src/contracts.ts`, and `src/transport/` — AI SDK provider, prompt/tool mapping,
  configuration, JSON/NDJSON parsing, and transport protections.
- `src/tui.tsx` and `src/presentation.ts` — display-only bottom panel and cumulative savings.
- `test/`, `scripts/`, and `.github/workflows/` — provider/auth/transport/TUI/release coverage and
  candidate/promotion automation.
- `release-manifest.json` — beta.9 candidate/final-channel intent, not evidence of publication.
- `dist/`, coverage, tarballs, isolated installs, and acceptance output are generated.

Use Bun 1.3.14 and `bun.lock`; do not add npm, pnpm, or Yarn lockfiles. Preserve OpenCode
`>=1.18.4 <2`, strict TypeScript, AI SDK provider v3, SolidJS/OpenTUI, the `adrouter` IDs, and root,
`./server`, and `./tui` exports.

## Public versus local state

Public npm state was rechecked before beta.9 work on 2026-08-10: `beta`/`latest` resolve to beta.8
and no `candidate` tag exists. Re-query before every release claim; beta.9 candidate publication
must not move accepted public channels.

The active beta.9 source registers the six currently qualified coding model IDs and their exact
524,288- or 1,048,576-token context windows while enforcing a conservative 4,096-token integration
output cap. Its registered provider package must include the exact beta.9 version. The integration
path remains text/tool-only even when the underlying model accepts images.

## Remaining release blockers

- Staging currently exposes the integration endpoint and eight-model catalog, but local source or a
  package manifest alone never proves the deployed contract; re-run the authenticated canaries.
- Beta.9 must pass clean-tree checks, protected CI, staging canaries, and registry-backed OpenCode
  execution before it may be published as `candidate`.
- The beta.9 immutable tag, staged assets, registry candidate, cross-host acceptance, and draft
  GitHub release do not exist until their protected release steps succeed.
- The complete clean-tree `release:check` remains a release gate. Local unit, type, lint, build, and
  release-policy checks do not authorize publication by themselves.

## Product and security invariants

- Sponsor/settlement information is provider display metadata only. Never put it in prompts,
  assistant text, tool definitions/results, commands, edits, or compacted context.
- Keep the Tier A/B/C bottom panel to at most three width-bounded rows, prioritize current subsidy
  and deduplicated cumulative savings before the URL, and retain stale-state clearing plus compact
  off/degraded/NONE outcomes.
- Hosted origins must use live execution. Custom remote URLs require HTTPS and HTTP is loopback-only.
  Reject credentialed URLs, authenticated redirects, protected-header overrides, oversized bodies
  or lines, idle streams, malformed events, and divergent final snapshots.
- Preserve native conversation/tool mapping while rejecting unsupported attachments, non-function
  tools, and provider-executed tool approvals. Never log/persist credentials or private response
  bodies.

## Verification and releases

During local source work, use focused checks first:

```sh
bun run format
bun run lint
bun run typecheck
bun test
bun run build
bun run release:policy
```

Do not run hosted canaries, acceptance upload, release soak, tag/publish commands, dist-tag
operations, or repository-visibility changes without explicit authorization. Run the full
`bun run release:check` only from a clean release-input tree when release readiness is actually in
scope. Only a clean, committed, exact candidate with Router conformance, package inspection,
cross-host acceptance, and separate release authorization may be tagged or published. Versions
and tags are immutable; fix forward. Never change protected environments or remote secrets
without explicit authorization.

---
> Source: [adrouter-co/adrouter-opencode](https://github.com/adrouter-co/adrouter-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
