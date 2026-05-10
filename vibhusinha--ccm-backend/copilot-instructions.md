## ccm-backend

> - **6 microservices** (auth, clubs, matches, scoring, communication, commerce) running as Docker containers on a single EC2 instance

# CCM Backend - Project Context

## Architecture
- **6 microservices** (auth, clubs, matches, scoring, communication, commerce) running as Docker containers on a single EC2 instance
- **Nginx** on the EC2 box acts as API gateway/reverse proxy, routing by URL prefix to containers on ports 8001-8006
- **TLS** via Certbot/Let's Encrypt on the EC2 instance
- **RDS PostgreSQL 16** in a private isolated subnet (only resource not on the EC2 box)
- **Frontend** (ccm-ui, sibling dir) is an Expo React Native SPA, deployed to S3 + CloudFront
- **Region**: eu-west-2 (London)
- **Domain**: crickitup.com (Namecheap)
- **Staging API**: https://api-staging.crickitup.com (TLS via Let's Encrypt, auto-renews)

## CDK Infrastructure (infrastructure/)
- IaC: AWS CDK (TypeScript) in `infrastructure/`
- 4 stacks, all prefixed `ccm-backend`:
  - `ccm-backend-{env}-vpc` — VPC, 2 AZs, public + isolated subnets, no NAT gateway
  - `ccm-backend-{env}-compute` — EC2 (t4g.micro staging / t4g.small prod), ECR repo, IAM role, Elastic IP, Nginx config
  - `ccm-backend-{env}-database` — RDS PostgreSQL 16, Secrets Manager credentials
  - `ccm-backend-{env}-frontend` — S3 + CloudFront for the web SPA
- Internal resource naming convention: `ccm-backend-{env}-*`

## Staging Environment
- **API**: https://api-staging.crickitup.com
- **EC2**: `i-0c5da302a6719dd85` (t4g.micro), Elastic IP `13.41.112.35`
- **RDS**: `ccm-backend-staging-db.cdesquo0u4du.eu-west-2.rds.amazonaws.com`
- **DB Credentials**: Secrets Manager `ccm-backend-staging-db-credentials`
- **ECR**: `418884736370.dkr.ecr.eu-west-2.amazonaws.com/ccm-backend-staging`
- **Frontend**: S3 `ccm-backend-staging-web` → CloudFront `d3uwovn5d8td7k.cloudfront.net`
- **TLS**: Let's Encrypt certificate, auto-renews via certbot scheduled task

## Current State / TODO
- **CDK stacks deployed to staging** (Feb 2026) — all 4 stacks CREATE_COMPLETE
- **Deploy workflow enabled** — auto-deploys on push to main
- **GitHub secrets set**: `STAGING_EC2_INSTANCE_ID`, `ECR_REGISTRY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- **DNS configured**: `api-staging.crickitup.com` → `13.41.112.35`
- **TLS configured**: certbot/Let's Encrypt on Nginx
- **All 6 services running** on staging EC2

### TODO
- Production environment not yet deployed
- Configure `api.crickitup.com` DNS once production EIP exists
- Frontend (ccm-ui) not yet deployed to S3/CloudFront

## GitHub Workflows
- `.github/workflows/deploy.yml` — builds Docker images, pushes to ECR, deploys via SSM
  - Auto-deploys on push to main (staging)
  - Manual `workflow_dispatch` for choosing environment/service
- ECR repo name: `ccm-backend-{env}`
- Docker container names on EC2: `ccm-{service}` (e.g., ccm-auth, ccm-clubs)
- Env files on EC2: `/opt/ccm-backend/{service}.env` (e.g., auth.env, clubs.env)

## Key Config Files
- `infrastructure/bin/app.ts` — CDK app entry point
- `infrastructure/lib/stacks/` — all 4 CDK stack definitions
- `infrastructure/cdk.json` — CDK context (env configs)
- `nginx/api.conf` — Nginx routing config
- `.github/workflows/deploy.yml` — CI/CD pipeline

---
> Source: [vibhusinha/ccm-backend](https://github.com/vibhusinha/ccm-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
