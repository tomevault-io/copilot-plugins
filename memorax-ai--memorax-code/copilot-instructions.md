## memorax-code

> This file defines the durable working rules for coding agents in this

# MemoraX Code Agent Guide

This file defines the durable working rules for coding agents in this
repository. Current user instructions, live source, and executable tests take
precedence over generated memory or historical context.

## 1. Start Safely

- Run `git status --short --branch` before editing. Confirm the intended
  worktree and preserve unrelated tracked, untracked, and staged changes.
- Read the nearest implementation and tests before changing behavior. Prefer a
  small, layered change with an explicit verification target.
- Do not mix unrelated formatting, renaming, dependency updates, or lockfile
  rewrites into the task. Do not run broad auto-fix commands unless required.
- Use English for source identifiers, comments, docstrings, and canonical
  public documentation. Keep `README.md` and `README.zh.md` synchronized for
  user-facing changes.
- Treat `.repo_memory/` as local retrieval guidance, not current-code
  authority. It is Git-ignored and must not leak into public artifacts.
- Use isolated `MEMORAX_CODE_HOME`, `CODEX_HOME`, and `CLAUDE_CONFIG_DIR`
  locations for lifecycle, install, migration, or destructive tests.

## 2. Module Ownership

- `packages/ts/memorax-code-backend` owns Backend lifecycle, typed Hook HTTP,
  the memory service, MemoraX transport, local trace, Memory Viewer, and
  writeback reconciliation.
- `packages/ts/memorax-code-adapter-common` owns durable runtime records,
  cross-process configuration primitives, and shared Hook and Repo Memory
  helpers. It does not interpret client transcripts or plugin policy.
- `packages/ts/memorax-code-codex-adapter` owns the Codex plugin, Hooks,
  native session/workspace observation, diagnostics, and bundled skill.
- `packages/ts/memorax-code-claude-adapter` owns the Claude Code plugin, Hooks,
  native observation, and diagnostics. The shared skill is materialized from
  the canonical bundled source during packaging.
- `packages/npm/memorax-code` owns the public CLI and npm installation, update,
  uninstall, postinstall, staging, and package layout.
- `scripts` and `.github` own repeatable repository checks and CI, not runtime
  behavior.

The Backend is not a model runtime or provider proxy. Codex and Claude Code
retain ownership of provider credentials, models, native tools, provider
traffic, and transcript creation.

## 3. Hook, Session, and Scope Invariants

- Hook commands are versioned and client-qualified. Required session, turn, or
  prompt correlation must be validated at the HTTP boundary; incomplete,
  conflicting, unknown, or client-inapplicable identities fail closed.
- Codex rollout JSONL and Claude Code transcript JSONL are the only content
  authorities for their respective automatic writeback. Hook text, local
  trace, latest-turn guesses, or the other client's format are not fallbacks.
- Session, turn metadata, trace, and operational identity always include the
  client. Equal native IDs from different clients must remain isolated.
- A live session is pinned to its resolved workspace and repository scope.
  Linked worktrees of one repository may share repository scope; unrelated
  repositories and genuine non-Git workspaces keep separate local identity.
  Missing, unreadable, malformed, or conflicting scope authority must not
  silently fall back or rebind.
- Repository resolution is read-only and must not execute Git merely to derive
  memory identity. Preserve path canonicalization, Git marker validation,
  symlink/junction containment, and fail-closed behavior.
- Consume turn metadata only after the matching downstream operation is
  accepted. Rejection, missing content, interruption, or concurrent
  replacement must retain or discard metadata with an explicit reason.
- Backend connection, token, PID, Hook generation, and lifecycle authority use
  versioned private records. Cross-process read/modify/write requires bounded
  locking or equivalent version validation; in-memory serialization is not
  sufficient.
- Keep the default Backend on loopback. External binding requires explicit
  opt-in and authentication.

## 4. Data and User-Facing Boundaries

- Never log, commit, or publish provider or MemoraX credentials, Backend
  tokens, Authorization headers, private transcripts, raw rollout data,
  retained trace files, personal memory, or user absolute paths.
- MemoraX receives only documented query/add/writeback payloads. Retained
  trace artifacts, trace-only provenance, and local transcript paths stay
  local.
- Memory Viewer is a content-free local projection, not memory, transcript,
  session, or lifecycle authority.
- `memorax-code` is the shared user-facing skill. Changes must work in both
  Codex and Claude Code packaging and must keep triggers, metadata, references,
  and resource paths valid.
- Packaged skills must address product users. Do not include maintainer
  runbooks, private paths, unpublished plans, secrets, internal fixtures, or
  local diagnostic artifacts.
- Public commands, requirements, data handling, or client support changes
  update both README files. Configuration changes update
  `docs/configuration.md`; user-facing diagnosis updates
  `docs/troubleshooting.md`; vulnerability and trust-boundary changes update
  `SECURITY.md`.

## 5. Verification

Run the smallest relevant check first, then expand when a change crosses
boundaries:

- Backend: `npm run typecheck --prefix packages/ts/memorax-code-backend` and
  `npm test --prefix packages/ts/memorax-code-backend`.
- Codex: `npm test --prefix packages/ts/memorax-code-codex-adapter`.
- Claude Code: `npm test --prefix packages/ts/memorax-code-claude-adapter`.
- Shared Hook or adapter behavior: run both adapter suites and affected Backend
  tests.
- Documentation: `make docs-check`.
- Install, update, uninstall, CLI, or artifacts: `make npm-package-check`.
- Broad cross-layer changes: `make test`.

Real-client and MemoraX-backed checks are explicit opt-in tests. Keep their
credentials and artifacts outside Git and redact results before sharing.

## 6. Git and Handoff

- Name branches for the change with `feat/`, `fix/`, `refactor/`, `docs/`,
  `test/`, or `chore/` plus lowercase kebab-case. Do not use agent-specific
  prefixes.
- Use commit titles in the form `type:(module) content`. Separate mechanical
  formatting from behavior changes.
- Do not amend, rebase, force-push, or rewrite another contributor's history
  without explicit authorization.
- Do not commit `dist/`, build output, package staging, dependency graphs,
  metrics, trace/report bundles, or temporary test artifacts.
- In the handoff, report changed behavior, verification performed, checks not
  run, remaining risk, and worktree state. Push only when explicitly asked.

---
> Source: [memorax-ai/memorax-code](https://github.com/memorax-ai/memorax-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
