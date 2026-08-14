## ga4-gtm-config-mcp

> Template version: 2026-07-23

# AGENTS.md

Template version: 2026-07-23

Drop-in operating instructions for coding agents. Read this file before every task.

**Working code only. Finish the job. Plausibility is not correctness.**

This file follows the [AGENTS.md](https://agents.md) open standard (Linux Foundation / Agentic AI Foundation). Claude Code, Codex, Cursor, Windsurf, Copilot, Aider, Devin, Amp read it natively. For tools that look elsewhere, symlink at the project root:

```bash
ln -s AGENTS.md CLAUDE.md
ln -s AGENTS.md GEMINI.md
```

For any directory-specific `AGENTS.md`, create the same colocated `CLAUDE.md` and `GEMINI.md` symlinks from that subfolder.

When the agent runtime supports Superpowers, install or enable it for the project on first start and invoke `using-superpowers` before ordinary task handling. If Superpowers is unavailable, say so explicitly and continue with this file as the fallback.

---

## 0. Non-negotiables

1. **No filler.** Start with the answer or action.
2. **Disagree when needed.** If the premise is wrong, say so before doing the work.
3. **Never fabricate.** Read the file, run the command, or say you do not know.
4. **Stop when materially ambiguous.** Ask instead of silently choosing.
5. **Touch only what the task requires.** No drive-by refactors or formatting.
6. **Protect contracts.** Existing architecture, interfaces, migrations, and explicit decisions are requirements unless the user changes them.
7. **Keep local data local.** Never commit real secrets, emails, IDs, hostnames, usernames, or absolute machine paths. Use placeholders.
8. **No agent or tool branding in project content.** Do not mention assistants, models, agent workflows, or generation tools in source, docs, comments, commits, or PR text unless the task is specifically about them.

---

## 1. Before editing

- State a one- or two-sentence plan and success criteria.
- For non-trivial work, use a short numbered plan with a verification step for each item.
- Read the files you will change and the code or docs that depend on them.
- Read relevant plans, architecture notes, and postmortems before changing related behavior.
- Match the repository's existing patterns and public contracts.
- Surface assumptions that materially affect the result.
- When two approaches are viable, present both and recommend one before implementing.

---

## 2. Implementation scope

- Build the smallest complete solution to the stated problem.
- Do not add speculative features, abstractions, hooks, or configurability.
- Reuse existing components, utilities, tokens, templates, and conventions.
- Put shared behavior at the highest applicable shared layer, not in one-off local variants.
- Do not refactor adjacent working code unless the task requires it.
- Clean up only the unused code, imports, files, or documentation made obsolete by your own change.
- Preserve edge cases and architectural constraints even when a shorter implementation is possible.

---

## 3. Files and instruction hierarchy

- Follow the nearest `AGENTS.md`; deeper files may add constraints but must not weaken parent rules.
- Keep colocated `CLAUDE.md` and `GEMINI.md` symlinked to the local `AGENTS.md`.
- Put new files in the repository's established folders. If no layout exists, use `src/`, `tests/`, `docs/`, `scripts/`, and `assets/` as appropriate.
- Do not create empty directories or placeholder files.
- Follow `docs/AGENTS.md` for agent work artifacts and `docs/postmortem/AGENTS.md` or legacy `postmortem/AGENTS.md` for postmortems when present.

---

## 4. Verification

Define success before implementation, then verify it:

1. Turn vague requests into observable outcomes.
2. Add or identify a test, script, benchmark, or visual check where practical.
3. Run the narrowest relevant verification first.
4. Run the broader affected suite before claiming completion.
5. Read the complete failure output and fix the cause, not the test.
6. Update plans, docs, examples, and READMEs to match shipped behavior.

Never report success from a plausible diff alone.

---

## 5. Tools and runtimes

- Prefer running the code to guessing.
- Use repository-local or pinned runtimes and dependency managers.
- Python: use an existing `.venv`; create one only when Python work requires it. Never install into system Python.
- Node: use the repository's pinned runtime and lockfile.
- Prefer existing CLI tools and repository scripts over ad hoc replacements.
- Read full logs, errors, and stack traces.
- For UI changes, verify visually before and after.

---

## 6. Git and session hygiene

- Inspect `git status` before editing and preserve unrelated user changes.
- Never discard, reset, or overwrite changes you did not create.
- Keep diffs surgical and reviewable.
- Use descriptive commit messages with a subject under 72 characters and a body that explains why when needed.
- Do not add assistant attribution or generated-by trailers.
- After two failed corrections on the same issue, stop, summarize the evidence, and ask for direction.
- At the start of a new session, check `https://raw.githubusercontent.com/Juce-me/init_agents_md/main/AGENTS.md` for a newer template. When newer, read `template-migrations.md`, ask before updating, and preserve project-specific sections 10 and 11.

---

## 7. Communication

- Be direct and concise.
- Lead with the assessment or result.
- Separate what existing tools already solve from what custom code still adds.
- Prefer structural critique over surface polish.
- When two paths are viable, state the tradeoff and recommend one.
- Do not hide uncertainty. Say what is unknown and how you will verify it.

---

## 8. When to ask

Ask before proceeding when:

- Two plausible interpretations materially change the output.
- The change affects a versioned, load-bearing, or migration-sensitive contract.
- Credentials, secrets, production resources, or external approval are required.
- The stated goal conflicts with the literal request.

Proceed without asking when:

- The change is trivial and reversible.
- Reading the repository or running a command resolves the ambiguity.
- The user already answered the question in the current session.

---

## 9. Durable learning

When a correction is likely to recur:

1. Decide whether the instruction was missing or ignored.
2. Add or tighten one concrete rule in section 11.
3. Remove duplicate or stale guidance.
4. Keep the rule specific enough to change future behavior.

For significant regressions or repeated misses, review and update the applicable postmortem workflow before related work.

---

## 10. Project context

### Purpose
Custom MCP server for safe GA4 and Google Tag Manager configuration automation. It consumes an approved `*.mcp-execution.yaml` desired-state spec generated by `google-analytics-implementation-planner`, validates it, compares it against current GA4/GTM state, applies approved changes to a new GTM workspace, creates preview versions, and blocks publishing unless explicitly approved.

This is an execution layer, not an analytics planner. It must not invent events, tracking strategy, custom dimensions, or GTM architecture.

### Stack
- TypeScript 5.9.x, ESM, `module/moduleResolution: NodeNext`, `strict + noUncheckedIndexedAccess`.
- Node.js >= 20 LTS.
- npm (lockfile committed; exact pinned versions, no `^` ranges).
- Runtime entrypoint: `dist/server.js` after `npm run build`. The MCP transport is stdio.
- Core deps: `@modelcontextprotocol/sdk` 1.29.0, `googleapis` 172.0.0, `zod` 4.4.3, `yaml` 2.9.0.

### Commands
- Install: `npm install`
- Build: `npm run build` (TypeScript → `dist/`)
- Test: `npm test` (vitest, run-mode; `npm run test:watch` for watch)
- Typecheck: `npm run typecheck`
- Authorize local user OAuth: `npm run login` (build + loopback browser flow)
- Run locally: `npm run dev` (build + run) or `npm run mcp` (run prebuilt). Server speaks MCP over stdio.

### Layout
- Project root: repository root (the `ga4-gtm-config-mcp/` directory).
- Source: `src/` — subdirs `utils/` (errors, redact, stableJson, logger, names), `spec/` (zod schema, readSpec, validateSpec, summarize), `safety/` (9 guards), `auth/` (user OAuth paths/token validation, scope tiers, auth factory), `cli/` (loopback OAuth login), `ga4/` (Admin client + read/upsert wrappers), `gtm/` (Tag Manager client + read/upsert/version/preview/publish wrappers), `planner/` (desiredState, currentState, diff, applyPlan), `tools/` (12 MCP tool registrations).
- Tests: `tests/` (vitest), with `tests/fixtures/specs/*.yaml` for spec fixtures.
- Build output: `dist/` (gitignored).
- Audit log: `.audit/audit-YYYY-MM-DD.log` (gitignored; one JSON line per safety event, written through `utils/redact`).
- Docs: `docs/AGENTS.md` defines agent work artifact rules; agent artifacts live under `docs/agents/features/`, `docs/agents/prompts/`, `docs/agents/bugfixes/`, `docs/agents/reviews/`. `postmortem/` contains the postmortem workflow.

### Conventions
- Reusable rules and design guidance belong at the highest applicable `AGENTS.md`; subfolder `AGENTS.md` files are for local constraints only.
- Keep `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md` aligned at the root and in subfolders; `CLAUDE.md` and `GEMINI.md` should point to the local `AGENTS.md`.
- Agent work artifacts under `docs/agents/` use the `STATUS-summary.md` (or `STATUS-YYYY-MM-DD-summary.md`) naming defined in `docs/AGENTS.md`. Subfolder `AGENTS.md` files cannot redefine this scheme.
- ESM module imports include the `.js` suffix (NodeNext resolution requires this even from `.ts` source). Example: `import { readSpec } from "../spec/readSpec.js";`.
- The `logger` writes to `process.stderr` only — `process.stdout` is reserved for the MCP stdio transport. Do not `console.log` from `src/`.
- Errors surfaced to MCP tool consumers are `MCPError` instances with one of the 12 codes in `src/utils/errors.ts`. Tools serialize them via `error.toJSON()` and return `{ isError: true }`.
- Tool descriptions registered on the MCP server MUST start with one of: `[read-only]`, `[dry-run-capable write]`, `[write — non-live workspace only]`, `[gated]`, `[gated dangerous]`. `assertSafeToolMetadata` enforces this at boot and in tests.

### Repo-specific constraints
- No raw secret values may appear in source, tests, fixtures, or audit log. The `secret_value` field in the spec is constrained at the zod level to the literal string `"NEVER_STORE_SECRET_IN_SPEC"`.
- Runtime authentication uses a local Google user refresh token created by `npm run login`; the operator's user must already have the intended GA4/GTM product permissions.
- `GOOGLE_OAUTH_CLIENT_SECRETS` and `GOOGLE_OAUTH_TOKEN_PATH` are required absolute paths. Login requests all runtime scopes and stores only the validated token record at mode `0600`.
- `INCLUDE_PUBLISH_SCOPE=1` gates publish-mode operations; it does not control which scopes login requests and does not replace the remaining publish guards.
- Every gated dangerous tool requires both a spec-level boolean flag AND a per-call `approval_token`. Gates return ALL failing reasons, never just the first.
- The live/default GTM workspace (`workspaceId: "0"` or name `"Default Workspace"`) is unconditionally rejected by `assertWorkspaceSafe`.
- Tag.type at the zod schema layer is `z.string()` (free); disallowed types (consent, UA-era) are rejected by `validateSpec` with the correct semantic error code, not a generic schema error.

### Git workflow
- Solo dev, feature-branch convention emerging. The full M0–M8 server shipped on branch `feat/m0-m3-validator-slice` (kept past its original M0–M3 scope; commits append). Plan execution commits one task at a time with conventional-commit prefixes (`feat(scope):`, `fix(scope):`, `docs(scope):`, `chore:`, `test(scope):`).
- Do not commit `.audit/`, `dist/`, `.env`, or `node_modules/` (all gitignored).

---

## 11. Project Learnings

- Keep this section short and concrete.
- Add a new line only when the user corrects the agent and the correction is likely to recur.
- Tighten an existing line instead of adding a near-duplicate.
- Delete stale learnings when the underlying issue goes away.
- Document Google Cloud OAuth-client setup separately from the operator's existing GA4/GTM product permissions; OAuth never grants product access.
- Use absolute placeholder paths for the Desktop client JSON, token file, Node executable, and server entrypoint in every MCP configuration example.
- State that `npm run login` requests all runtime scopes and stores a plaintext refresh token at mode `0600`.
- Describe `INCLUDE_PUBLISH_SCOPE` only as the publish-mode operation gate; it does not alter scope acquisition or bypass publish guards.
- Never put real project IDs, account IDs, property IDs, container IDs, user emails, OAuth client values, tokens, or machine paths in public repo docs; use placeholders.

When the user corrects your approach, append a one-line rule here before ending the session. Write it concretely ("Always use X for Y"), never abstractly ("be careful with Y"). If an existing line already covers the correction, tighten it instead of adding a new one. Remove lines when the underlying issue goes away (model upgrades, refactors, process changes).

---
> Source: [Juce-me/ga4-gtm-config-mcp](https://github.com/Juce-me/ga4-gtm-config-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
