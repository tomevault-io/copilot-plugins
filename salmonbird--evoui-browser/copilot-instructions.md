## evoui-browser

> Evoui Browser is an optional enhancement Skill for `agent-browser`. It records

# AGENTS.md

## Project

Evoui Browser is an optional enhancement Skill for `agent-browser`. It records
browser operations, replays verified mechanical steps, and distills reusable
knowledge from completed tasks. The Agent always decides what to do next.

`skills/evoui-browser/SKILL.md` is the operating contract that an Agent follows
when using the Skill. The runtime implementation lives in
`skills/evoui-browser/scripts/evoui/`. This file defines the rules for changing
this repository; see `README.md` for installation and usage.

## Public repository layout

```text
evoui-browser/
├── .github/workflows/ci.yml
├── skills/evoui-browser/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   ├── references/{agent-browser,runtime,knowledge,evolution}/
│   └── scripts/
│       ├── evoui.py
│       └── evoui/
├── tests/
├── .gitignore
├── AGENTS.md
├── README.md
├── SECURITY.md
├── LICENSE
└── NOTICE
```

This tree lists only stable entries in the public repository. Git-ignored local
data and development material are intentionally omitted.

## Product principles

1. **Stay decoupled from `agent-browser`; pass through by default.** Without
   `--evoui`, Evoui must preserve the original invocation exactly: argv,
   standard streams, TTY behavior, signals, and exit code must match a direct
   `agent-browser` invocation.
2. **Accelerate repeated operations; do not make business decisions.** Evoui
   executes browser operations whose intent is already known. The Agent chooses
   the target, interprets the page, and decides what to do next.
3. **Keep Replay scoped.** Evoui does not try to replay every action in a task.
   Partial Replay is useful when it removes mechanical work and cleanly returns
   uncertain work to the Agent. Replay must stop when the page does not match or
   the result cannot be verified. A click or submission that may already have
   taken effect must not be retried mechanically. Making unstable or
   judgment-heavy actions replayable is outside the product scope.
4. **Success does not imply that a task is worth learning.** Knowledge should be
   saved only when the scope is clear, the steps are likely to recur, and the
   result can be verified.
5. **Do not hard-code product exceptions for individual websites.** Site,
   selector, and product-specific differences belong in scoped knowledge, not
   in product code.

## Development conventions

- Persist data as JSON. Give every persisted format change an explicit version,
  and write files atomically.
- When replacing a mechanism, remove the old implementation completely. Do not
  preserve compatibility for hypothetical callers or leave dead code and stale
  tests behind.
- Test public behavior and failure boundaries, not private helpers or incidental
  implementation structure.
- Keep documentation focused on product contracts, operational boundaries, and
  third-party references that code cannot express. Do not add documentation that
  merely restates the implementation.
- Preserve source notices and license attribution when modifying upstream-derived
  material under `references/agent-browser/`.
- Use English in public project files, code comments, docstrings, and test
  descriptions unless a file is explicitly localized.
- Managed mode may temporarily write raw argv, stdin, and stock JSON results to
  local traces. Normal-mode terminal cleanup is the single final trace-lifecycle
  step. Evoui attempts it after Finish succeeds—or after Finish and recovery have
  conclusively stopped—and after every Evolution lane has committed, been
  abandoned, or reached a terminal failure. This includes Runs where
  `trace_persistence_failed` prevents Finish. Successful cleanup deletes the
  content-bearing Run trace. To retain a complete debug record, the same non-empty
  `EVOUI_DEBUG_PLAN_TRACE` value must remain set throughout the Managed Run, every
  Plan, and terminal cleanup; Evoui then retains the full Run/Evolution trace and
  Plan JSONL without automatic rotation. Cleanup reduces retained data; it is not
  redaction or privacy isolation. Never put real credentials in code, tests,
  logs, or commits.

## Validation

This repository has no project package manager. Run the checks directly with
system Python 3.10 or later:

```bash
python3 -m compileall -q skills/evoui-browser/scripts tests
python3 -m unittest discover -s tests -p 'test_*.py'
```

The fake `agent-browser` used by the test suite verifies only process boundaries.
Changes to browser capabilities must also be tested with the real
`agent-browser`, a visible Chrome window, and a real website. Passing mock tests
does not prove that a browser capability works.

## Ask before editing

Get confirmation before:

- adding, removing, or changing an external dependency;
- changing public CLI behavior or the Skill's trigger and operating contract;
- changing the knowledge or trace persistence format or on-disk layout;
- changing credential handling, redaction, permissions, or other
  security-sensitive behavior; or
- performing a destructive or irreversible operation.

---
> Source: [Salmonbird/evoui-browser](https://github.com/Salmonbird/evoui-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
