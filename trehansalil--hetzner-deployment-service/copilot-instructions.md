## hetzner-deployment-service

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a **Kubernetes GitOps deployment repository** for two applications running on a Hetzner server with k3s. It contains no application source code — only Kubernetes manifests and deployment automation.

## Common Commands

### Cluster Bootstrap (one-time)
```bash
make cluster-init        # Install Traefik (k3s built-in), cert-manager, Let's Encrypt ClusterIssuers
```

### Neonatal Care App
```bash
make deploy-neonatal                     # Apply all manifests (full deploy)
make rollout-neonatal IMAGE_TAG=<sha>    # Rolling update with new image tag
make status-neonatal                     # Show pods, services, ingress status
make logs-neonatal                       # Tail backend logs
make rollback-neonatal                   # Rollback to previous revision
make init-clickhouse                     # One-time database schema initialization
make destroy-neonatal                    # DELETE entire namespace (destructive)
```

### Airline HR Chatbot App
```bash
make deploy-hr                           # Apply all manifests (full deploy, includes RBAC)
make rollout-hr IMAGE_TAG=<sha>          # Rolling update (app + oracle deployments)
make status-hr                           # Show pods, services, ingress status
make logs-hr                             # Tail app logs
make rollback-hr                         # Rollback to previous revision
make ghcr-secret-hr GITHUB_PAT=<token>  # Create docker-registry secret for private GHCR
make k8s-secrets-hr                      # Apply app-secrets from secret.yaml
make ingest-hr                           # Run data ingest pipeline
make shell-hr                            # SSH into app pod
make port-app-hr                         # Port-forward app to :9040
make port-grafana-hr                     # Port-forward Grafana to :3000
make port-prometheus-hr                  # Port-forward Prometheus to :9090
make port-adminer-hr                     # Port-forward Adminer to :8080
make destroy-hr                          # DELETE namespace + ClusterRole/Binding (destructive)
```

### Manual Secret Setup
```bash
# Neonatal Care
cp apps/neonatal-care/secret.yaml.example apps/neonatal-care/secret.yaml
kubectl apply -f apps/neonatal-care/secret.yaml -n neonatal-care

# Airline HR Chatbot
cp apps/airline-hr-chatbot/secret.yaml.example apps/airline-hr-chatbot/secret.yaml
make ghcr-secret-hr GITHUB_PAT=<token>   # Required to pull private GHCR images
make k8s-secrets-hr
```

## Architecture

### Directory Layout
- `cluster/` — cluster-wide bootstrap: namespaces, Traefik HelmChartConfig (HTTP→HTTPS), cert-manager, ACME issuers
- `apps/neonatal-care/` — configmap, secret template, PVCs, deployments, services, ingress, init job
- `apps/airline-hr-chatbot/` — configmap (PostgreSQL init SQL), secret template, PVCs, deployments, StatefulSet, DaemonSet, RBAC, services, ingresses
- `.github/workflows/deploy.yml` — triggered by `repository_dispatch` (from source repos) or manual `workflow_dispatch`

### Neonatal Care — Request Flow
```
Internet → Traefik (TLS termination + HTTP→HTTPS)
        → nginx (static SPA + reverse proxy)
            /api/             → Flask backend :5000 (gunicorn/gevent, 4 workers)
            /minio/           → MinIO API :9000
            /minio-console/   → MinIO console :9001
            /automation/      → n8n :5678
```
Static files are copied from the backend image by an nginx initContainer at startup (emptyDir shared volume). The ConfigMap holds both env vars and `nginx.conf`.

### Neonatal Care — Services (`neonatal-care` namespace)
| Service | Port | Storage |
|---------|------|---------|
| Flask backend | 5000 | — |
| ClickHouse | 8123 (HTTP), 9000 (native) | 10Gi data + 2Gi logs |
| MinIO | 9000 / 9001 | 20Gi |
| Redis | 6379 | none |
| n8n | 5678 | 2Gi |
| nginx | 80 | — |

### Airline HR Chatbot — Services (`hr-chatbot` namespace)
| Service | Port | Storage |
|---------|------|---------|
| PostgreSQL (StatefulSet, pgvector:pg16) | 5432 | 10Gi |
| Oracle Mock Server (FastAPI) | 8001 | — |
| Chainlit app | 9040 (UI), 9091 (metrics) | — |
| Adminer | 8080 | — |
| Prometheus | 9090 | 20Gi |
| Loki | 3100 | 10Gi |
| Grafana | 3000 | 5Gi |
| Promtail (DaemonSet) | — | — |

Chainlit waits for both Postgres and the Oracle Mock Server via init containers. Promtail requires a ClusterRole (get/list/watch pods/nodes/namespaces) to discover log targets; `make destroy-hr` also cleans up the ClusterRoleBinding.

### TLS & Networking
- **Ingress class**: `traefik` (k3s built-in)
- **TLS**: cert-manager v1.14.5 with Let's Encrypt HTTP-01 challenge
- **Issuers**: `letsencrypt-prod` (production) and `letsencrypt-staging` (rate-limit-free testing)
- **Domains**: `neonate-logger.saliltrehan.com`, `airline-hr.saliltrehan.com`, `grafana-hr.saliltrehan.com`, `adminer-hr.saliltrehan.com`

### Storage
Default storage class is k3s `local-path` (single-node HostPath). To use Hetzner block storage on multi-node clusters, uncomment `storageClassName: hcloud-volumes` in PVC files.

### Deployment Pipeline
Source repos (`neonatal-care-repo`, `airline-hr-chatbot`) build images and push to GHCR (`ghcr.io/trehansalil/...`), then fire a `repository_dispatch` event here. The deploy workflow decodes `secrets.KUBECONFIG_B64`, applies manifests, and runs `kubectl set image` for a rolling update. The HR chatbot images are in a private GHCR registry requiring the `ghcr-credentials` pull secret.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trehansalil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
