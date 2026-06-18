## overton

> PSU Research Impact Dashboard — combines data from Penn State RMD (Researcher Metadata Database), OpenAlex, and Overton to track how Penn State research is cited in policy documents. Streamlit dashboard deployed on ECS Fargate, backed by a 6-stage data pipeline orchestrated by Step Functions with Lambda fan-out, writing to RDS PostgreSQL + S3.

# CLAUDE.md

## Project Overview

PSU Research Impact Dashboard — combines data from Penn State RMD (Researcher Metadata Database), OpenAlex, and Overton to track how Penn State research is cited in policy documents. Streamlit dashboard deployed on ECS Fargate, backed by a 6-stage data pipeline orchestrated by Step Functions with Lambda fan-out, writing to RDS PostgreSQL + S3.

## Architecture

- **Database**: RDS PostgreSQL (db.t3.micro) with JSONB columns — `overton-postgres.c86jss96xcpx.us-east-1.rds.amazonaws.com`
- **Storage**: S3 bucket `overton-datalake-700032885189` for raw API response archival, PDF documents, and Lambda deployment packages
- **Dashboard**: Streamlit on ECS Fargate (existing `overton-dashboard` stack)
- **Pipeline**: 6-stage ETL via Lambda functions, with fan-out parallelism for stages 3-5
- **Orchestration**: Step Functions with Map states for fan-out + EventBridge for weekly scheduling
- **Region**: us-east-1
- **VPC**: vpc-b653f0cb (default VPC)
  - Public subnets: EC2 dev instance, NAT Gateway, ECS tasks
  - Private subnets: `subnet-05e2b3532a74fd147` (us-east-1a), `subnet-024916f3b1d8b27a7` (us-east-1b) — Lambda functions route through NAT Gateway for internet access while maintaining VPC access to RDS

## Pipeline Stages

```
Stage 1: RMD Cohort         → researchers table  (cohort + grants/profile + RMD DOI list)
Stage 2: OpenAlex Enrichment → researchers (update: works_dois, metrics, flags)
Stage 3: Overton DOI-set    → articles + article_citations + policy_documents stubs
Stage 4: Overton Documents  → policy_documents (enrich stubs)
Stage 5: PDF Download       → S3 (with pre-check to skip already-downloaded)
Stage 6: Export             → flat JSON files for dashboard fallback
```

**Cohort = ~1,011 researchers** = 752 ORCID-mapped (real ORCID PK) + 259 unmapped (`wa:<webaccess_id>` synthetic PK). Built by `_scan_rmd_publications()` walking all 17 HHD orgs.

**Stage 2 author resolution**: real ORCID → `/authors/orcid:X`; synthetic ID → `/authors?search=<name>&filter=ror:PSU` + `pipeline.name_match` matcher. Auto-accept only when PSU `last_known_institution` AND name matches at top tier; everything else → review queue.

**Disambiguation guard** (Stage 2): if `oa_works/rmd_works > 5×` AND `overlap < 10%`, the OpenAlex author record is conflated; flag suspect, drop OA DOIs, write to review file.

**Stage 3 DOI-set flow**: per researcher, union RMD + OpenAlex DOIs → POST `/generate_id_set.php` (newline-separated DOIs, must include `Content-Type: application/x-www-form-urlencoded` header) → GET `/articles.php?dois=<set_id>`. Article responses contain nested `cited_by_documents` for the policy graph.

**Articles table** holds every PSU work, not just policy-cited ones. Stage 3 stubs the OA-only DOIs; only the Overton-tracked subset gets full metadata.

**Run locally**:
```bash
export AWS_DEFAULT_REGION=us-east-1
export DATABASE_SECRET_ARN="arn:aws:secretsmanager:us-east-1:700032885189:secret:overton/rds-credentials-M85fjC"
export S3_BUCKET_NAME="overton-datalake-700032885189"
export OPENALEX_EMAIL="overton-pipeline@psu.edu"
export OVERTON_API_KEY="<from secretsmanager: overton/api-keys/overton>"
export PSU_RESEARCH_API_KEY="<from secretsmanager: overton/api-keys/rmd>"

python -m pipeline.run --rebuild-map               # ~20 min, builds ORCID/DOIs/names maps in S3
python -m pipeline.run --all                       # full pipeline end-to-end
python -m pipeline.run --stage <stage>             # one stage only
python -m pipeline.run --all --max-researchers 50  # smoke test
```

**Cut-over runbook** (when schema changes need fresh tables):
```bash
python -m scripts.drop_pipeline_tables --confirm   # drops 4 tables, keeps pipeline_metadata + S3 PDFs
python -m pipeline.run --rebuild-map
python -m pipeline.run --all
```

## Database Tables

