## claude

> **ALL infrastructure changes MUST be implemented via Terraform.** Never make infrastructure changes directly via AWS CLI, API, or console and call it done — the change must be codified in `.tf` files so it is reproducible, reviewable, and version-controlled.

# Erudite-s Project Defaults

## Rule #1: Terraform-First and Branch-PR Workflow (ABSOLUTE - STRICT - NEVER VIOLATE)

**ALL infrastructure changes MUST be implemented via Terraform.** Never make infrastructure changes directly via AWS CLI, API, or console and call it done — the change must be codified in `.tf` files so it is reproducible, reviewable, and version-controlled.

**ALL code changes MUST go through a branch and PR.** Never push directly to `main`. The workflow is always: branch → commit → push → open PR → review/CI passes → merge.

### NEVER DO
- **NEVER make infrastructure changes only via CLI/API** without updating Terraform to match
- **NEVER push directly to main** — always use a branch and PR
- **NEVER consider a manual change "done"** — it must be in Terraform before it's done
- **NEVER skip the PR process** even for "small" changes

### ALWAYS DO
- **Implement infrastructure changes in Terraform first**, then apply
- **If an emergency change was made via CLI/API**, immediately create a PR to import/codify it in Terraform
- **Use `terraform import`** to bring manually-created resources under Terraform management
- **Create a branch, push, and open a PR** for every change, no exceptions

## Fully Automated Build & Deploy Pipeline (ABSOLUTE - STRICT - NEVER VIOLATE)

**ALL build, push, and deploy operations MUST be 100% automated via Terraform and GitHub Actions pipelines. Zero manual intervention, zero SSH, zero manual container builds.**

### The ONLY Permitted Workflow
```
Code change → git commit → git push → Open PR → Merge PR → GitHub Actions pipeline:
  1. Build Docker image(s)
  2. Push to ECR
  3. Sync configs to S3
  4. Trigger EC2 service restart (via SSM Run Command, not SSH)
  5. Verify health checks pass
```

### NEVER DO (applies to ALL erudite-s repos and child folders)
- **NEVER run `docker build` locally** to build deployment images — the pipeline builds them
- **NEVER run `docker push` locally** to push images to ECR — the pipeline pushes them
- **NEVER SSH into EC2** to run `docker compose up`, restart services, set env vars, or sync configs
- **NEVER run `aws ecr get-login-password` locally** as a deployment step
- **NEVER run `aws s3 sync` locally** to deploy configs — the pipeline syncs them
- **NEVER manually export secrets/env vars on a server** to make Docker Compose work
- **NEVER use `sudo -E docker compose up -d`** over SSH — this is ClickOps
- **NEVER run `terraform init/plan/apply` locally** with AWS profiles — always trigger via GHA deploy workflow
- **NEVER use local AWS_PROFILE to test apps** designed for EC2 IAM instance roles — test on the actual EC2 via SSM

### ALWAYS DO
- **Fix the pipeline** if it doesn't exist or is broken — creating automation is the job, not working around it
- **Use Terraform** for infrastructure changes (EC2, RDS, S3, IAM, security groups, etc.)
- **Use GitHub Actions** for build, push, deploy, and verification steps
- **Use SSM Run Command or user-data.sh** for server-side operations — never interactive SSH
- **Use AWS Secrets Manager** for runtime secrets — never hardcode or manually export them
- **Verify deployments via health check endpoints** in the pipeline — never by SSHing in to check

### Formpak AI PoC (`erudite-s/formpak-ai-poc`) — EC2 GPU Architecture
- **Instance type: g6.xlarge** (NVIDIA L4, 23GB VRAM) — single EC2, not SageMaker
- **Architecture: 4 containers** on one instance — inference (BGE-M3 + 2 rerankers + TATR), postgres (PG18 + pgvector + AGE + db2_fdw), db2, app (FastAPI)
- **Inference API**: port 8080, endpoints: `/health`, `/models`, `/predict`
- **App API**: port 8100, 21 endpoints including `/ai/query`, `/ai/chat`, `/ai/agent`
- **LLM**: Anthropic API direct (ChatAnthropic via `create_llm()`), no Bedrock
- **Single AWS account: 572444560619 / eu-west-2** — Terraform in `formpak-ai-poc/terraform/`, `terraform init -backend-config=backend.hcl`
- **Platform healthcheck**: `scripts/platform-healthcheck.sh` — 36-check E2E test harness. Run via SSM: `aws ssm send-command --parameters commands=["bash /opt/formpak-ai/platform-healthcheck.sh"]`. Supports `--full` (default) and `--quick` modes.
- **DevOps refactoring plan**: `.planning/devops-refactoring/PLAN.md` — 5 phases, Phases B-E remaining

### GitHub Org Secrets & Variables (Erudite-s)
Shared credentials are stored at **org level** (visibility: ALL), not duplicated per-repo.
**RULE: Tokens MUST be stored as BOTH a secret AND a variable.** Workflows use `secrets.*`, MCP/tools use `vars.*`. Missing either causes silent failures.

