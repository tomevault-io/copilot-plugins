## protected-config-files

> Certain config files that enforce code quality gates are permanently read-only for agents. NEVER modify: .eslintrc.cjs, .dependency-cruiser.cjs, .husky/pre-commit, or the check/lint/typecheck scripts in package.json.


# Protected configuration files — never modify

The following files enforce automated code quality gates (linting, architecture checks, type checking, pre-commit hooks). They must **never** be edited, created, deleted, or worked around by any agent or subagent under any circumstance.

## Permanently read-only files

| File | Gate it enforces |
|------|-----------------|
| `.eslintrc.cjs` | ESLint rules |
| `.dependency-cruiser.cjs` | Architecture / import boundary rules |
| `.husky/pre-commit` | Pre-commit hook runner |

## Protected scripts in `package.json`

The following `scripts` entries in `package.json` must never be removed, renamed, or weakened:

```
"lint":       "expo lint"
"lint:arch":  "depcruise src config app --config .dependency-cruiser.cjs --validate --output-type err"
"typecheck":  "tsc --noEmit"
"check:arch": "npm run lint:arch"
"check":      "npm run lint && npm run typecheck && npm run lint:arch"
"prepare":    "husky"
```

## Hard rules

- **Do not edit** any of the files listed above for any reason, including to make a task "pass" faster.
- **Do not relax** ESLint rules, depcruise rules, or pre-commit hooks to bypass a failing check.
- **Do not remove or skip** `--validate`, `--noEmit`, or any flag that enforces strictness in those scripts.
- **Do not add** `--no-verify` or equivalent flags to git commands to bypass the pre-commit hook.
- If a check fails, **fix the source code** so it conforms to the rules — never adjust the rules to fit broken code.
- This restriction applies to the agent, all subagents, and any automated workflow operating in this repo.

---
> Source: [espressif/esp-rainmaker-home](https://github.com/espressif/esp-rainmaker-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
