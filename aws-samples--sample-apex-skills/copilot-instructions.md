## sample-apex-skills

> - Prefer the AWS MCP Server for AWS interactions — it provides scoped

# AWS & EKS Guidance

- Prefer the AWS MCP Server for AWS interactions — it provides scoped
  IAM access and CloudTrail-compatible audit logging. If unavailable,
  use the AWS CLI directly.
- Before starting a task, check whether a relevant AWS skill is available.
  Prefer skill guidance over general knowledge.
- Prefer the EKS MCP Server for live cluster interactions — it provides
  Kubernetes API access (read by default, write if configured), upgrade
  insights, and cluster discovery. If unavailable, use kubectl and AWS
  CLI directly.
- Before starting an EKS task, check whether a relevant APEX skill is
  available. Skills are installed at `~/.claude/skills/` (Claude Code)
  or `~/.kiro/skills/` (Kiro). Prefer skill guidance over general
  knowledge.
- When uncertain about specific AWS details (API parameters, permissions,
  limits, error codes), verify against documentation rather than guessing.
  State uncertainty explicitly if you cannot confirm.
- When uncertain about specific EKS details (add-on versions, API parameters,
  Karpenter behavior, VPC CNI settings), verify against upstream source repos
  rather than guessing. Cross-reference the upstream repo to confirm the
  behavior still holds in the current release:
  - Karpenter: `github.com/aws/karpenter-provider-aws`
  - VPC CNI: `github.com/aws/amazon-vpc-cni-k8s`
  - AWS Load Balancer Controller: `github.com/kubernetes-sigs/aws-load-balancer-controller`
  - CoreDNS: `github.com/coredns/coredns`
  - EKS AMI: `github.com/awslabs/amazon-eks-ami`
  - EKS Terraform module: `github.com/terraform-aws-modules/terraform-aws-eks`
- When creating infrastructure, prefer infrastructure-as-code (Terraform with
  `terraform-aws-modules/terraform-aws-eks`, AWS CDK, or CloudFormation) over
  direct CLI commands.
- When working with infrastructure, follow AWS Well-Architected Framework
  principles.
- Never execute destructive operations (node drain, cluster delete, force
  eviction) without explicit user confirmation.
- Never output secrets, credentials, or kubeconfig contents in responses.
- Avoid non-ASCII characters (including em dashes) in AWS resource names
  and descriptions. Use hyphens instead.

---
> Source: [aws-samples/sample-apex-skills](https://github.com/aws-samples/sample-apex-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