- **SonarQube**: `SONAR_TOKEN`, `SONARQUBE_MCP_TOKEN` (both as secret + variable), `SONAR_HOST_URL` (variable only — not sensitive)
- **Component Test DBs**: `COMPONENT_TEST_{DB2,MSSQL,POSTGRES}_{URL,USER,PASSWORD}` (9 secrets)

### Terraform Sensitive Variables (Erudite-s/infrastructure repo)
Terraform variables are provided via **repo-level GitHub Secrets** on `Erudite-s/infrastructure` (not org-level). The GHA deploy workflow passes them as `TF_VAR_*` env vars.

**Source of truth — AWS Secrets Manager (572444560619, eu-west-2):**
| SM Secret Name | GHA Repo Secret | TF Variable |
|----------------|-----------------|-------------|
| `cloudflare/api-token` | `TF_VAR_CLOUDFLARE_API_TOKEN` | `cloudflare_api_token` |
| `cloudflare/account-id` | `TF_VAR_CLOUDFLARE_ACCOUNT_ID` | `cloudflare_account_id` |
| `formpak-test-infra/formpak-admin-password` | `TF_VAR_FORMPAK_ADMIN_PASSWORD` | `formpak_admin_password` |
| `formpak-test-infra/formpak-user-password` | `TF_VAR_FORMPAK_USER_PASSWORD` | `formpak_user_password` |
| — (in GHA secret directly) | `TF_VAR_FORMPAK_SSH_AUTHORIZED_KEYS` | `formpak_ssh_authorized_keys` |
| — (in GHA secret directly) | `TF_VAR_SSH_AUTHORIZED_KEYS` | `ssh_authorized_keys` |
| — (in GHA secret directly) | `TF_VAR_CLOUDFLARE_TUNNEL_ID` | (undeclared — stale, to be removed) |
| — (in GHA secret directly) | `TF_VAR_CODEBUILD_GITHUB_TOKEN` | `codebuild_github_token` |
| — | `AWS_OIDC_ROLE_ARN` | (used by `configure-aws-credentials` action) |

**For local `terraform plan/apply`:** Fetch from SM via CLI:
```bash
export TF_VAR_cloudflare_api_token=$(aws secretsmanager get-secret-value --secret-id cloudflare/api-token --region eu-west-2 --query SecretString --output text --profile formpak)
export TF_VAR_cloudflare_account_id=$(aws secretsmanager get-secret-value --secret-id cloudflare/account-id --region eu-west-2 --query SecretString --output text --profile formpak)
```

### Design System — Single Source of Truth (ABSOLUTE - STRICT - NEVER VIOLATE)

**`Erudite-s/design-system` is the ONLY source of UI components, tokens, and styling for ALL Erudite-s React applications.**

#### The Principle
Every React UI element — buttons, dialogs, sidebars, inputs, dropdowns, navigation components, typography, spacing, color — MUST come from `@erudite-s/design-system` via npm import. Consumer repos contain ZERO local UI primitives.

#### NEVER DO
- **NEVER create `src/components/ui/` directories** with local copies of design system components
- **NEVER copy shadcn components locally** — the registry builds the npm package, consumers import from npm
- **NEVER customize a design system component in a consumer repo** — if it needs changing, change it in `Erudite-s/design-system`
- **NEVER copy `globals.css` or token files locally** — import from the npm package
- **NEVER add Radix UI primitives directly to a consumer repo** — they belong in the design system
- **NEVER create one-off styled components** that duplicate design system patterns

#### ALWAYS DO
- **Import ALL UI components from `@erudite-s/design-system`** — buttons, inputs, dialogs, dropdowns, sidebar, top-bar, collapsible, everything
- **Import CSS/tokens from `@erudite-s/design-system`** — globals, themes, shadcn semantic mappings
- **If a component or feature is missing**: PR to `Erudite-s/design-system` first → add with tests + stories → publish → then import in the consumer
- **If a component API doesn't fit**: extend it in the design system, not locally
- **Keep consumer repos on latest DS version** via Dependabot + auto-merge

#### Design System Architecture
- **Registry**: `registry.design-system.formpak.dev` — shadcn custom registry (build source for npm)
- **npm package**: `@erudite-s/design-system` on GitHub Packages — React components + CSS + tokens + TypeScript declarations
- **Themes**: Modern (plum, Satoshi/DM Sans) and Classic (ExtJS blue, system fonts) via `data-shell-mode` attribute
- **Infrastructure**: Cloudflare R2 bucket for registry hosting, Terraform-managed

#### Consumer Repos
- `Erudite-s/formpak-react-ui` — React shell (full component consumer)
- `Erudite-s/formpak-ai-poc` — AI chat UI (registry component consumer)
- `Erudite-s/aws-infrastructure-manager` — Go dashboard (CSS-only consumer)

