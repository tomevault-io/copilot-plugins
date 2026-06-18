## skillproof

> Verify whether a candidate coding-agent skill helps in a specific repository on a specific task slice with acceptable risk. Use when an agent needs to compare baseline versus skill-enabled runs, score repo compatibility, inspect instruction and script risk, and leave behind auditable verification artifacts in `.skillproof/`.


# SkillProof

Verify a candidate coding-agent skill before adopting it.

Default invocation:

> Use `$skillproof` to verify whether this candidate skill helps in this repo on these tasks with acceptable risk.

## Inputs

SkillProof expects four concrete inputs:

1. a candidate skill path or local reference
2. a target repository
3. a task suite
4. a verification config

Use `.skillproof/verification.yaml` and `.skillproof/tasks.yaml` inside the target repo as the default contract.

## Non-Negotiables

1. Compare baseline and treatment fairly.
2. Keep the same task slice, validators, and command envelope except for skill enablement.
3. Prefer deterministic validation commands over judge-style scoring.
4. Ground compatibility and risk claims in visible evidence.
5. Use fresh temporary repo copies for execution so baseline and treatment start from the same source state.
6. Leave auditable artifacts behind in `.skillproof/`.
7. Do not describe the skill as universally good or bad. State where it helps, where it hurts, and where confidence is limited.

## Operating Sequence

### 1. Parse the candidate skill

- Read `SKILL.md`, metadata, scripts, and references as needed.
- Summarize the skill's assumptions about language, framework, repo type, and task shape.

### 2. Profile the target repo

- Detect the repo's primary language and framework.
- Infer major task surfaces such as frontend, backend, CLI, library, or infra.

### 3. Normalize the verification inputs

- Load `.skillproof/verification.yaml`.
- Load `.skillproof/tasks.yaml`.
- Reject ambiguous or missing deterministic validation commands in v1.

### 4. Analyze risk

- Score shell execution, remote fetching, bundled scripts, broad writes, env or secret access, hidden external references, self-escalation, and misleading claims.
- Record evidence for every elevated signal.
- Load [references/risk-model.md](references/risk-model.md) when classifying or explaining risk.

### 5. Analyze compatibility

- Compare skill assumptions to repo signals.
- Produce matched signals, mismatched signals, and a recommendation.
- Load [references/compatibility-model.md](references/compatibility-model.md) when the fit is ambiguous.

### 6. Run baseline and treatment

- Execute both conditions against fresh repo copies.
- Keep task definitions and validators identical.
- Preserve condition-level stdout, stderr, exit codes, validation results, and latency.

### 7. Summarize utility

- Measure pass-rate delta, failed-validation delta, regression count, latency overhead, and repeat stability.
- Identify where treatment helps and where it regresses.

### 8. Generate artifacts

- Write the Markdown and JSON report.
- Write machine-readable risk, compatibility, task-result, and run-manifest artifacts.
- Follow [references/artifact-contract.md](references/artifact-contract.md) for file locations.

## Anti-Patterns

- Do not compare different task slices between baseline and treatment.
- Do not use a different validator set for the two conditions.
- Do not call a skill "safe" without showing which risk signals were checked.
- Do not call a skill "compatible" without stating the matched and mismatched signals.
- Do not reduce the report to one magical score with no explanation.
- Do not let baseline or treatment mutate the source repo used for the comparison.

## Reference Loading Guide

- Read [references/verification-method.md](references/verification-method.md) when you need the detailed run protocol.
- Read [references/risk-model.md](references/risk-model.md) when you need the risk taxonomy and escalation thresholds.
- Read [references/compatibility-model.md](references/compatibility-model.md) when you need the compatibility heuristics.
- Read [references/artifact-contract.md](references/artifact-contract.md) when you are creating or reviewing `.skillproof/` outputs.

---
> Source: [gangj277/SkillProof](https://github.com/gangj277/SkillProof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
