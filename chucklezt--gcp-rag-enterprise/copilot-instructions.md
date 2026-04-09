## gcp-rag-enterprise

> This file is read automatically by Claude Code at the start of every session.

# CLAUDE.md — Project Instructions for Claude Code

This file is read automatically by Claude Code at the start of every session.
It contains project-specific context, conventions, and constraints.

---

## Project Overview

**enterprise-rag-gcp** is a production-grade Retrieval-Augmented Generation system
on Google Cloud Platform, aligned to the GCP Well-Architected Framework.

**Owner:** Chuck Tsocanos (https://chucktsocanos.com)
**Purpose:** Portfolio artifact + GCP Professional Cloud Architect cert prep

---

## GCP Configuration

```
Project ID:     rag-demo-491202
Project Number: 204300710565
Region:         us-central1
Zone:           us-central1-a
Bucket name:    rag-demo-491202-rag-documents
AR repo:        us-central1-docker.pkg.dev/rag-demo-491202/rag-docker
```

Always use `us-central1` unless there is a specific reason to deviate.
Always use the project ID from above — never hardcode a different project.

### Live Cloud Run URLs (stable project-number format)
```
Chunker:   https://rag-chunker-204300710565.us-central1.run.app
Query API: https://rag-query-api-204300710565.us-central1.run.app
```

### Vector Search IDs (stored in Secret Manager)
```
Index ID:          4418411665473667072
Endpoint ID:       5195370562125299712
Deployed Index ID: rag_embeddings_deployed
```

---

## Architecture Decisions (already made — do not re-litigate)

- **Dedicated service accounts** — one per Cloud Run service + one for Cloud Build, never the default compute SA
- **CMEK on Cloud Storage** — via Cloud KMS, key ring in us-central1
- **Secret Manager** — all credentials stored there, never in env vars or code
- **Vertex AI Vector Search** — not Pinecone, not ChromaDB, not FAISS in production
- **VPC-peered Vector Search endpoint** — private gRPC via `match()`, not public `find_neighbors()`
- **Private Service Access** — VPC peering to servicenetworking.googleapis.com for Vector Search
- **Gemini 2.5 Flash** — not Pro, not 1.5 — Flash is the cost/quality target
- **text-embedding-004** — 768 dimensions, 500-token chunks, 50-token overlap, batch size 20
- **Pub/Sub push subscription** — triggers Cloud Run chunker on GCS finalize event (lives in cloud-run module)
- **Cloud Build** — CI/CD via triggers, not GitHub Actions — keeps everything GCP-native
- **Terraform** — all infrastructure as code, modular structure under terraform/modules/
- **Cloud Run URLs** — use stable `https://SERVICE-PROJECT_NUMBER.REGION.run.app` format, not hash-based URLs
- **Firestore (rag-chunks database)** — stores chunk text keyed by Vector Search datapoint ID; chunker writes on ingest, query-api reads after match()
- **Vector Search index lifecycle** — `prevent_destroy = true` on `google_vertex_ai_index.rag_index`; never destroy the index, it contains all embeddings and takes hours to rebuild
- **Cloud Build machine type** — E2_MEDIUM (covered by 120 free build-minutes/day), not E2_HIGHCPU_8

### Temporary dev/demo exceptions (remove before production)
- `rag-query-api` has `allUsers` invoker binding — replace with IAP
- `rag-query-api` has `INGRESS_TRAFFIC_ALL` — restrict after IAP is configured
- `rag-chunker` has `INGRESS_TRAFFIC_ALL` — required for Pub/Sub push delivery (auth via OIDC)
- CORS on query API allows all origins — restrict to frontend URL

---

## Naming Conventions

```
Service accounts:   [service]-sa          e.g. chunker-sa, query-api-sa, cloudbuild-sa
Cloud Run services: rag-[service]          e.g. rag-chunker, rag-query-api
GCS buckets:        [project-id]-rag-[purpose]
Pub/Sub topics:     rag-[event]            e.g. rag-ingest-trigger
Secret names:       rag-[resource]-[field] e.g. rag-vector-search-index-id
KMS key rings:      rag-keyring
KMS keys:           rag-storage-key
AR repository:      rag-docker
Firestore DB:       rag-chunks
Firestore coll:     chunks (doc ID = datapoint_id)
Terraform modules:  snake_case
Python files:       snake_case
TS/JS files:        camelCase or kebab-case (Next.js convention)
```

---

## Code Style

### Python (Cloud Run services)
- Python 3.11+
- FastAPI for both services (chunker handles Pub/Sub POST, query-api handles /query)
- LangChain text splitters for chunking (RecursiveCharacterTextSplitter with tiktoken cl100k_base)
- Pydantic v2 for data models
- `structlog` for structured JSON logging (JSONRenderer, no print())
- Type hints on all functions
- Docstrings on all public functions and classes
- Exception tracebacks via `traceback.format_exc()` in log fields (not structlog's exc_info)
- gunicorn + uvicorn workers in production containers

### TypeScript / Next.js (frontend)
- Next.js 14.2.35 App Router (standalone output for containerized deployment)
- TypeScript strict mode
- Tailwind CSS for styling — match chucktsocanos.com color palette:
  - Background: #0a0e1a
  - Panel: #111827
  - Blue accent: #2E5FA3 / #60a5fa
  - Text: #e2e8f0
  - Muted: #64748b
- No external UI component libraries — keep it lean
- SSE (Server-Sent Events) for streaming Gemini responses
- sse-starlette on the backend, custom async generator SSE client on the frontend

### Terraform
- Terraform 1.5+
- Use modules for every logical group of resources
- All resources tagged: `project = "enterprise-rag-gcp"`, `owner = "chuck-tsocanos"`, `env = var.environment`
- Variables in variables.tf, outputs in outputs.tf
- No hardcoded values in resource blocks — always use variables or locals
- `terraform fmt` before every commit

---

## Security Rules (non-negotiable)

1. **Never commit secrets** — use Secret Manager references in all configs
2. **Never use the default compute service account** — always create dedicated SAs
3. **Never open firewall rules broader than necessary**
4. **CMEK must be applied** to any new Cloud Storage bucket
5. **Container images must come from Artifact Registry** — never Docker Hub in production

---

## Terraform Module Structure

```
terraform/
├── main.tf               # Root module — calls all child modules
├── variables.tf          # Input variables
├── outputs.tf            # Output values
├── providers.tf          # Google provider config
├── terraform.tfvars.example  # Template — never commit actual .tfvars
└── modules/
    ├── networking/       # VPC, subnets, Cloud NAT, VPC connector, Private Service Access (VPC peering)
    ├── storage/          # GCS bucket + KMS key ring/key + Pub/Sub topic + GCS notification
    ├── vector-search/    # Vertex AI index + index endpoint + deployed index
    ├── cloud-run/        # Both Cloud Run services + Pub/Sub push subscription + IAM bindings
    │                     # (includes chunker-sa + Pub/Sub SA → rag-chunker run.invoker bindings)
    └── security/         # Service accounts (chunker-sa, query-api-sa, cloudbuild-sa),
                          # IAM bindings, Secret Manager, Artifact Registry, Budget,
                          # Firestore API + rag-chunks database
```

### Module dependency graph
```
networking ──→ vector-search (depends_on for VPC peering)
             ↘ cloud-run (vpc_connector_id)
storage ────→ cloud-run (bucket_name, pubsub_topic_name)
           ↘ security (bucket_name, kms_crypto_key_id)
security ──→ cloud-run (SA emails, secret IDs)
```

---

## Service Architecture

### Ingestion flow
```
GCS upload → Pub/Sub (rag-ingest-trigger) → push subscription → Cloud Run rag-chunker (POST /)
  → Download from GCS
  → Content-type dispatch (text, PDF, EPUB, DOCX, PPTX)
  → LangChain text splitter (500 tok / 50 overlap, tiktoken cl100k_base)
  → Vertex AI text-embedding-004 (768 dims, batch size 20)
  → Firestore write (collection: chunks, doc ID = datapoint_id, fields: text, raw_id, object_name + metadata)
  → Vertex AI Vector Search index upsert (streaming update, deterministic SHA-256 IDs)
```

#### Supported document formats
| Format | MIME type | Chunking strategy | Metadata restricts |
|--------|-----------|-------------------|--------------------|
| Text/Markdown/CSV/JSON | text/* | Flat chunking | source |
| PDF | application/pdf | Page-by-page | source, page_number |
| EPUB | application/epub+zip | Chapter-aware | source, book_title, chapter_title, chapter_index, source_file |
| DOCX | application/vnd.openxmlformats-officedocument.wordprocessingml.document | Heading-sectioned | source, chapter_title, section_index |
| PPTX | application/vnd.openxmlformats-officedocument.presentationml.presentation | Slide-by-slide | source, slide_number, slide_title |

### Query flow
```
Next.js UI → Cloud Run rag-query-api (POST /query)
  → text-embedding-004 (embed question, RETRIEVAL_QUERY task type)
  → Vector Search private endpoint (match() via VPC-peered gRPC, top-5 ANN)
  → Firestore read (fetch chunk text + metadata by datapoint ID, parallel via ThreadPoolExecutor)
  → Gemini 2.5 Flash (streaming generation with RAG system instruction, cites book/chapter)
  → SSE stream (event: token, event: metadata, event: done) → UI renders tokens live
```

### SSE event format
```
event: token
data: <streamed text>

event: metadata
data: {"query_id":"...","chunk_count":5,"embed_latency_ms":45,"retrieve_latency_ms":87,"generate_latency_ms":210,"total_latency_ms":342,"input_tokens":1487,"output_tokens":312}

event: done
data:
```

---

## Observability Standards

Every Cloud Run request must emit a structured JSON log with these fields:
```json
{
  "severity": "INFO",
  "service": "rag-query-api",
  "query_id": "...",
  "latency_ms": 342,
  "embed_latency_ms": 45,
  "retrieve_latency_ms": 87,
  "generate_latency_ms": 210,
  "chunk_count": 5,
  "input_tokens": 1487,
  "output_tokens": 312
}
```

Chunker logs include: `service`, `bucket`, `object_name`, `content_type`, `chunk_count`, `embed_latency_ms`, `upsert_latency_ms`, `total_latency_ms`, plus `firestore_chunks_stored` event. Errors include `exception` field with full traceback.

---

## CI/CD

### Root cloudbuild.yaml (trigger-based, uses $PROJECT_ID and $COMMIT_SHA)
- Builds all three images in parallel (rag-chunker, rag-query-api, rag-frontend)
- Tags with $COMMIT_SHA + latest
- Deploys to Cloud Run sequentially: chunker → query-api → frontend
- Uses `cloudbuild-sa` service account with least-privilege roles
- Machine type: E2_MEDIUM (free tier), logging: CLOUD_LOGGING_ONLY

### Frontend cloudbuild.yaml (manual submit fallback)
- Hardcoded project ID (rag-demo-491202) — $PROJECT_ID doesn't resolve in `gcloud builds submit`
- Passes NEXT_PUBLIC_QUERY_API_URL as Docker build arg

---

## Do Not

- Do not create resources outside `us-central1` without explicit instruction
- Do not use `google_project_iam_member` with `roles/editor` or `roles/owner`
- Do not use `local-exec` or `remote-exec` provisioners in Terraform
- Do not store Terraform state locally — use GCS backend
- Do not use `latest` as a container image tag in production resources
- Do not destroy `google_vertex_ai_index.rag_index` — it has `prevent_destroy = true` and contains all vector embeddings
- Do not use `terraform destroy` targeting vector-search resources — use gcloud commands + `terraform state rm` (see rag-cost-control.sh)
- Do not add `--no-verify` to git commands
- Do not skip `terraform plan` before `terraform apply`
- Do not use $PROJECT_ID in cloudbuild.yaml for manual `gcloud builds submit` — it only resolves in trigger-based builds

---

## Helpful Commands

```bash
# Authenticate gcloud (quota project is set in providers.tf via user_project_override)
gcloud auth login
gcloud auth application-default login

# Set project
gcloud config set project rag-demo-491202

# View Cloud Run logs
gcloud logging read "resource.type=cloud_run_revision" --limit=50 --format=json

# Tail logs live
gcloud beta run services logs tail rag-query-api --region=us-central1

# Terraform workflow
cd terraform
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out=tfplan
terraform apply tfplan

# Build and push containers manually
gcloud builds submit --tag us-central1-docker.pkg.dev/rag-demo-491202/rag-docker/rag-chunker:v1 services/chunker/
gcloud builds submit --tag us-central1-docker.pkg.dev/rag-demo-491202/rag-docker/rag-query-api:v1 services/query-api/
gcloud builds submit --config=cloudbuild.yaml --project=rag-demo-491202 frontend/

# Frontend local dev
cd frontend && npm install && npm run dev

# Cost control (teardown/restore billable resources)
./rag-cost-control.sh                  # status check
./rag-cost-control.sh --teardown       # stop VS endpoint + VPC connector (~$73/mo savings)
./rag-cost-control.sh --restore        # bring back VS endpoint + VPC connector (~30-40 min)
./rag-cost-control.sh --full-teardown  # stop everything except data (~$79/mo savings)
./rag-cost-control.sh --full-restore   # rebuild entire stack (~45-60 min)
./rag-cost-control.sh --stop-services  # delete Cloud Run services (no endpoints accessible)
./rag-cost-control.sh --deep-teardown  # destroy all infra, keep VS index + GCS bucket
./rag-cost-control.sh --bare-project   # destroy ALL resources, leave empty GCP project (IRREVERSIBLE)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chucklezt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chucklezt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
