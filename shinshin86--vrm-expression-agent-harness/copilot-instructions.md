## vrm-expression-agent-harness

> You are an AI coding agent working in the VRM Expression Agent Harness.

# AGENTS.md

You are an AI coding agent working in the VRM Expression Agent Harness.

## Mission

Given a user-owned VRM file, produce the best model-specific AITuber expression extension that the file can honestly support. Follow the model-specific workflow: inspect, reason, derive from existing morph deltas, verify, self-review, and prepare for WebUI preview.

## Non-Negotiable Rules

- Read this file, `docs/agent-operating-manual.md`, and `docs/success-contract.md` before touching the target VRM.
- Never overwrite the input VRM.
- Never assume a universal conversion algorithm is sufficient.
- Do not invent geometry or expressions without source morph material.
- Keep source VRM/VRMA/GLB files and generated model files in the local workspace unless the user confirms they may be shared.
- Keep reports focused on project-relative paths and model facts.
- Every model-specific transformation script must live under that model's workspace output/scripts directory.
- Run the self-review loop in `harness/self-review-loop.md` at least three times.

## Required Outputs

For each target model, create an output folder containing:

- extended VRM file with a new name,
- `INSPECTION.md`,
- `PLAN.md`,
- `EXPRESSIONS.md`,
- `VERIFY.md`,
- `SELF_REVIEW.md`,
- model-specific scripts used for inspection, derivation, and verification.

## Required Expression Classification

For each requested expression, classify it as one of:

- `already_available`
- `aliasable`
- `derivable`
- `unsupported`

Unsupported is acceptable when the model lacks source morphs. Do not hide this.

## Required Review Gates

1. Inspection review before writing derivation code.
2. Plan review before modifying any output VRM.
3. Structural verification after derivation.
4. Visual WebUI review after generation.
5. Asset and license review before final response.

## Final Response Requirements

Report:

- input and output filenames,
- VRM version and license metadata summary,
- expressions added, aliased, already available, and unsupported,
- verification result,
- WebUI preview instructions,
- any residual risks.

---
> Source: [shinshin86/vrm-expression-agent-harness](https://github.com/shinshin86/vrm-expression-agent-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
