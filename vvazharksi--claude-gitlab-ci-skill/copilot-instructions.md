## claude-gitlab-ci-skill

> One sentence describing what this skill does and when to use it. Be specific — this is how Claude decides whether to load the skill.


# [Your Skill Name]

One paragraph describing what this skill encodes. Focus on **how to think**, not just what to do. The goal is consistent output even when the specific situation differs from what you've seen before.

---

## Updating This Skill

When this skill is updated (manually or by an automated agent after a change), follow these rules before writing anything:

1. **Check before adding.** Read the full current content of SKILL.md, all files in `references/`, `symptoms.json`, and any scripts first. If the pattern, concept, or symptom already exists — do not add it again. Update in place.
2. **No update if the change is redundant.** If the diff introduces something already covered, skip it entirely. Write nothing and say why.
3. **Scope updates to what actually changed.** Don't rewrite sections that are unaffected.
4. **Distinguish behavior change from rename.** If something was renamed, update every reference across all files.
5. **Script updates only when the contract changes.** Only touch scripts if the detect output would differ or scaffold would produce wrong files.
6. **New failure symptoms belong in `symptoms.json`, not here.** Append a new entry with `symptom`, `cause`, `fix`, and `keywords` fields. `keywords` must be substrings that appear verbatim in error output. Do not add symptom rows to SKILL.md.
7. **New reference files must be registered in `references/index.json`.** Each entry needs a `file` path, a `description`, and a `tags` array of lowercase keywords that appear in error output or context when that reference is relevant. A file not in the index will never be used.

---

## Core Philosophy

Before generating anything, internalize these principles. They explain **why** the patterns exist.

### 1. [First Principle Name]

Explain the principle and why it exists. Connect it to a real failure mode it prevents.

### 2. [Second Principle Name]

Explain the principle and why it exists.

### 3. [Third Principle Name]

Add as many principles as needed. Each should answer "why" not just "what".

---

## Pattern Library

Each pattern: what it solves, when to apply it, how to adapt it.

---

### Pattern: [Pattern Name]

**What it solves:** One sentence on the specific problem this addresses.

**When to apply:** The condition under which this pattern is the right choice.

**Core structure:**
```
Show the minimal, canonical form of this pattern.
```

**Adapting:** How to apply this to variations of the base case. What changes, what stays the same.

---

### Pattern: [Another Pattern Name]

**What it solves:** ...

**When to apply:** ...

**Core structure:**
```
...
```

**Adapting:** ...

---

## Structural Decisions

When setting up a new [thing], decide:

**[First decision the agent must make]?**
- Option A → consequence / when to choose it
- Option B → consequence / when to choose it

**[Second decision]?**
- ...

---

## Generating a New [Thing]

Gather from the user (or infer from context):

| Question | Why it matters |
|---|---|
| [Question 1] | [Why the answer changes what you generate] |
| [Question 2] | [Why the answer changes what you generate] |

Then apply the patterns above. The specific output is derived from the principles — not copied from examples.

**For the common [default case]**, the concrete defaults are:
- [Setting 1]: [value]
- [Setting 2]: [value]

---

## Self-Healing: Diagnosing Failures

When an agent uses this skill to fix a broken [thing], use this decision rule for CONFIDENCE:

**CONFIDENCE: HIGH** — the fix is directly derivable from:
- A matching symptom injected into the prompt from `symptoms.json` (error text matches, even partially), OR
- A named pattern in this skill, OR
- A core philosophy principle that was clearly violated

**CONFIDENCE: LOW** — the fix requires reasoning from scratch with no grounding in the patterns, principles, or injected symptoms above.

The distinction is: **did this skill teach you the fix, or did you invent it?** If the skill taught it — even indirectly via a principle — use HIGH.

Relevant symptoms from `symptoms.json` are pre-filtered by keyword and injected directly into each diagnosis prompt. Do not maintain a symptom table in this file — all symptoms live in `symptoms.json`.

---

## Using the Generator Script

For mechanical scaffolding of new [things]:

```bash
python scripts/scaffold.py detect ./my-project
python scripts/scaffold.py scaffold ./my-project --option value --dry-run
```

The script handles the common [default] case. For other variations, use the pattern library above to reason about what to generate — the script is a shortcut, not a constraint.

---
> Source: [vvazharksi/claude-gitlab-ci-skill](https://github.com/vvazharksi/claude-gitlab-ci-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
