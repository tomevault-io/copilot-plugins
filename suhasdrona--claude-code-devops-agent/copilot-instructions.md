## claude-code-devops-agent

> You are working in the `agentic-devops-platform` repository.

# Claude Code Project Memory

You are working in the `agentic-devops-platform` repository.

## Project Purpose

This project demonstrates a production-grade AI-assisted DevOps workflow for generating and validating Terraform and Kubernetes infrastructure changes.

## Safety Rules

Do not bypass validation.
Do not suggest direct production deployment without PR review.
Do not remove policy checks to make a change pass.
Do not use wildcard IAM permissions unless explicitly documented as a temporary exception.
Do not use `latest` image tags.
Do not generate Kubernetes workloads without CPU and memory requests and limits.

## Standard Workflow

1. Read the developer request YAML under `examples/` or `config/`.
2. Generate a change package using:

```bash
agentic-devops generate --request <request-file> --output generated/<change-id>
```

3. Validate the generated change using:

```bash
agentic-devops validate --path generated/<change-id>
```

4. Review generated docs:

```text
docs/CHANGE_SUMMARY.md
docs/RISK_ASSESSMENT.md
docs/ROLLBACK_PLAN.md
docs/PR_DESCRIPTION.md
```

5. Run tests:

```bash
pytest -q
```

## Engineering Standards

- Prefer least-privilege IAM.
- Prefer GitOps pull-request workflows.
- Treat generated infrastructure as a proposal, not an automatic production change.
- Include rollback steps for every change.
- Keep all generated YAML readable and reviewable.
- Use environment-specific settings for dev, stage, and prod.

## Interview Framing

When explaining this project, emphasize safe AI adoption in DevOps:

- AI assists with change generation.
- Deterministic policy checks enforce guardrails.
- GitHub Actions validates the change.
- Humans approve before deployment.
- Rollback and auditability are built into the workflow.

---
> Source: [SuhasDrona/claude-code-devops-agent](https://github.com/SuhasDrona/claude-code-devops-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
