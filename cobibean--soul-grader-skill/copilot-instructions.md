## soul-grader-skill

> Use when grading, reviewing, rewriting, or approving a Hermes Agent SOUL.md. Uses the SOUL.md field-guide research artifacts as the only normative source for what makes a good SOUL.md.


# SOUL Grader

## Linked references

- `references/fleet-soul-grading-workflow.md` — fleet-wide grading workflow: active/retired classification, companion-doc contradiction checks, non-Hermes service handling, durable report shape, and secret-safe archive handling.
- `references/research-deliverable-and-fleet-remediation.md` — deep-research/swarm deliverable pattern, polished static HTML review surfaces, and live fleet SOUL remediation notes.

## Overview

Use this skill to grade a Hermes Agent `SOUL.md`, draft a SOUL review, or turn a weak SOUL into a stronger one. The grading standard is intentionally narrow: **use the linked SOUL.md research artifacts as the only normative source for what makes a good SOUL.md.**

Do not import generic prompt-engineering advice, personal taste, web articles, model-provider docs, or vibes into the grade. You may use tools to read the SOUL being graded and to verify Hermes runtime facts, but quality judgments must come from the research artifacts bundled with this skill.

## Public / SSR publication posture

This is an unofficial community skill for Hermes Agent. The bundled references are intended to be safe for SSR sharing and public release: examples are anonymized, private workspace paths are removed or made relative, and no secrets or live deployment facts should appear in the bundle. If you add new examples or evidence, keep the same standard: cite public Hermes docs/source paths or anonymized patterns, not private customer, user, account, host, or credential details.

## Required source files

Before grading, load at least the grading standard reference:

```text
skill_view(name="soul-grader", file_path="references/soul-md-grading-standard.md")
```

Use the other references when you need more detail or citations:

```text
skill_view(name="soul-grader", file_path="references/soul-md-field-guide.html")
skill_view(name="soul-grader", file_path="references/soul-md-wording-verbiage-layer.md")
```

Source hierarchy for grading:

1. `references/soul-md-grading-standard.md` — canonical grader rubric and procedure.
2. `references/soul-md-field-guide.html` — full research report and evidence ledger.
3. `references/soul-md-wording-verbiage-layer.md` — detailed wording, verbiage, weak/strong examples, and slop detector.

If these references conflict, use the higher-ranked source. If the references are unavailable, stop and report that the grader source bundle is missing instead of grading from memory.

## When to use

Use when the user asks to:

- grade, score, audit, review, or approve a `SOUL.md`
- compare two `SOUL.md` files
- rewrite or improve a `SOUL.md`
- check whether a new agent identity is ready to deploy
- create acceptance criteria for a SOUL file
- diagnose agent identity drift, overreach, genericness, or approval-boundary failures
- decide what belongs in `SOUL.md` versus `CLAUDE.md`, `AGENTS.md`, skills, memory, manifests, or operator guides

Do not use as the sole workflow for:

- installing or configuring Hermes itself — load `hermes-agent`
- full new-agent intake/design — load the project’s new-agent intake or design skill too, if one is available
- authoring a new skill — load `hermes-agent-skill-authoring` too
- editing a user’s actual SOUL file without permission

## Strict source rule

The target `SOUL.md` being graded is evidence, not a standard. Runtime/tool output is evidence about deployment state, not a standard. The bundled references are the standard.

Allowed sources:

- the user-provided SOUL text or file path
- adjacent files only when checking contradictions or placement, such as `CLAUDE.md`, `AGENTS.md`, manifests, roster entries, or operator guides
- live Hermes/runtime output only for runtime hygiene checks
- this skill’s linked references

Not allowed as grading sources:

- generic prompt-engineering heuristics not present in the references
- web search results
- model-provider documentation
- personal preference unless the user explicitly asks for a custom overlay after the source-grounded grade
- unstated assumptions about a deployed agent’s access, host, credentials, or service state

## Grading rubric

Score out of 100 using the reference-defined categories:

| Category | Points | What to evaluate |
|---|---:|---|
| Mission clarity | 15 | Names who/what the agent serves and what outcome matters. |
| Identity + negations | 12 | Says what the agent is and what it must not become. |
| Core thesis | 10 | States the durable decision lens about the user/domain/problem. |
| Optimization hierarchy | 10 | Ranks tradeoffs instead of listing virtues. |
| Hard constraints | 10 | Includes 3–5 true filters with approval/override semantics. |
| Soft preferences | 8 | Separates scoring signals from bans. |
| Authority + escalation | 10 | Allowed / ask-before / never boundaries are clear. |
| Voice + truthfulness | 10 | Covers tone, vocabulary, never-claims, and evidence thresholds. |
| Success / artifacts | 8 | Defines durable/verifiable completion. |
| Artifact separation | 5 | Keeps commands, workflows, secrets, and volatile state elsewhere. |
| Runtime hygiene | 2 | Fits Hermes loading behavior and avoids hidden metadata assumptions. |

Automatic fail conditions from the field guide:

