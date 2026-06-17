## flowith-terminal-agent

> Operate a local software project through the Flowith Terminal Agent workflow. Use when Codex needs to inspect a repository, read or edit files, search code, run shell commands, or complete small coding tasks in a terminal-first loop with minimal, controlled changes. Prefer this skill when the environment is explicitly built around Flowith models or Flowith API behavior.


# Flowith Terminal Agent

Use this skill to work on a local codebase through a terminal-first coding loop.

Keep the implementation Flowith-first. Preserve only a light internal seam for future providers; do not broaden the user-facing scope unless the task explicitly calls for it.

## Work in this order

1. Understand the user's smallest useful goal.
2. Inspect the relevant files before proposing edits.
3. Search before guessing paths or module names.
4. Prefer local edits over broad rewrites.
5. Verify the change with the smallest meaningful check.
6. Report what changed, what was verified, and what still looks uncertain.

## Use these defaults

- Read a file before editing it.
- Prefer exact, minimal edits to whole-file rewrites.
- Keep interfaces and file layout coherent with the existing project.
- Reuse existing scripts, configs, and entrypoints when possible.
- Ask only when a missing decision would materially change the outcome.

## Handle common tasks

### Repository reading

- Identify the entrypoint, config files, and main modules.
- Summarize structure from actual files, not guesses.

### Small implementation work

- Change one main behavior at a time.
- Preserve unrelated logic.
- Avoid opportunistic refactors unless they are required to finish safely.

### Debugging

- Capture the exact error or failure symptom.
- Locate the responsible file and line when possible.
- Compare expected behavior with actual behavior before patching.

### Validation

- Start with the smallest useful verification.
- Prefer import checks, focused tests, or a single command over a broad validation sweep.
- Do not overclaim from partial evidence.

## Keep the response shape simple

- Lead with the conclusion.
- Then state the concrete change.
- Then state the verification result.
- Keep the tone concise, direct, and helpful.

---
> Source: [M2artiD/flowith-terminal-agent](https://github.com/M2artiD/flowith-terminal-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
