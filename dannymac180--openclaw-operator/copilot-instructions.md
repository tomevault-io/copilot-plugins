## openclaw-operator

> This repository may contain an OpenClaw installation, workspace, or supporting code.

# AGENTS.md

## Purpose

This repository may contain an OpenClaw installation, workspace, or supporting code.

When a task involves OpenClaw setup, troubleshooting, skills, cron jobs, channels, providers, integrations, or runtime validation, consult `skills/openclaw-operator/SKILL.md` before planning changes.

## Operating rules

- Inspect before editing.
- Prefer official OpenClaw CLI commands and local docs over memory.
- Make the smallest reversible change that satisfies the task.
- Preserve existing structure and naming.
- Validate changes before finishing.
- Never expose secrets in output, logs, diffs, or committed files.

## OpenClaw-first workflow

1. Identify whether this is a source repo, live workspace, or both.
2. Locate the active config, workspace, and relevant skill/integration files.
3. If the `openclaw` CLI exists, start with:
   - `openclaw status`
   - `openclaw gateway status`
   - `openclaw docs <topic>`
   - `openclaw doctor`
4. Read `skills/openclaw-operator/SKILL.md` for the detailed task playbook.
5. Use `skills/openclaw-operator/references/validation_checklist.md` before finishing.
6. Make targeted edits.
7. Re-run the most relevant validation commands.

## Editing guidance

- Do not rewrite whole config files unless necessary.
- Prefer adding narrowly scoped files over embedding large instructions into root docs.
- Keep skill definitions concise and move lengthy references into supporting files.
- Use `skills/openclaw-operator/assets/` for reusable examples, templates, and scaffolds.
- If command names or behavior differ from expectation, trust the installed version and local docs.

## Response format

When you complete work, summarize:

1. What you found
2. What you changed
3. How you validated it
4. Any follow-up risks or next steps

---
> Source: [DannyMac180/openclaw-operator](https://github.com/DannyMac180/openclaw-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