- secrets, tokens, passwords, API keys, or connection strings in SOUL
- false or unverified claims of access, deployment state, health, publication, or authority
- ungated spend, publishing, outreach, destructive edits, production mutations, or customer-visible actions
- cross-client data/credential/workspace contamination
- assuming YAML/frontmatter is hidden from Hermes native SOUL when it is visible prompt text
- contradictions with nearby operating files, manifests, or approval policy

Automatic fail does not always mean `0/100`; it means the SOUL is **not deployable/approvable** until the blocker is fixed. Report the blocker above the score.

## Procedure

1. **Load the grading reference.** Use `skill_view` for `references/soul-md-grading-standard.md`. Load the full HTML or wording layer when you need citations, examples, or wording help.
2. **Get the SOUL.** If the user provides a path, read it with `read_file`. If they provide inline text, grade that. If they ask for the current Hermes profile, verify the live profile path before reading `$HERMES_HOME/SOUL.md`.
3. **Identify scope.** Classify the agent as personal, business/internal, client/business, public/open-source, meta/operator, multi-agent peer, or temporary/tactical. Scope changes the safety bar.
4. **Check automatic fails first.** Search for secrets, false claims, missing gates, cross-client leakage, production/publishing authority, and hidden-frontmatter assumptions.
5. **Score each rubric row.** Award points only for behaviorally specific text in the SOUL. Do not give credit for generic virtues.
6. **Separate SOUL issues from adjacent-file issues.** If a command belongs in `CLAUDE.md`, mark it as artifact separation, not as missing identity. If exact service state belongs in a manifest, do not reward it for being in SOUL.
7. **Write findings as drift risks.** Explain how the weak wording could cause a future session to drift.
8. **Give patches only when useful.** If the user asked for a rewrite, provide a replacement section or full revised SOUL. Otherwise give prioritized fixes.
9. **For “should we make these SOUL updates?” questions, re-read live state before opining.** Fetch/read the current live SOUL and the latest grade/report if available, compare them, and identify which suggested fixes are already present. Recommend a surgical patch, not a full rewrite, unless blockers or scope changes require it. Do not apply the change unless the user explicitly asks you to edit.
10. **Cite the bundled sources.** Cite the linked reference section names and, when useful, file names. Do not cite outside sources.

## Output format: full grade

Use this shape by default:

```md
# SOUL.md grade: [agent/name]

Verdict: [Excellent / Operational / Scaffold / Needs rewrite / Not deployable]
Score: [N]/100
Deployability: [Approved / Approved with fixes / Not approved]
Scope: [personal/business-internal/client-business/public/meta/multi-agent/tactical]

## Automatic blockers

- [None] or [blocker, why it matters, exact evidence]

## Score table

| Category | Points | Score | Notes |
|---|---:|---:|---|
| Mission clarity | 15 |  |  |
...

## Top drift risks

1. [Risk] — [where the SOUL permits drift]
2. [Risk] — [where the SOUL permits drift]
3. [Risk] — [where the SOUL permits drift]

## What is strong

- [Concrete strengths tied to source criteria]

## What to fix first

1. [Highest leverage fix]
2. [Second]
3. [Third]

## Suggested wording

```md
[patch/replacement sections, if requested or obviously helpful]
```

## Source basis

- `references/soul-md-grading-standard.md`: [sections used]
- `references/soul-md-field-guide.html`: [sections used]
- `references/soul-md-wording-verbiage-layer.md`: [sections used]
```

## Output format: quick grade

For quick review requests:

```md
Score: [N]/100 — [verdict]
Deployability: [status]

Biggest issue: [one sentence]
Best thing: [one sentence]
Fix next:
1. ...
2. ...
3. ...
```

## Attachment / HTML delivery pattern

When the user asks to “send me the SOUL,” “put it in an HTML file,” or otherwise wants a reviewable artifact rather than only chat text:

1. Fetch or read the exact target `SOUL.md` first; for deployed fleet agents, prefer the live profile/workspace path from the manifest over stale cached copies.
2. Include both the raw SOUL and the grade in the delivered artifact unless the user explicitly asks for only one.
3. Produce a self-contained HTML file for reading when requested: embedded CSS, verdict cards, score table, automatic blockers, top drift risks, suggested wording, source basis, and raw SOUL in a readable `<pre>` block.
4. Also save a plain Markdown grade report and raw `SOUL.md` beside the HTML when practical, so the user has both human-friendly and copy/paste/edit-friendly forms.
5. Keep the artifact secret-safe: do not include raw credentials from manifests or companion docs; cite credential locations only if needed.
6. Use canonical roster/manifest spelling for the agent name in the report, while noting any user spelling variant only if it could cause confusion.

## Verdict bands

- **90–100 Excellent** — production-grade identity; keep it reviewed as scope changes.
- **75–89 Operational** — usable; patch missing layers before high-risk autonomy.
- **60–74 Scaffold** — serviceable draft; needs constraints, negations, or success artifacts.
- **0–59 Needs rewrite** — rewrite from mission/constraints upward.
- **Not deployable** — any automatic fail remains unresolved.

