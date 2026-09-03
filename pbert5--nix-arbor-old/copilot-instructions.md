## nix-arbor-old

> This repository follows Codex's `AGENTS.md` guidance: keep durable common

# Agent Working Agreement

This repository follows Codex's `AGENTS.md` guidance: keep durable common
instructions here, and put path-specific rules in closer `AGENTS.md` files.
Nested files add to or override this file for their subtree.

## Required First Steps

1. Read `/home/example/.codex/RTK.md`.
2. Use `rtk` for every shell command. This is mandatory.
3. For coding tasks, load Serena's instructions first and keep using
   project-aware tools during investigation and editing.
4. Discover and use applicable MCP servers for current external facts instead
   of relying on stale model knowledge or broad web searches.
5. For Nix, NixOS, Home Manager, flakes, overlays, derivations, dev shells, or
   repo-layout work, invoke the `nixos` skill.
6. For flake architecture, module-boundary decisions, cross-class composition,
   or experiment-local flake design, invoke the `dendritic` skill as an
   appendix to `nixos`.

## Shell Commands

- Prefix every shell command with `rtk`.
- Examples: `rtk git status`, `rtk nixos-rebuild build --flake .#desktoptoodle`.
- If raw command behavior is required, use `rtk proxy <command>`.
- Verify availability with `rtk --version` when uncertain.
- CODEX: dont run background commands, just runn them in the foreground

## Repo Defaults

- Prefer declarative Nix configuration over ad-hoc imperative scripts.
- Keep system behavior encoded in Nix whenever practical.
- Favor small, composable modules with clear ownership.
- Prefer the simplest minimal change. Use native or standard-library features
  before new dependencies or abstractions, and avoid speculative generality.
- Beyond the core host age encryption key installed during enrollment, manage
  machine setup and cluster behavior declaratively through the flake.
- you can be lazy but dont leave things ugly

## Tooling

- Use `tool_search` when an applicable MCP tool is deferred or not visible.
- Use Serena for semantic code navigation and edits when it fits the file type.
  - Use Serena for Python work when semantic navigation saves context or reduces mistakes:
    - Prefer Serena when a task touches 3 or more files, requires finding definitions or callers, involves refactors, or needs project-structure understanding.
    - Use Serena for class/function/method navigation, reference tracing, signature changes, and low-token edits in larger Python subprojects.
    - Do not force Serena for tiny scripts, simple syntax fixes, formatting-only edits, or traceback-local bugs where the failing line is already clear.
    - Pair Serena with normal verification tools such as pytest, ruff, mypy, or the narrowest relevant project check.
- for manipulating nix code:
  - Serena for repo navigation and low-token edits.
  - nixd for Nix-aware LSP behavior, options, packages, and cross-file analysis
  - MCP-NixOS for accurate package/options lookup instead of hallucinated option names. It provides current NixOS packages, options, Home Manager options, nix-darwin, flakes, Noogle, and related Nix resources.
  - 
- Use shell tools for builds, tests, Git, generated output, and cases where
  semantic tools do not fit.
- Use `apply_patch` for small line-oriented edits.
- When work genuinely requires the whole repository, read
  `repomix-output.xml` instead of scanning every source file individually.

## Verification

- Before handoff, run the narrowest check that exercises the changed behavior.
- Do not run deployments or activation commands such as `clusterctl deploy` or
  `nixos-rebuild switch` unless the user explicitly grants permission for that
  specific deployment.
- Prefer build and evaluation checks when deployment permission has not been
  granted.

## CodeQL

- Use the flake-pinned scanner for local CodeQL runs:
  `rtk nix run .#codeql-scan`.
- Run one language when narrowing investigation:
  `rtk nix run .#codeql-scan -- --language actions`,
  `rtk nix run .#codeql-scan -- --language python`, or
  `rtk nix run .#codeql-scan -- --language javascript-typescript`.
- Results are written under `.artifacts/codeql/results` by default. Use
  `--output-dir <dir>` for disposable verification runs.
- The local wrapper accepts `--ram <MB>` and `--threads <N>`, or `CODEQL_RAM`
  and `CODEQL_THREADS`. JavaScript/TypeScript analysis may need the default
  4096 MB RAM budget or higher on smaller machines.
- Keep `.github/codeql/codeql-config.yml` focused on portable path filters.
  Put query-suite choices in the GitHub workflow or the local scanner command.

### Viewing and triaging results

The terminal summary shows only the first ~20 findings. The full result set
is in the SARIF files under `.artifacts/codeql/results/`. Parse all findings:

```bash
python3 -c "
import json, sys
with open('.artifacts/codeql-scan/results/python.sarif') as f:
    sarif = json.load(f)
for r in sarif['runs'][0]['results']:
    loc = r['locations'][0]['physicalLocation']
    path = loc['artifactLocation']['uri']
    line = loc['region']['startLine']
    print(r['ruleId'], f'{path}:{line}', r['message']['text'][:80])
"
```

Pull live alerts from GitHub (full severity + state):

```bash
rtk gh api repos/pbert5/nix-arbor/code-scanning/alerts --paginate \
  --jq '.[] | [.rule.id, .rule.severity, .most_recent_instance.location.path + ":" + (.most_recent_instance.location.start_line | tostring), .state] | @tsv'
```

The public GitHub repo for this flake is `pbert5/nix-arbor`.

### Suppressing false positives

Use `# lgtm[rule-id]` as an **inline trailing comment** on the flagged line:

```python
path.write_text(public_key, encoding="utf-8")  # lgtm[py/clear-text-storage-sensitive-data]
```

This format is processed by the `AlertSuppression.ql` queries, which our local
scanner includes alongside the main query suite (python and javascript-typescript).
A suppressed finding shows `suppressed: true` in the SARIF output.

Rules of thumb:
- Inline (same line) only — preceding-line comments shift line numbers and are not recognized.
- `paths-ignore` in `.github/codeql/codeql-config.yml` is the right tool for whole directories (e.g., experiments, vendored code).
- Reserve `# lgtm[...]` for individual false positives where the surrounding code is correct and the concern is structurally impossible.

## Commits

- Commit completed changes before handoff unless the user explicitly asks not
  to commit.
- When working from a nested subunit, still commit the completed work in this
  flake repository before handoff.
- Use a concise subject and a detailed commit-message body that records the
  motivation, important implementation details, and verification performed.

---
> Source: [pbert5/nix-arbor-old](https://github.com/pbert5/nix-arbor-old) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
