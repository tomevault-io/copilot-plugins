## cloud-devops-skill

> Best practices, patterns, and decision frameworks for cloud-native DevOps engineers working with Terraform, Kubernetes, and CI/CD pipelines on Azure and AWS. Use when asked about infrastructure-as-code, container orchestration, deployment automation, cloud architecture decisions, or DevOps workflows.


# Cloud DevOps Skill

A skill for cloud-native DevOps engineers covering Terraform, Kubernetes, and CI/CD pipelines on Azure and AWS.

## Scope

This skill covers:
- **Infrastructure as Code** — Terraform modules, state management, testing, and best practices
- **Kubernetes** — Cluster setup, workload management, networking, security, and GitOps
- **CI/CD Pipelines** — GitHub Actions, Azure DevOps Pipelines
- **Azure** — AKS, networking
- **AWS** — EKS, networking

## Core Principles

1. **Infrastructure as Code first** — all infrastructure changes go through version-controlled IaC
2. **Immutable infrastructure** — prefer replacing over mutating running resources
3. **GitOps** — Git is the single source of truth for desired state
4. **Least privilege** — apply minimal permissions at every layer
5. **Shift left on security** — integrate security checks early in the pipeline

## Step-by-Step Guides

See `references/` for detailed guides on each topic area.

| Topic | Reference File |
|-------|---------------|
| Terraform patterns | `references/terraform-best-practices.md` |
| Kubernetes deployment | `references/kubernetes-deployment.md` |
| CI/CD frameworks | `references/cicd-frameworks.md` |
| Azure patterns | `references/azure-patterns.md` |
| AWS patterns | `references/aws-patterns.md` |
| Security & compliance | `references/security-compliance.md` |
| Cost optimization | `references/cost-optimization.md` |
| Troubleshooting | `references/troubleshooting.md` |

---
> Source: [sebastianczech/cloud-devops-skill](https://github.com/sebastianczech/cloud-devops-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
