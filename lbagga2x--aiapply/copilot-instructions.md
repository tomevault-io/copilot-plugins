## aiapply

> This file is read by Claude at the start of every conversation to restore full context.

# CLAUDE.md — AIApply Project Memory

This file is read by Claude at the start of every conversation to restore full context.

---

## What This Project Is

**AIApply** — an AI-powered job application platform built for personal use first (spouse's job search), with a plan to turn it into a SaaS product later.

**Core flow**: Upload CV → Set career goals → AI finds matching jobs → AI tailors CV per company → User reviews → Applies

**Key differentiator vs aiapply.co**: Quality over quantity. 20 targeted applications with company-specific CVs, not 200 generic blasts. Career goal alignment built in.

---

## Owner Context

- **Who uses it**: Spouse is the primary user right now (job searching)
- **Developer**: AWS Solutions Architect certified. Comfortable with frontend (React/Next.js). Python/AI agents are newer territory.
- **Budget**: As cheap as possible. Currently ~$5–7/month (just Claude API — everything else is AWS free tier)
- **Goal**: Personal tool now → SaaS later
- **Auto-apply mode**: Review-first by default (user approves each tailored CV before submission). Toggle to full auto in settings.
- **Claude API keys**: Already set up and working

---

## What Has Been Built (Phase 1 — Complete)

### Frontend — Next.js 15 + TypeScript + shadcn/ui + Tailwind
Located at: `/Users/l.bagga/Documents/AIApply/frontend/`

| Page | File | Status |
|------|------|--------|
| Landing page | `app/page.tsx` | ✅ Done |
| Sign up | `app/signup/page.tsx` | ✅ Done |
| Log in | `app/login/page.tsx` | ✅ Done |
| Onboarding (CV upload + career goals) | `app/onboarding/page.tsx` | ✅ Done |
| Dashboard (Kanban board) | `app/dashboard/page.tsx` | ✅ Done |
| Application detail + CV diff | `app/applications/[id]/page.tsx` | ✅ Done |
| Settings (goals + auto-apply toggle) | `app/settings/page.tsx` | ✅ Done |

Key support files:
- `lib/auth.ts` — Amplify/Cognito helpers (signIn, signUp, getAuthToken)
- `lib/api.ts` — API client (getUploadUrl, uploadFileToS3, getCareerGoals, getApplications)
- `components/amplify-provider.tsx` — client-side Cognito config wrapper
- `components/ui/` — shadcn/ui components (button, card, badge, dialog, tabs, etc.)

### Backend — Python 3.12 Lambda functions
Located at: `/Users/l.bagga/Documents/AIApply/backend/lambdas/`

| Lambda | Trigger | What it does |
|--------|---------|-------------|
| `cv_analyst` | S3 ObjectCreated | Parses PDF/DOCX → extracts structured data via Claude Sonnet |
| `api` | API Gateway HTTP | Routes all `/api/*` REST requests |
| `job_scout` | SQS | Scrapes LinkedIn/Indeed (JobSpy) → scores matches via Claude Haiku |
| `cv_tailor` | SQS | Rewrites CV per company → diff + cover letter via Claude Sonnet |

### Infrastructure — Terraform
Located at: `/Users/l.bagga/Documents/AIApply/infrastructure/terraform/`

| Module | What it creates |
|--------|----------------|
| `modules/storage` | S3 (CV bucket) + 4 DynamoDB tables |
| `modules/cdn` | S3 (frontend) + CloudFront distribution |
| `modules/auth` | Cognito User Pool + App Client |
| `modules/queue` | SQS job-scout + cv-tailor queues (with DLQs) |
| `modules/api` | Lambda functions + API Gateway HTTP API + IAM roles + SSM Parameter (API key) |
| `modules/monitoring` | CloudWatch dashboards + alarms |

Environment: `infrastructure/terraform/environments/dev/`

### CI/CD — GitHub Actions
Located at: `/Users/l.bagga/Documents/AIApply/.github/workflows/`

| Workflow | Trigger | Does |
|----------|---------|------|
| `ci.yml` | Every PR | Lint frontend (ESLint) + lint backend (Ruff) + build check |
| `deploy.yml` | Push to main | Deploy Lambda zips + S3 sync + CloudFront invalidation |
| `terraform.yml` | Changes to `infrastructure/` | Plan on PR, apply on merge |

Uses GitHub OIDC → IAM role (no long-lived AWS keys in GitHub secrets).

### Local Dev
- `docker-compose.yml` — LocalStack (emulates S3, DynamoDB, SQS, Lambda, SSM)
- `scripts/localstack-init.sh` — creates all local AWS resources on startup
- `frontend/.env.local.example` — template for environment variables

---

## Tech Stack Summary

| Layer | Technology | Monthly Cost |
|-------|-----------|-------------|
| Frontend | Next.js 15, TypeScript, shadcn/ui, Tailwind | $0 (CloudFront free tier) |
| Backend | Python 3.12, AWS Lambda | $0 (1M req/mo free) |
| Database | DynamoDB (4 tables, PAY_PER_REQUEST) | ~$0 (<$0.01/mo at personal scale) |
| Queue | SQS | $0 (1M msgs/mo free) |
| Auth | AWS Cognito | $0 (50K MAU free) |
| File Storage | S3 | $0 (5 GB free tier) |
| AI | Claude API — Sonnet for CV work, Haiku for scoring | ~$5–7/mo |
| IaC | Terraform | $0 |
| CI/CD | GitHub Actions | $0 |
| Monitoring | CloudWatch Log Groups only (alarms + dashboard removed) | $0 |
| **Total** | | **~$5–7/mo** |

---

## DynamoDB Schema

| Table | PK | SK | Key Fields |
|-------|----|----|-----------|
| `aiapply-dev-users` | `userId` | — | careerGoals (object) |
| `aiapply-dev-cvs` | `userId` | `cvId` | s3Key, structuredData (JSON string), skills (list), isPrimary |
| `aiapply-dev-job-listings` | `jobId` | — | title, company, description, url, source, matchScore |
| `aiapply-dev-applications` | `userId` | `applicationId` | jobId, cvId, status, companyName, jobTitle, matchScore, careerAlignmentScore, tailoredCvKey, cvChanges, coverLetter |

Application statuses: `pending` → `matching` → `tailoring` → `review` → `submitted` → `interview` → `offer` / `rejected`

---

## Agent Pipeline (How It Works End-to-End)

```
1. User uploads CV to S3 (presigned URL from /api/upload-url)
        ↓ S3 ObjectCreated event
2. cv_analyst Lambda — Claude Sonnet extracts structured JSON → DynamoDB

3. User sets career goals on /onboarding → saved to DynamoDB

4. (Manual for now) SQS message sent to job-scout queue
        ↓ SQS trigger
5. job_scout Lambda
   - JobSpy scrapes LinkedIn + Indeed
   - Claude Haiku scores each job: matchScore + careerAlignmentScore
   - Filters: both scores >= 70, no dealbreakers
   - Saves top 10 to DynamoDB as Application records (status: "tailoring")
   - Sends to cv-tailor SQS queue

        ↓ SQS trigger
6. cv_tailor Lambda
   - Claude Sonnet rewrites CV specifically for that company/role
   - Returns: tailored CV JSON + changes list + ATS score + cover letter
   - Saves tailored CV to S3 (tailored/{userId}/{appId}/cv.json)
   - Updates Application status → "review"

7. User sees application on dashboard
   - Reviews CV diff (green=added, yellow=modified, red=removed)
   - Clicks "Approve & Submit" → status → "submitted"
```

---

## What Is NOT Done Yet (Phase 2)

- [ ] **SQS trigger from career goals save** — currently must manually send SQS message to start job scout
- [ ] **Submit Agent** — Playwright Lambda that auto-fills and submits job application forms
- [ ] **Email notifications via SES** — "Your CV has been tailored for Stripe"
- [ ] **CloudFront custom domain** — currently uses default CloudFront URL
- [ ] **Mobile responsive polish** — works but not optimised for mobile
- [ ] **Real Terraform deploy** — modules written but `terraform apply` not run yet against real AWS
- [ ] **GitHub Actions secrets** — OIDC role and repo variables not configured yet

---

## What Is NOT Done Yet (Phase 3 — SaaS)

- [ ] Stripe payments (Free / Pro $29 / Premium $59)
- [ ] Multi-user support (currently designed for 1–2 users)
- [ ] Migrate DynamoDB → RDS PostgreSQL + pgvector (for vector similarity search)
- [ ] Scale Lambda → ECS Fargate (for long-running Playwright sessions)
- [ ] Admin panel (user management, AI cost tracking, pipeline monitoring)
- [ ] Analytics dashboard (interview rates, skill demand, salary insights)

---

## Key File Paths

```
/Users/l.bagga/Documents/AIApply/
├── CLAUDE.md                                          ← this file
├── README.md                                          ← project overview
├── docker-compose.yml                                 ← LocalStack local dev
├── scripts/localstack-init.sh                         ← creates local AWS resources
├── frontend/
│   ├── .env.local.example                             ← copy to .env.local
│   ├── app/page.tsx                                   ← landing page
│   ├── app/onboarding/page.tsx                        ← CV upload + career goals
│   ├── app/dashboard/page.tsx                         ← kanban board
│   ├── app/applications/[id]/page.tsx                 ← CV diff view
│   ├── lib/auth.ts                                    ← Cognito helpers
│   └── lib/api.ts                                     ← API client
├── backend/
│   ├── requirements.txt                               ← pip freeze (full deps)
│   └── lambdas/
│       ├── cv_analyst/handler.py                      ← parse CV with Claude
│       ├── api/handler.py                             ← REST API routes
│       ├── job_scout/handler.py                       ← scrape + score jobs
│       └── cv_tailor/handler.py                       ← tailor CV per company
└── infrastructure/terraform/
    ├── modules/{storage,cdn,auth,queue,api,monitoring}/main.tf
    └── environments/dev/{main,variables,outputs,backend}.tf
```

---

## How to Resume Development

### Run the app locally
```bash
cd /Users/l.bagga/Documents/AIApply
docker-compose up -d          # start LocalStack
npm run dev --prefix frontend  # start frontend on :3000
```

### Next immediate task
The most impactful next step is running `terraform apply` against real AWS so the app is actually deployed and usable for the spouse's job search:

```bash
cd infrastructure/terraform/environments/dev
terraform init
terraform apply -var="anthropic_api_key=sk-ant-YOUR_KEY"
# Then copy outputs to frontend/.env.local
```

---

## Decisions Already Made (Don't Re-discuss)

- **AWS serverless** (Lambda + DynamoDB) not ECS/RDS — keeps cost near $0 at personal scale
- **Review-first mode** is default — user approves each tailored CV before submission
- **Claude API** (not OpenAI) — better quality/cost, prompt caching saves ~40%
- **JobSpy** (open source) for job scraping — free, covers LinkedIn + Indeed
- **Cognito** for auth — free up to 50K MAU, cert-aligned
- **No admin panel yet** — monitor via CloudWatch + Anthropic dashboard
- **DynamoDB** (not PostgreSQL) for now — free tier, no vector search needed at personal scale
- **GitHub Actions OIDC** for CI/CD — no long-lived AWS credentials
- **SSM Parameter Store** (not Secrets Manager) for Anthropic API key — SSM standard params are free; Secrets Manager costs $0.40/secret/month
- **DynamoDB PAY_PER_REQUEST** — the 25 RCU/WCU free tier only applies to PROVISIONED mode; PAY_PER_REQUEST has no free tier but costs <$0.01/month at personal scale (fractions of a cent)
- **S3 versioning** with a 30-day lifecycle rule on non-current versions — prevents old CV versions accumulating cost

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lbagga2x)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lbagga2x)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
