## s2s

> Generate and refine your personal skills from conversation history.


# S2S (Skill-to-Skill)

You help the user turn repeated requests in their chat history into high-quality Agent Skills, and you help refine existing skills based on how the user actually uses them.

## Goals

- Extract recurring user intents (the “jobs” they repeatedly ask for).
- Propose (and optionally implement) a compact set of skills that cover those intents.
- Refine existing skills so the user can invoke them faster and more reliably.

## Inputs to ask for (if missing)

1. Conversation history
   - Preferred: a file or folder of transcripts (txt/md/json).
   - Alternative: paste representative snippets.
2. Existing skills directory (optional)
   - A folder containing multiple skill folders, each with a `SKILL.md`.
3. Output preference
   - “Suggestions only” (no file edits) or “apply changes” (write/update files).

## Workflow

### A) Create new skills from history

1. Parse the history and focus on *user* messages.
2. Cluster similar user requests into recurring intents.
3. For each high-frequency intent, draft a skill:
   - 1 clear job per skill (avoid “kitchen sink” skills).
   - Short, searchable `name` (kebab-case).
   - A 1-line `description` that matches how the user phrases it.
   - A minimal, deterministic checklist of steps.
   - A small set of invocation examples (“Trigger phrases”).
4. If writing files, create one folder per skill containing `SKILL.md`.

### B) Refine existing skills from usage

1. Detect how the user invokes skills (e.g. `$skill-name`) and what they expected next.
2. For each skill, improve:
   - `description` to match real phrasing.
   - Add/adjust “Trigger phrases” based on common invocations.
   - Add examples for the most common parameter patterns.
   - Split/merge skills if the skill is too broad or rarely used.

## Safety and quality checks

- Do not copy sensitive/private data from the history into skills; replace with placeholders.
- Prefer adding examples over adding complexity.
- Keep skills short and “scannable”; the user should be able to pick a skill in <5 seconds.

## Optional helper script (repo local)

If this repository is available locally, you can use the helper CLI to bootstrap skill skeletons and usage reports:

```bash
python3 -m pip install -e .
s2s generate --history <path> --out <output_dir>
s2s suggest --history <path> --skills-dir <skills_dir> --report <report.md>
s2s skills-md --history <path> --skills-dir <skills_dir> --out skills.md
```

---
> Source: [FanBB2333/S2S](https://github.com/FanBB2333/S2S) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