- `researchers` — PK: `orcid` (real ORCID or `wa:<webaccess_id>` synthetic). JSONB `data` with:
  - `openalex.works_count`, `cited_by_count`, `h_index`, `topics`, `affiliations`, **`works_dois`** (full DOI list)
  - `rmd.profile`, `grants`, `presentations_count`, `etds_count`, `org_memberships`, **`dois`** (RMD-known DOI list), **`name`** (real person name from publications scan)
  - `flags.lookup_method` ('orcid' or 'name_search'), `flags.oa_disambiguation_suspect` (bool)
  - `discovered_orcid` (when name search resolved an ORCID we didn't know)
- `articles` — PK: `doi`. Columns: `data` (JSONB), `policy_citation_count` (int), `last_policy_cited_at` (timestamp), `source_set` (JSONB array of `{rmd, openalex, overton}`). Holds every PSU work; non-policy-cited rows are stubs with just the DOI.
- `policy_documents` — PK: `policy_document_id`. Columns: `data` (JSONB w/ Overton metadata), `pdf_url`, `s3_pdf_key`, `download_status` (`pending`|`downloading`|`downloaded`|`skipped_too_large`|`failed`), `dont_show_pdf`. Stub rows with just `{policy_document_id}` get enriched by Stage 4.
- `article_citations` — junction (`doi`, `policy_document_id`). Only contains edges where the article is by a PSU cohort researcher.
- `pipeline_metadata` — run tracking (run_id, stage, status, stats).

## CloudFormation Stacks

| Stack | Template | Status |
|-------|----------|--------|
| `overton-ecr` | ecr-only.yaml | Deployed |
| `overton-dashboard` | ecs-fargate.yaml | Deployed |
| `overton-vpc-endpoints` | vpc-endpoints.yaml | Deployed |
| `overton-s3-datalake` | s3-datalake.yaml | Deployed |
| `overton-rds` | rds-postgres.yaml | Deployed |
| `overton-pipeline-iam` | pipeline-iam.yaml | Deployed |
| `overton-nat-gateway` | nat-gateway.yaml | Deployed |
| `overton-lambda-functions` | lambda-functions.yaml | Deployed (Lambda code is **stale** — pre-refactor) |
| `overton-step-functions` | step-functions.yaml | Deployed |

## Lambda Functions

Lambda code is currently the **pre-refactor pipeline**. The new pipeline runs locally only until the zip is rebuilt and pushed.

| Function | Handler | Stage |
|----------|---------|-------|
| `overton-rmd` | `rmd_handler` | Stage 1: RMD cohort |
| `overton-openalex` | `openalex_handler` | Stage 2: OpenAlex enrichment |
| `overton-chunker` | `chunker_handler` | Splits work for Map state fan-out |
| `overton-articles` | `overton_articles_handler` | Stage 3: Overton articles (chunk worker) |
| `overton-documents` | `overton_documents_handler` | Stage 4: policy doc enrichment (chunk worker) |
| `overton-document-download` | `document_download_handler` | Stage 5: PDF download (chunk worker) |

Lambda deployment package: `s3://overton-datalake-700032885189/lambda-packages/pipeline-lambda.zip`. Rebuild + push:
```bash
bash scripts/build_lambda_package.sh
for fn in overton-openalex overton-rmd overton-chunker overton-articles overton-documents overton-document-download; do
  aws lambda update-function-code --function-name "$fn" --s3-bucket overton-datalake-700032885189 --s3-key lambda-packages/pipeline-lambda.zip
done
```

**EventBridge weekly schedule (`overton-pipeline-weekly`, Sun 06:00 UTC) is currently DISABLED** — keep it disabled until the Lambda zip is rebuilt with the refactored code, otherwise scheduled runs clobber researcher records with the old shape.

## Key Security Groups

- `sg-098cc4d9e34afd357` — overton-dashboard-task-sg (ECS tasks + Lambda functions)
- `sg-0a14c4c496906c471` — overton-dashboard-alb-sg (ALB)
- `sg-003335a00482554ab` — overton-rds-sg (RDS, allows 5432 from task SG)
- `sg-09b766244cf6996ea` — alex-dev-sg (dev instance, has temp RDS access via sgr-058461653ed7a8a36)

## Secrets Manager

- `overton/rds-credentials` — DB username/password/host/port (auto-generated)
- `overton/api-keys/overton` — Overton API key
- `overton/api-keys/rmd` — PSU RMD API key

## Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `DATABASE_SECRET_ARN` | RDS credentials secret ARN | Yes (or DATABASE_URL) |
| `DATABASE_URL` | Direct PostgreSQL connection string (local dev) | Alternative to above |
| `S3_BUCKET_NAME` | S3 data lake bucket | Yes for S3 archival |
| `AWS_DEFAULT_REGION` | AWS region (local only, reserved in Lambda) | Yes for local runs |
| `OPENALEX_EMAIL` | Contact email for OpenAlex API | Yes |
| `OVERTON_API_KEY` | Overton API key | Yes for stages 3-4 |
| `PSU_RESEARCH_API_KEY` | RMD API key | Yes for stage 1 |

## Project Structure

```
pipeline/               — 6-stage ETL pipeline
  lambda_handlers/      — Lambda entry points for each stage + chunker
  db/                   — PostgreSQL + S3 data layer (connection, schema, operations, s3_client)
  run.py                — CLI orchestrator (local runs)
  config.py             — All configuration (auto-detects Lambda via IS_LAMBDA)
  models.py             — TypedDict schemas
  name_match.py         — Conservative name matcher (rapidfuzz, no soundex)
dashboard/              — Streamlit app
  utils/                — data_loader.py (DB with flat file fallback), db_connection.py
  pages/                — 4 dashboard pages
infrastructure/         — CloudFormation templates
scripts/                — Build scripts, drop_pipeline_tables.py, validation utilities
notebooks/              — explore_rmd.ipynb, explore_openalex.ipynb, explore_overton.ipynb, pipeline_walkthrough.ipynb
data/                   — Local data files (gitignored)
  pipeline/review/      — Review queues from Stage 2 (name_match_review.json, oa_disambiguation_suspects.json)
docs/                   — Data source docs, future extensions
archive/                — Old standalone scripts and pre-migration plan
```

## Conventions

- **Idempotent upserts**: pipeline writes use INSERT ... ON CONFLICT. Stage 3 uses `insert_policy_document_stub_if_missing` (DO NOTHING on conflict) for stub rows so it never clobbers Stage 4's enriched data — past bug, see `pipeline/db/operations.py`.
- **Conservative record-linkage**: `pipeline.name_match` requires last-name exact match + first-name exact / spaceless-merge / fuzzy ≥ 90; everything else routes to `data/pipeline/review/name_match_review.json`. No phonetic matching (soundex too noisy for typed names).
- **Disambiguation guard** in Stage 2 must run *before* downstream stages trust the OA DOI list.
- **Dashboard tries PostgreSQL first**, falls back to flat JSON files in `dashboard/data/` if DB unavailable. The flat-file fallback is regenerated by Stage 6 export.
- **Per-researcher policy citation count** is computed by joining `researcher.openalex.works_dois ∪ rmd.dois` to `article_citations`. Pattern in `pipeline.db.operations.get_researcher_policy_citations` and replicated in `dashboard/utils/data_loader.py`.
- Raw API responses archived to `s3://bucket/raw-api-responses/{stage}/{date}/{id}.json`
- PDFs stored at `s3://bucket/policy-documents/{policy_document_id}.pdf`. Stage 4 pre-checks S3 existence (single ListObjectsV2 walk) so Stage 5 skips already-downloaded.
- Lambda deployment package at `s3://bucket/lambda-packages/pipeline-lambda.zip`
- All infrastructure tagged with `Project: overton`. Deploy stacks with `--tags "Project=overton"`
- Lambda functions use `/tmp` for any file writes (detected via `IS_LAMBDA` in config.py)

## Overton API Notes

Several useful filters live undocumented in Swagger but were validated in our notebooks:

- **`/articles.php?r_open_institution_authors=<ROR>__OVSEP__<Name>__OVSEP__<lower>`** — author-at-institution filter; canonical Overton identifier
- **`/articles.php?dois=<DOI or set:N:hash>`** — DOI filter; accepts a single DOI or a set ID from `/generate_id_set.php`
- **`/documents.php?plain_dois_cited=<DOI or set ID>`** — policy docs citing the given DOI(s)
- **`POST /generate_id_set.php`** — body `dois=<newline-separated DOIs>`, returns `{"set": "set:N:hash"}`. Header `Content-Type: application/x-www-form-urlencoded` is **required** or the body is silently mis-parsed.

**Rate limit**: a single API key has a sustained-burst limit on `/generate_id_set.php`. We hit a wall at ~80 sequential POSTs in our first run. Mitigation: 5/10/20/40s exponential backoff on 429 + `OVERTON_DELAY` between researchers (`pipeline/overton_articles_stage.py`).

## Networking Note

Lambda functions run in private subnets with a NAT Gateway for outbound internet access. This is required because VPC-attached Lambdas don't get public IPs, but they need both VPC access (for RDS) and internet access (for external APIs like OpenAlex, Overton, RMD). The NAT Gateway costs ~$32/month.

## IAM Note

The dev instance role `alex-dev-instance-role` currently has `AdministratorAccess` attached for deployment. This should be scoped down to specific policies needed for day-to-day use.

## Future Extensions

See [`docs/future_extensions.md`](docs/future_extensions.md) for planned work: indirect citation traversal (PSU paper → cited-by paper → policy doc), cohort expansion beyond HHD, ORCID API as additional works source, Lambda zip rebuild + EventBridge re-enable, Stage 5 PDF backfill.

---
> Source: [ajw5296/overton](https://github.com/ajw5296/overton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