## Wording standards

Use the wording layer’s rule: operational language beats ornamental persona language.

Prefer:

- `You are [name], [user/client]’s [specific layer/domain] agent.`
- `No [risky action] without [approval/evidence].`
- `Do not claim [status/access/result] until [verification source].`
- `[Durable source] wins for [fact class]; [volatile source] does not.`
- `Public-facing output must [brand/audience rule], not [private voice leak].`

Cut or rewrite:

- `helpful assistant`, `friendly and professional`, `be proactive`, `use best practices`
- `never hallucinate` without evidence thresholds
- mission statements that name vibes instead of outcomes
- tone adjectives without context behavior
- long command/runbook dumps
- exact ports/processes/current state
- secrets or credential values

## Deep research / swarm deliverables

When the user asks for a broad SOUL.md research project, a “swarm,” or a polished review artifact, use the workflow in `references/research-deliverable-and-fleet-remediation.md`:

1. Split research into independent lanes: Hermes semantics, rubric/wording quality, fleet/examples, and deliverable/design.
2. Consolidate lane results into durable class-level guidance rather than a one-session narrative.
3. Produce a self-contained static HTML review surface when the user asks for something to view over Tailscale: embedded CSS, navigation, verdict cards, score tables, before/after examples, and source-basis notes.
4. Keep durable source copies in the corpus (usually `docs/research/`) and convenience/download copies in the active profile cache when needed.
5. Do not claim a Tailscale/local review URL works until the server or `tailscale serve` path has been verified live.

## Live SOUL remediation after grading

When a grading session turns into live profile edits, keep the remediation class-level and evidence-safe:

1. Re-read the relevant roster/manifest and verify the live profile/workspace identity paths before editing. A profile `SOUL.md` may be a real file, symlink, or stale unrostered stub.
2. Back up every target identity file before edits, including both profile and workspace copies when they are intentionally duplicated.
3. Move volatile workflow, startup, provider, memory-service, and runbook detail out of `SOUL.md` into `AGENTS.md`, skills, manifests, or ops docs. Leave `SOUL.md` as the compact identity/authority layer.
4. If a workspace `AGENTS.md` exists but the profile lacks one, wire the profile `AGENTS.md` to the workspace agreement when that is the intended Hermes project-context surface.
5. For profile stubs/anomalies, archive rather than delete when there is any state/history worth preserving, and verify no gateway/service expects the profile before calling it removed.
6. For surgical post-grade patches on a remote/deployed agent, prefer this proof chain before reporting done: fetch/read the live `SOUL.md`; create a timestamped backup outside the committed workspace if possible; apply only the approved identity/authority wording; run a targeted secret scan on the edited file; commit the workspace identity change if the workspace is git-backed; run a fresh/new-session exact-marker smoke that exercises the new rule; verify affected services remain healthy when the edit was on a live profile; then record a concise manifest/ops note with backup path, commit, smoke session/marker, and no secret values.
7. Do not over-edit a strong SOUL just because a grade found small gaps. If the grade says “approved with fixes,” keep the patch to those behavior-changing gaps; avoid turning SOUL into a runbook, manifest, or service-health log.
8. If Hermes command-safety approval times out or blocks a destructive remote batch, stop and wait for the user. Do not retry, rephrase, split, or route around the blocked operation; resume only after the user explicitly re-approves the same scoped action.

## Common pitfalls

1. **Grading from vibes.** Use the bundled references, not generic prompt advice.
2. **Rewarding length.** A long SOUL can still be weak if it lacks mission, negations, gates, and truth rules.
3. **Punishing missing runbooks.** SOUL should not contain every command. Missing commands may belong in `CLAUDE.md`, `AGENTS.md`, skills, or manifests.
4. **Ignoring scope.** Client/business SOUL files need stronger isolation, approval, credential, and handoff language than personal agents.
5. **Treating YAML as hidden.** Hermes native SOUL loading injects SOUL content as prompt text; frontmatter is visible unless an adapter strips/uses it before Hermes sees it.
6. **Forgetting session cache.** A corrected SOUL may not affect an existing Hermes session until a new session/restart/compression rebuild.
7. **Over-rewriting personality.** Keep distinctive voice only when it serves mission and does not leak into public/client contexts improperly.
8. **Missing false-claim thresholds.** Truth policy must name tempting claims and what proves them.

## Verification checklist

- [ ] Loaded `references/soul-md-grading-standard.md` before grading.
- [ ] Read the target SOUL text/file actually being graded.
- [ ] Classified the agent scope.
- [ ] Checked automatic fail conditions.
- [ ] Scored all 11 rubric categories.
- [ ] Separated SOUL problems from CLAUDE/AGENTS/skill/manifest placement problems.
- [ ] Cited only bundled reference artifacts as normative sources.
- [ ] If a rewrite was provided, it preserves real agent facts and does not invent access, authority, credentials, or deployment state.

---
> Source: [cobibean/soul-grader-skill](https://github.com/cobibean/soul-grader-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
