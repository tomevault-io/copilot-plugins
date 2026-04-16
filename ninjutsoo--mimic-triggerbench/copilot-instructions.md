## mimic-triggerbench

> - Treat every concrete user correction or repeated complaint as a signal to improve rules, even if the user does not explicitly ask for a rule change.


# Meta Rule
- Treat every concrete user correction or repeated complaint as a signal to improve rules, even if the user does not explicitly ask for a rule change.
- For each such signal, classify the root cause as `missing`, `unclear`, `conflicting`, `outdated`, `not followed`, or `not a rules problem`.
- When the root cause is `missing`, `unclear`, `conflicting`, or `outdated`, proactively update or add rules so the same mistake is less likely next time, focusing on patterns that could recur (not one-off edge cases).
- Apply the smallest durable change in `.cursor/rules/`: prefer editing an existing rule over adding a new one; if needed, add or move to a better-fit rule file; delete stale or redundant rules when they no longer reflect behavior.

## Mandatory self-improvement protocol (do this in every chat)

### Step 0 — detect a learning signal

A **learning signal** is present if the user message contains any of:
- A direct correction: "don't do X", "do Y instead", "you missed X", "stop doing X", "I don't want you to…"
- A complaint about assistant behavior: "this is wrong", "you keep doing X", "it's not getting updated", "why aren't you…"
- A repeated preference + mismatch: the user restates a preference that was already stated earlier because it wasn't followed.

If **no learning signal**, do not change rules.

### Step 1 — classify root cause

Choose exactly one:
- `missing`: no rule exists for the preference/constraint.
- `unclear`: a rule exists but is ambiguous or underspecified (weak triggers, vague wording).
- `conflicting`: two rules would lead to different behavior in the same situation.
- `outdated`: rule no longer matches current project reality.
- `not followed`: rule exists and is clear, but the assistant didn't comply.
- `not a rules problem`: user is asking for a one-off or subjective change that should not become a persistent rule.

### Step 2 — patch rules immediately (default behavior)

If root cause is `missing|unclear|conflicting|outdated|not followed`, then in the same turn:
- Apply the **smallest durable** edit to `.cursor/rules/` so the same mistake is less likely next time.
- Prefer editing the **most specific existing** rule file; fall back to editing this file if nothing more specific exists.
- Keep edits minimal: tighten triggers, add a clear constraint, add an example if needed.

If root cause is `not a rules problem`, do not edit rules.

### Step 3 — report learning applied (in the assistant reply)

After any rule edit, include a short section (1–3 bullets) in the user-visible reply:
- The learning signal (brief quote)
- The root-cause tag
- The rule file(s) edited

### Guardrails (must not learn the wrong thing)
- Never relax or contradict project invariants in `.cursor/rules/10-project-invariants.mdc`.
- Don't "learn" one-off preferences that are unlikely to recur.
- Don't create new top-level docs/files as part of learning unless an existing project plan explicitly requires it.
- Never create new `.mdc` rule files — always merge into the most relevant existing rule file.

## Rules: 
- Prefer minimal, necessary files and docs. Do not add new `.md`, `.mdc`, or other files unless explicitly required by the project plan.
- After a phase or subphase in `IMPLEMENTATION_TASKS.md` is fully implemented and has passing tests, append a small `[DONE]` marker to that phase heading so future sessions know where to resume.
- Do not edit `.cursor/plans/*.plan.md` files unless the user explicitly asks to update the plan itself (implementation should not mutate the approved plan artifact).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ninjutsoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
