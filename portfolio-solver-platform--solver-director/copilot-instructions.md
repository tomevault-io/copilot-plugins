## solver-director

> This is a Flask-based microservice called "solver-director" designed to manage different solver-controllers. It's containerized with Docker and deployable to Kubernetes using Helm charts with Skaffold for development workflow automation.

# Claude Code Documentation - solver-director

## Project Overview
This is a Flask-based microservice called "solver-director" designed to manage different solver-controllers. It's containerized with Docker and deployable to Kubernetes using Helm charts with Skaffold for development workflow automation.

## Repository Structure
general overview
```
.
├── src/
│   └── app.py              # Main Flask application
├── helm/                   # Kubernetes deployment configurations
│   ├── Chart.yaml         # Helm chart metadata
│   ├── values.yaml        # Default configuration values
│   ├── values-prod.yaml   # Production-specific values
│   └── templates/         # Kubernetes resource templates
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── hpa.yaml       # Horizontal Pod Autoscaler
│       ├── pdb.yaml       # Pod Disruption Budget
│       └── _helpers.tpl   # Helm template helpers
├── Dockerfile             # Container image definition
├── requirements.txt       # Python dependencies
├── skaffold.yaml         # Development workflow configuration
└── README.md             # Basic project description
```

## Key Components

### Container Configuration (`Dockerfile`)
- Base: `python:3.13-slim`
- Security: Non-root user (uid 10001), optimized for production
- Exposed port: 8080
- Python environment variables for optimization

### Kubernetes Deployment (`helm/`)
- **Chart**: solver-director v0.1.0
- **Security**: Comprehensive security context with non-root, read-only filesystem, dropped capabilities
- **Scaling**: HPA with 1-10 replicas based on 70% CPU utilization
- **Resources**: 200m-1000m CPU, 256Mi-2Gi memory
- **Probes**: Liveness (30s delay) and readiness (5s delay) on `/` endpoint
- **Service**: ClusterIP on port 8080
- **PDB**: Max 1 unavailable pod
- **Rolling updates**: 0 unavailable, 1 max surge

## Development Workflow

### Available Commands
```bash
# Development (local Kubernetes)
skaffold dev                    # Build, deploy, and watch for changes

# Staging
skaffold run -p staging        # Deploy to staging environment

# Production
skaffold run -p prod           # Deploy to production environment

```

### Skaffold Profiles
- **dev**: Local development with port forwarding (localhost:8080), no image push
- **staging**: Staging deployment with production values, local build
- **prod**: Production deployment with image push enabled

## Environment Requirements
- Docker
- Kubernetes cluster (local or remote)
- Skaffold (`/usr/local/bin/skaffold`)
- Helm (`/usr/local/bin/helm`)
- Python 3.13+ (for local development)


## Notes for Claude
- This is a microservice architecture component
- Focus on Flask web development patterns
- Kubernetes-native deployment approach
- Security-first container and deployment configuration
- Development workflow optimized with Skaffold for rapid iteration% 

---
> Source: [Portfolio-Solver-Platform/solver-director](https://github.com/Portfolio-Solver-Platform/solver-director) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