### GitHub Actions Node Version Policy
- **All GitHub Actions workflows MUST use actions targeting Node 24+** — never pin to versions that target Node 20 or earlier
- **NEVER use `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true`** — this is a temporary workaround, not a solution. Use action versions that natively target Node 24.
- **When adding new actions**, verify the version targets Node 24 natively by checking the action's `action.yml` for `using: 'node24'`
- **Pin actions by full commit SHA** with a version comment (e.g., `uses: actions/checkout@<sha> # v6.0.2`)

### JIRA API Access
- **Token in macOS Keychain:** `security find-generic-password -s "formpak-jira-api-token" -a "andrea.candian@formpak-software.com" -w`
- **Base URL:** `https://formpak.atlassian.net`
- **Auth:** Basic auth (`$EMAIL:$TOKEN`)
- WebFetch cannot access JIRA — always use `curl` with the keychain token

### Formpak Java App (`erudite-s/Formpak`)
- **Session reference:** See [`formpak-session-F10-2002.md`](formpak-session-F10-2002.md) for build, deploy, Jasper report architecture, test server details, and lessons learned
- **Build:** `cd formpak/formpak && ./gradlew war` (Gradle 9.3.1) → `build/libs/formpak-{version}.war`
- **Ant fallback:** `cd formpak/formpak && ant clean war` → `release/4DedicatedServer/formpak_release_{version}.war`
- **Test server (test2):** `ssh Administrator@10.2.3.33` (Windows, PowerShell)
  - Tomcat at `C:\Program Files\Formpak Software\Formpak\Tomcat\` — **NOT** `C:\formpak\`
  - Service: `Formpak` (Stop-Service/Start-Service)
  - Custom designs: `C:\uploads\customDesign\{typeId}\`
- **InlineReportType next available typeId:** 1051 (1050 = CommercialInvoice)
- **Jasper reports:** `.jrxml` MUST be compiled to `.jasper` before deployment (use jshell + JasperReports 6.20.6)
- **Fork for dev work:** `ndrcndn/formpak` (remote: `myfork`)

### Test Infrastructure & Playwright E2E Reference
- **Full reference:** See [`formpak-test-infrastructure.md`](formpak-test-infrastructure.md) for test server inventory, Playwright execution guides, deployment steps, login credentials, and known gotchas
- **Key servers:** test1-win-db2 (`i-04a72462e5019281b`, 10.2.3.30) has Formpak + Edge; runner-1 (`i-06999936805b6b075`, 10.2.3.20) has Docker for Playwright
- **E2E runs:** Docker on runner-1 for Chromium/Firefox/WebKit; native on Windows for Edge
- **Edge on Windows Server:** Cannot launch under SYSTEM account (exitCode=1002). Use bundled Chromium with Edge user-agent by default; set `PLAYWRIGHT_USE_SYSTEM_EDGE=true` for interactive sessions
- **macOS tarballs:** Always `COPYFILE_DISABLE=1` to avoid `._` resource fork files breaking Playwright
- **Playwright baseURL:** Must end with `/` for relative URL resolution to work correctly
- **S3 staging:** `s3://formpak-test-artifacts-eu-west-2/react-shell/` for test bundles and build artifacts

### Formpak Installers & S3 Buckets (Cross-Account)

**Test account (`572444560619`, eu-west-2):**
- `formpak-test-artifacts-eu-west-2/installers/` — DB2 installer (`v11.1_win64_expc.zip`) and Formpak installers by build number
- `formpak-latest-release-test` — latest WAR deployed to test (build 45573 as of 2025-09)
- `formpak-releases-test` — release WARs by build number
- `formpak-releases-archive/Installer/` — archived installers (builds up to 45021, from 2023)

**Dev/prod account (`421700764597`, eu-west-1):**
- `formpak-releases/<build>/` — **authoritative source for latest Formpak releases** (WARs + installers with setup.exe)
- `formpak-artefacts/<build>/` — full build artifacts (WAR, JARs, scripts, dbscript.zip, analysis reports)
- `formpak-latest-release` — latest WAR (symlink-style, e.g. build 46530 as of 2026-03-30)
- `formpak-installshield` — InstallShield build tools (not Formpak installers)
- `software-for-formpak-development/DB2/` — DB2 installers: v10.5, v11.1, v11.5.8

**Cross-account access:** Test account EC2 instances cannot directly access dev account buckets. Use presigned URLs or copy to `formpak-test-artifacts-eu-west-2` first.

**Installer naming:** `setup_<build>_tomcat<ver>_temurin<ver>.exe` (e.g. `setup_46520_tomcat9.0.102_temurin17.0.14.exe`)

### Infrastructure Repo (`erudite-s/infra`) Specific
- Docker images (allure3, tempo, otel-collector) are built and pushed to ECR via GitHub Actions
- Service configs in `infrastructure/config/` are synced to S3 via `scripts/sync-configs.sh` in the pipeline
- EC2 pulls configs from S3 and images from ECR at boot (user-data.sh) and on deploy (pipeline trigger)
- Terraform manages all AWS resources — EC2, EBS, RDS, S3, IAM, security groups, CloudWatch
- If a pipeline step is missing, **add it to `.github/workflows/`** — do not do it manually

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Erudite-s) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
