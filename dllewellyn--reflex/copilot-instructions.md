## reflex

> Auto-generated from all feature plans. Last updated: 2025-12-09

# obsidian-lunar Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-12-09

## Active Technologies
- Go 1.23+ + Apache Kafka client (Go), Google Cloud Storage client (Go), Firebase SDK (Go - specifically for Firestore or Realtime Database), `wire` for dependency injection. (001-gcs-read-tracker)
- Firebase Firestore (001-gcs-read-tracker)
- Go 1.23+ + `cloud.google.com/go/storage`, `github.com/segmentio/kafka-go`, `cloud.google.com/go/vertexai/genai` (003-llm-judge-batch)
- Google Cloud Storage (Source & Sink), Kafka (Alerts Sink) (003-llm-judge-batch)
- Go 1.23+ + `cloud.google.com/go/aiplatform/apiv1` (JobClient), `google.golang.org/protobuf` (003-llm-judge-batch)
- Go 1.24 (001-ingest-datasets)
- Pinecone (Vector Database) (001-ingest-datasets)
- Go 1.23+ + `github.com/oapi-codegen/runtime` (OpenAPI), `github.com/segmentio/kafka-go` (Kafka) (007-analyze-to-topic)
- Kafka (Topic: `analysis-requests`) (007-analyze-to-topic)

- Go 1.23+ (001-gcp-kafka-ingest)

## Project Structure

```text
src/
tests/
```

## Commands

# Add commands for Go 1.23+

## Code Style

Go 1.23+: Follow standard conventions

## Recent Changes
- 007-analyze-to-topic: Added Go 1.23+ + `github.com/oapi-codegen/runtime` (OpenAPI), `github.com/segmentio/kafka-go` (Kafka)
- 005-extract-prompt-injection: Added [if applicable, e.g., PostgreSQL, CoreData, files or N/A]
- 001-ingest-datasets: Added Go 1.24



<!-- MANUAL ADDITIONS START -->
# Code generation 

A number of code-generation tools are available in the makefile - ensure that you check the makefile and run generate as part of / before a build for schemas and openapi -> code generation. 

When developing, you should also follow this pattern.

# Terraform

Infrastructure is managed using Terraform. It is stored in the `terraform` directory.

Ensure that as part of any plans and tasks there is a view of the required terraform infrastructure changes.

# Make tools 

* make tools command installs all the required tools like oapi-codegen

# Deployment

* Builds use cloudbuild.yaml and are triggered to automatically build and deploy when code is pushed to the repository main branch 

# Linting 

* Ensure that you run `make lint-go` as part of your development process and resolve the issues

<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dllewellyn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dllewellyn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
