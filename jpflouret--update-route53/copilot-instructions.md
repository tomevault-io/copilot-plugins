## update-route53

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`update-route53` is a Go application that periodically checks the host's external IP address and updates an AWS Route 53 DNS record when the IP changes. It is designed for homelab environments with dynamic IPs. It ships as a Docker image and a Helm chart for Kubernetes deployment.

## Common Commands

```bash
# Build
go build -o update-route53 .

# Run tests
go test ./...

# Format code
go fmt ./...

# Vet code
go vet ./...

# Build Docker image
docker build -t update-route53 .

# Run locally (requires AWS credentials and env vars)
DNS_NAME=home.example.com HOSTED_ZONE_ID=ZXXXXXXXXX ./update-route53 -console

# Lint Helm chart
helm lint charts/update-route53/

# Template Helm chart (dry-run)
helm template update-route53 charts/update-route53/ --set dnsName=home.example.com --set hostedZoneId=ZXXXXXXXXX
```

## Architecture

All application logic lives in `main.go` (single file, ~270 lines):

1. **Configuration** — Reads CLI flags (`-console`, `-port`) and required environment variables (`DNS_NAME`, `HOSTED_ZONE_ID`, `DNS_TTL`, `CHECK_IP`, `SLEEP_PERIOD`). AWS credentials come from the SDK's default credential chain (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`).

2. **HTTP server** — Runs on port 8080 (configurable) serving `/healthz` (health probe) and `/metrics` (Prometheus). The server runs in a goroutine alongside the main loop.

3. **Main loop** — Polls `checkip.amazonaws.com` (or custom URL) for the current external IP, compares it to the existing Route 53 record, and issues an upsert if the IP or TTL has changed. After an update, it polls Route 53 until the change reaches `INSYNC` status. Tracks cumulative update duration as the Prometheus counter `update_route53_duration_total`.

## Helm Chart

The Helm chart is at `charts/update-route53/`. Key templating notes:

- `checkIPURL` is the canonical values key for the check-IP URL. The old misspelling `chechIPURL` is deprecated but still supported as a fallback (see `configmap.yaml` and `NOTES.txt` deprecation warning). Remove `chechIPURL` in a future major release.
- AWS credentials can be supplied either through an existing secret (`secret.existingSecret`) or by letting the chart create one (`secret.create: true`). Alternatively, a Kubernetes service account with IRSA/Pod Identity can be used.
- Chart version and app version are in `Chart.yaml` and must be bumped manually on changes.
- The Helm release CI workflow (`helm-release.yml`) uses `chart-releaser` and publishes to `https://jpflouret.github.io/update-route53/`.

## Commit Guidelines

- Make one commit per distinct change (e.g. separate commits for Go, Dockerfile, and Helm changes)
- Keep commit messages short
- Do not add co-authored-by tags

## CI/CD

Two GitHub Actions workflows:
- **`docker-image.yml`** — Builds and pushes multi-platform (`linux/amd64`, `linux/arm64`) images to Docker Hub (`jpflouret/update-route53`) and GHCR on pushes to `main` and semver tags.
- **`helm-release.yml`** — Runs `chart-releaser` on pushes to `main` to publish updated Helm chart releases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpflouret) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
