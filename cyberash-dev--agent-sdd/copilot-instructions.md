## agent-sdd

> This file is for any AI coding agent (Claude Code, Codex, Cursor,

# AGENTS.md

This file is for any AI coding agent (Claude Code, Codex, Cursor,
Aider, et al.) working in this repository. It describes the rules of
the road, the canonical sources of truth, and the boundaries you must
not cross.

`CLAUDE.md` (project-level, sibling of this file) is the
Claude-specific extension. The two are kept in sync — start here, and
read `CLAUDE.md` if you are Claude Code.

---

## Rule zero — spec is the source of truth

`spec/spec.md` is the authoritative specification. Every behavior in
the code traces to a normative ID (`BEH-*`, `CTR-*`, `INV-*`, `POL-*`,
`CST-*`, `EXT-*`, `GEN-*`, `IMP-*`, `DLT-*`).

If a code change does not match the spec, the spec change comes
**first**, in the same PR, and the `sdd lint` gate is run before
implementation. There is no exception. "Quick hack to fix the build"
is not an exception either — `sdd lint` will refuse it.

---

## Don't self-approve

You are an agent. The `--approver` argument of `sdd approve` MUST be a
human identity. The CLI rejects agent identities (Claude, Codex,
spec-author-bot, bot:*, sdd-cli itself) case-insensitively. This is
`SDD §7.5` and `INV-005`. Don't try to argue your way around it; don't
suggest a placeholder human name; just stop and ask the user.

If you are about to write `lifecycle.status: approved` directly into
spec.md without going through `sdd approve`, STOP. That is a forbidden
back-door. The only way to flip status is the CLI, with a real human
approver, which writes a typed `approval_record` block atomically
(`INV-007`).

---

## What you may change without ceremony

- New `proposed` IDs (Behaviors, Contracts, Invariants, etc.) with
  `lifecycle.status: proposed` and `approval_record:
  not_applicable_for_proposed`. These are sandbox until a human
  approves them.
- Tests under `tests/unit/` and `tests/integration/`. Always add a
  test for new behavior; bug fixes get a regression test that fails
  before the fix.
- Implementation files for the slice that owns a `proposed` ID.
- Documentation (`README.md`, `CLAUDE.md`, `AGENTS.md`, `CHANGELOG.md`).

## What you may NOT change

- The `lifecycle.status` field of any `approved`/`deprecated`/`removed`
  record (only `sdd approve`/`sdd deprecate`/`sdd remove` may flip it).
- The `approval_record` block of any approved record.
- Surface contracts (`SUR-*`, `CTR-*`) without a `Delta` and a major or
  minor bump on the owning Surface.
- The architecture invariant `INV-004` / `CST-003` (vertical slice +
  hexagonal, no global layer folders, no cross-feature imports).
- The built-in git subcommand allowlist `EXT-001` / `POL-002`, or the
  pluggable-VCS contract (`SUR-017`, `CTR-031` / `CTR-032`, `POL-004`),
  without a `Delta`.

---

## Architecture cheat sheet

```
src/
  cli.ts + cli*.ts                       # composition-root entry layer (parse, wire, route)
  features/{token,check,refresh,lint,approve,ready,record,install,…}/
    domain/                              # pure logic
    application/                         # use cases
    ports/{inbound,outbound}/            # interfaces
    adapters/{inbound,outbound}/         # only here you may import node:*
  shared/
    domain/                              # cross-feature primitives, incl. PartitionGrammar
                                         # (CST-007 marker + CTR-015 regex) and the Vcs
                                         # port + VcsConformance (pure types/validator)
  vcs/                                   # built-in GitVcs + resolveVcs loader (node:* allowed);
                                         # wired only from the composition root
```

- Inside a feature: `adapters → ports → application → domain`. No
  arrows pointing the other way.
- Cross-feature imports: forbidden. Reach via `src/shared/domain/`.
- `src/shared/domain/` imports no `node:*` module **except**
  `node:crypto` inside `src/shared/domain/Token.ts`.
- `src/vcs/` may import `node:*` and `src/shared/domain`, but never a
  feature; only the composition root (`cli*`) imports `src/vcs`.
- New feature? Mirror the same layout.

`tests/unit/layer-imports.test.ts` mechanically enforces all of the
above. Run it after every meaningful refactor.

---

## Commands you should know

```sh
npm install                  # bootstrap
npm run tsc                  # type-check (no emit)
npm run test:unit
npm run test:integration
npm test                     # both
npm run build                # produces dist/cli.js with shebang + +x

# the tool against itself (the repo's own spec.md)
node dist/cli.js --help
node dist/cli.js token
node dist/cli.js check
node dist/cli.js refresh
node dist/cli.js lint        # MUST exit 0 in CI
node dist/cli.js ready       # SHOULD exit 0 in CI (gate-3)

# navigate / edit the spec one record at a time (no whole-file read)
node dist/cli.js record list                # index: id · type · status · title
node dist/cli.js record get <id>            # one record, verbatim
node dist/cli.js record set <id> --from-file body.yaml      # draft/proposed only
node dist/cli.js record add --after <id> --content "$BODY"  # new draft/proposed record

# distribute the SDD methodology rules (+ Claude hooks) into the agent config
node dist/cli.js install all --dry-run      # preview, write nothing
node dist/cli.js install claude             # ~/.claude (@import, skill, 2 hooks)
node dist/cli.js install codex              # ~/.codex/sdd + AGENTS.md reference
node dist/cli.js install all --scope project  # into THIS repo (default scope=user)
```

CI runs `tsc && test:unit && test:integration && build && sdd lint && sdd ready`.
Make all six green before you stop.

**Prefer `sdd record` over `Read`/`Edit` on `spec/spec.md`.** Use
`record list`/`get` to find and read individual records (read-only,
`INV-002`) instead of loading the whole file, and `record set`/`add` to
change one draft/proposed record atomically (`INV-015`) instead of
hand-editing. `set`/`add` refuse `approved`/`deprecated`/`removed`
records — those still go through a `Delta` + `sdd approve`/`sdd finalize`.

---

## Tests that must never regress

| Test                                             | Guards                                        |
|--------------------------------------------------|-----------------------------------------------|
| `tests/unit/layer-imports.test.ts`               | `INV-004` / `CST-003` (architecture)          |
| `tests/integration/git-shim-allowlist.test.ts`   | `POL-002` (git command allowlist)             |
| `tests/integration/vcs-external-adapter.test.ts`  | `SUR-017` / `POL-004` (pluggable VCS loading)  |
| `tests/integration/fs-readonly.test.ts`          | `INV-002` / `INV-008` / `INV-009` / `POL-001` |
| `tests/integration/lint-and-approve.test.ts`     | `BEH-011..016`, `INV-005..007`, `DLT-001`     |
| `tests/unit/constraints.test.ts`                 | `CST-001/002/004/005/006`, `EXT-002`          |
| `tests/unit/MarkerParser.test.ts`                | `CST-007` (marker grammar incl. multi-segment)|
| `tests/unit/Rewrite.test.ts`                     | `INV-007` (atomic flip; flat + nested form)   |

If any of these break under your change, your change is wrong — fix
the change, not the test. (Adjusting one of these tests requires a
spec update, like everything else.)

---

## Gotchas

- The composition root is the `cli*` entry layer: `src/cli.ts` plus
  `cliParse.ts`, `cliParseApprove.ts`, `cliDispatch.ts`, and `cliTypes.ts`.
  These modules parse argv and wire inbound + outbound adapters. Construct
  adapters only in this entry layer, never inside a feature. The layer is
  split across files because `max-lines` (350) will not fit the whole entry
  in `cli.ts` alone.
- The `yaml` package (`^2`) is the only YAML parser allowed (`CST-004`).
  Don't add `js-yaml` or hand-roll a parser.
- `mechanism` in `schema/sdd.config.schema.json` is a grammar
  (`^[a-z][a-z0-9_]*$`), pinned to `git_tree_hash_v1` when `vcs` is absent
  or `"git"` (`CST-005`). The active VCS adapter declares its own
  `mechanism`; the `vcs` config field selects the adapter — built-in git
  (`src/vcs/GitVcs.ts`) or an external package loaded by
  `src/vcs/resolveVcs.ts` (see `docs/writing-vcs-adapters.md`).
- The runtime dep tree must stay `{yaml}` only (`CST-006`). No
  third-party glob library — the matcher under
  `src/features/ready/domain/PartitionResolver.ts` is hand-rolled.
- `Surface` records use semver (`"0.1.0"` etc.); every other normative
  template uses integer `version` (`SDD §1.5`).
- `approval_record` lives nested under `lifecycle.approval_record:` in
  `sdd-cli`'s spec; the parser also accepts a top-level form. Both
  work — and the rewriter (`Rewrite.ts`) supports both shapes too.
- `sdd approve` rewrites `lifecycle.status` and `approval_record`
  atomically (`INV-007`). Never split them.
- The marker grammar (`CST-007`) is one-or-more colon-separated
  lowercase tokens (`my-partition:BEH-001`,
  `bridge:commands:CON-004`). Single source of truth lives in
  `src/shared/domain/PartitionGrammar.ts`. Never re-write the regex
  in two places — the duplication is exactly what landed this gap
  pre-v0.3.0.
- The marker scanner is byte-level (`CST-007` rationale). String
  literals containing `@covers` patterns inside `.test.ts` files
  WILL be picked up. Construct fixture markers via `"@cov" + "ers ..."`
  splits so the source bytes don't match the regex.
- `sdd ready` does NOT execute tests (`INV-008`); it byte-scans test
  files for `@covers` markers and is read-only on the working tree
  (`INV-009`).
- `sdd install` is the one command that may write outside the user home:
  `--scope user` (default) writes only under `~/.claude/**` and
  `~/.codex/**` (or `$SDD_INSTALL_HOME`); `--scope project` writes only the
  agent-config set under `process.cwd()` (`./CLAUDE.md`, `./AGENTS.md`,
  `./.claude/**`, `./.codex/**`) and never `spec/*.md`, `.sdd/config.json`,
  `.git`, or source (`INV-016` / `POL-003` / `POL-001`). Project-scope hook
  commands use `$CLAUDE_PROJECT_DIR/...` (not absolute paths). The
  scope→layout split lives in `installLayout()` in `InstallPlan.ts`; the
  memory file, `@import`/reference prefix, and hook command all branch on
  scope. It is plan-then-apply (a missing packaged source aborts before any
  write) and idempotent (managed blocks replaced in place, hooks deduped by
  matcher+command).
  The artifact list lives in `rules/manifest.json`, never hardcoded in
  `src/features/install/` (`CST-008`). The package must ship `rules/`
  (`package.json#files`).

When in doubt: read `spec/spec.md` Appendix B (Section ↔ §-rule
cross-reference) for which section governs which template, and run
`sdd lint --format=json` — every rule id is stable and self-explanatory.

---

## When you finish a task

1. Did you update `spec/spec.md` for the behavior change? If no, go back.
2. `npm run tsc` clean?
3. `npm test` green?
4. `npm run build` clean?
5. `node dist/cli.js lint` exit 0?
6. `node dist/cli.js ready` exit 0?

If all six — you're done. Tell the user what changed (one line per
file ideally), surface anything surprising, and stop.

---
> Source: [cyberash-dev/agent-sdd](https://github.com/cyberash-dev/agent-sdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
