## ai-engineering-workflow

> > Inspired by Andrej Karpathy's CLAUDE.md. This file is the single source of truth

# AGENTS.md — Engineering Workflow Constitution

> Inspired by Andrej Karpathy's CLAUDE.md. This file is the single source of truth
> for how AI coding agents must behave in this repository. Read it completely before
> touching any code.

---

## 1. Core Principle

**SPEC.md is the law. Code is a consequence of SPEC.md.**

Never modify code to fix a bug without first asking: "Does SPEC.md need to change?"
If the spec says the feature should behave one way and the code does it another,
the spec is wrong — update it first, then update the code to match.

---

## 2. Architecture

```
User Request
    │
    ▼
Orchestrator (workflow/orchestrator.py)
    ├── PR Skill         — spec-first code change pipeline
    ├── Deploy Skill     — GitHub Actions → App Store + Device Farm
    └── Figma Skill      — screenshot → production code
    │
    ├── spec_agent       — reads/writes SPEC.md
    ├── review_agent     — simplify → review → copilot-style check
    ├── deploy_agent     — triggers CI/CD and monitors QA
    └── figma_agent      — Claude Vision → component code
    │
    └── Integrations
        ├── GitHub API   — PR creation, Copilot review comments
        ├── JIRA         — ticket creation and linking
        ├── AWS Device Farm — Maestro QA runner
        └── Notifications — Slack/webhook for deploy+QA results
```

---

## 3. The PR Skill — Mandatory Workflow

Every code change MUST follow this exact sequence. No exceptions.

```
1. SPEC_UPDATE   Read SPEC.md. Identify which section covers this change.
                 Update SPEC.md to reflect the new desired behaviour first.
                 Commit: "spec: <what changed>"

2. CODE_CHANGE   Implement the code to match the updated SPEC.md.
                 Only change what is necessary. No cleanup, no refactors
                 beyond the spec change.
                 Commit: "feat|fix|refactor: <what changed>"

3. SIMPLIFY      Run the simplify review: look for dead code, redundant
                 logic, over-engineering. Remove or reduce. Do not add.

4. REVIEW        Self-review: correctness, edge cases, security.
                 Must answer: "Does every changed line match SPEC.md?"

5. COPILOT_REVIEW  Post a structured review comment in the style of
                    GitHub Copilot. Flag: logic errors, type issues,
                    missing error handling, naming violations.

6. TESTS         Run existing tests. If a new spec behaviour was added,
                 add a test for it. Tests must pass before PR.

7. JIRA          Create a JIRA ticket linked to the PR. Title mirrors
                 the PR title. Include spec section reference.

8. OPEN_PR       Open the PR. Copilot auto-review fires automatically
                 via GitHub Actions.
```

---

## 4. Absolute Prohibitions

- **NEVER** modify production database schemas without a migration script.
- **NEVER** commit secrets, API keys, tokens, or credentials to any file.
- **NEVER** skip the SPEC_UPDATE step. Code first = spec debt.
- **NEVER** open a PR with failing tests.
- **NEVER** merge a PR that has an unresolved Copilot review comment.
- **NEVER** deploy without QA passing on Device Farm.
- **NEVER** use `--no-verify` to bypass git hooks.
- **NEVER** add dependencies without updating `pyproject.toml` and noting why in SPEC.md.

---

## 5. Security Requirements

- All external inputs must be validated at the boundary (API layer).
- Authentication tokens live in environment variables only — never in code.
- All SQL uses parameterised queries. No string interpolation.
- Secrets scanning runs on every commit via `gitleaks` hook.
- PII must never appear in logs. Mask before logging.
- HTTPS only. No plain HTTP calls in production code.

---

## 6. Reasoning Model Usage

- Always use the highest reasoning effort available (`xhigh` / `thinking: budget_tokens: 10000+`).
- Prefer `claude-sonnet-4-6` with extended thinking for architecture decisions.
- Use `claude-haiku-4-5-20251001` for fast operations: formatting, JIRA titles, commit messages.
- Capability overhang is real — do not downgrade to a smaller model to save tokens on hard problems.

---

## 7. UI Change Workflow (Figma Skill)

1. Designer exports a Figma screen as PNG/screenshot.
2. Paste the image to the AI agent with the command `figma: implement this`.
3. The Figma skill runs Claude Vision to extract component structure.
4. Agent generates component code targeting the existing design system.
5. If Vision output is uncertain, request Figma Dev Mode JSON and pass that instead.
6. All generated UI goes through the normal PR Skill pipeline.

---

## 8. Deploy Skill

```
deploy [env: staging|production]
    │
    ├── Run tests (fail → abort)
    ├── Trigger GitHub Actions deploy workflow
    ├── Monitor App Store / Play Store submission
    ├── Trigger AWS Device Farm run (Maestro flows in /maestro/)
    ├── Wait for QA result
    └── Send notification (Slack / webhook)
         ✓ PASS → done
         ✗ FAIL → open incident ticket in JIRA
```

Human intervention is only required when:
- Copilot review and the coding agent disagree on the same code line.
- Device Farm QA fails on a flow that has no auto-fix.
- App Store review is rejected.

---

## 9. Commit Message Format

```
<type>(<scope>): <short description>

Types: feat, fix, refactor, spec, test, ci, docs, chore
Scope: the module or feature area (e.g. auth, order, ui/cart)

Examples:
  spec(auth): add biometric login requirement
  feat(auth): implement Face ID login
  fix(cart): correct tax calculation for multi-item orders
  test(auth): add Face ID fallback test
```

---

## 10. File Ownership

| Path | Owner agent | Notes |
|------|-------------|-------|
| `SPEC.md` | `spec_agent` | Only the spec agent writes here |
| `src/` | `review_agent` (gatekeeper) | All changes reviewed before merge |
| `.github/workflows/` | `deploy_agent` | CI/CD files — change with caution |
| `maestro/` | `deploy_agent` | QA flows — update when UI changes |
| `AGENTS.md` | Human only | This file. Humans approve changes. |

---
> Source: [JayKimProject/ai-engineering-workflow](https://github.com/JayKimProject/ai-engineering-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
