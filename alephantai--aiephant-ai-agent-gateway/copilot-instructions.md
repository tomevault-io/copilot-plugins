## infrastructure-guide

> This directory contains all infrastructure-as-code and deployment scripts for the LLM Proxy project.

# Infrastructure and Deployment

This directory contains all infrastructure-as-code and deployment scripts for the LLM Proxy project.

## Docker Compose

- [infrastructure/compose.yaml](mdc:infrastructure/compose.yaml): Defines services like the proxy, database, Redis, and observability tooling.

## Deployment Script

- [infrastructure/deploy.sh](mdc:infrastructure/deploy.sh): Bash script to deploy infrastructure and update services.

## Monitoring and Observability

- [infrastructure/prometheus/](mdc:infrastructure/prometheus): Prometheus configuration files.
- [infrastructure/grafana/](mdc:infrastructure/grafana): Grafana dashboards and data source definitions.
- [infrastructure/loki/](mdc:infrastructure/loki): Loki log aggregation setup.
- [infrastructure/tempo/](mdc:infrastructure/tempo): Tempo tracing configuration.
- [infrastructure/opentelemetry-collector/](mdc:infrastructure/opentelemetry-collector): OpenTelemetry Collector pipelines.

## Certificates

- [infrastructure/self-signed-certs/](mdc:infrastructure/self-signed-certs): Self-signed TLS certificates for local development.

---
> Source: [AlephantAI/AIephant-AI-Agent-Gateway](https://github.com/AlephantAI/AIephant-AI-Agent-Gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
