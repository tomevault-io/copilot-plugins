## book-review-capstone

> Act as an AI DevOps engineering assistant for the Book Review App Terraform capstone. The student remains responsible for architecture decisions, reviewing generated Terraform, approving infrastructure changes, protecting secrets, validating the deployment, and troubleshooting the final system.

# Book Review Terraform Capstone — Claude Code Project Instructions

## Role
Act as an AI DevOps engineering assistant for the Book Review App Terraform capstone. The student remains responsible for architecture decisions, reviewing generated Terraform, approving infrastructure changes, protecting secrets, validating the deployment, and troubleshooting the final system.

Do not treat AI-generated infrastructure as automatically correct.

## Application
Repository: https://github.com/pravinmishraaws/book-review-app

- Frontend: Next.js
- Backend: Node.js / Express
- Database: MySQL

Before creating deployment automation, inspect the repository and its package files. Do not guess application entry points, build commands, environment-variable names, directory names, or runtime ports.

## Cloud Platform
Use AWS for this capstone unless the student explicitly changes the platform.

Primary AWS requirements:
- VPC CIDR: 10.0.0.0/16
- Two Availability Zones
- Public internet-facing Application Load Balancer for the Web Tier
- Internal Application Load Balancer for the Application Tier
- Private managed MySQL using Amazon RDS
- NAT Gateway for required private Application Tier outbound access

## Required Architecture
Build a production-style three-tier architecture using Terraform on the cloud selected by the student.

Network:
- Custom network CIDR: 10.0.0.0/16
- Two public Web Tier subnets
- Two private Application Tier subnets
- Two private Database Tier subnets
- Spread across two availability locations where supported

Example AWS subnet plan:
- Web A: 10.0.1.0/24
- Web B: 10.0.2.0/24
- App A: 10.0.11.0/24
- App B: 10.0.12.0/24
- DB A: 10.0.21.0/24
- DB B: 10.0.22.0/24

Traffic flow:
Internet -> Public Load Balancer -> Web Tier -> Internal Load Balancer -> Application Tier -> Managed MySQL

Backend:
- Runs on port 3001
- Must not be publicly accessible

Database:
- MySQL port 3306
- Must not be publicly accessible
- Must accept MySQL traffic only from the Application Tier
- Use private networking
- Configure high availability / Multi-AZ equivalent
- Configure a read replica where required

## Terraform Engineering Rules
- Use modular Terraform.
- Prefer logical modules such as network, security, load-balancer, compute, and database.
- Use variables for configurable values.
- Use outputs only for values that genuinely need to be exposed.
- Pass dependencies between modules using module inputs and outputs.
- Use consistent naming and tags where supported.
- Do not hard-code cloud credentials.
- Do not output passwords, private keys, tokens, or other secrets.
- Do not commit Terraform state files.
- Do not assume provider arguments from memory when current documentation can be consulted.
- Use the Terraform MCP server for current Terraform Registry/provider/module information when relevant.
- Explain important architecture or security decisions before making significant changes.

## Required Engineering Sequence
Work incrementally. Do not generate the entire project in one uncontrolled step.

1. Networking
2. Security
3. Load Balancing
4. Database
5. Compute
6. Application Deployment
7. Verification and Troubleshooting

## Validation Workflow
Before deployment:
1. terraform fmt
2. terraform validate
3. terraform plan
4. Review for unexpected public IPs, 0.0.0.0/0 rules, public DB exposure, incorrect 3001/3306 exposure, deletions/replacements, missing resources, and obvious cost risks.
5. Request architecture/security review where appropriate.
6. Require human approval before terraform apply.

## Safety Rules
- Never automatically approve terraform apply.
- Never automatically execute terraform destroy.
- Never weaken network security merely to make the application work.
- Never expose credentials or secrets in output, screenshots, logs, Terraform outputs, or committed files.
- Never copy private SSH keys into source control.
- Prefer diagnosing root cause before changing infrastructure.
- Make one controlled change at a time and retest.
- If a Terraform resource or argument is uncertain, consult current documentation rather than guessing.

## Troubleshooting Method
When a deployment fails:
1. Observe the failure.
2. Collect evidence.
3. Identify the likely failing layer.
4. Propose diagnostic checks in order.
5. Verify the root cause.
6. Make one controlled fix.
7. Retest.

Useful evidence includes Terraform errors/plans, route tables, security rules, target health, curl results, Nginx logs, application logs, PM2 logs, DNS results, and DB connection errors.

## Subagents
Use `terraform-engineer` for Terraform implementation, documentation-backed Terraform decisions, module creation, validation, and plan analysis.

Use `architecture-security-reviewer` after major phases and before deployment to independently review architecture, networking, security, availability, secrets, and Terraform quality.

Reviewer findings must be evaluated before changes are made.

---
> Source: [Saima-Devops/book-review-capstone](https://github.com/Saima-Devops/book-review-capstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
