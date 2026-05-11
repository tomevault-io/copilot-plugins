## kubey

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kubey is a Kubernetes cluster monitoring and visualization tool with a React/TypeScript frontend and Go backend that connects to Kubernetes clusters via the Kubernetes API.

## Repository Structure

This is a monorepo with two main components:

- `web/` - React frontend (TypeScript + Vite)
- `api/` - Go backend (Gin framework)

## Development Commands

### Frontend (web/)

Working directory: `C:\Users\Justi\Documents\kubey\web`

- **Dev server**: `npm run dev` (runs on <http://localhost:5173>)
- **Dev server (network)**: `npm run dev+` (accessible at <http://0.0.0.0:5173>)
- **Build**: `npm run build` (TypeScript check + Vite build)
- **Lint**: `npm run lint`
- **Preview production build**: `npm run preview`
- **Preview (network)**: `npm run preview+`

### Backend (api/)

Working directory: `C:\Users\Justi\Documents\kubey\api`

- **Run server**: `go run cmd/api/main.go`
- **Build**: `go build -o kubey-server.exe ./cmd/api/main.go`
- **Install dependencies**: `go mod download`
- **Update dependencies**: `go get -u ./...`
- **Tidy dependencies**: `go mod tidy`

## Architecture

### Frontend Architecture

- **Framework**: React 19 with TypeScript and Vite
- **UI Library**: shadcn/ui components (new-york style) + Tailwind CSS v4
- **Routing**: react-router-dom v7
- **Theming**: next-themes for dark/light mode support
- **State Management**: React hooks (no external state library)
- **Type System**: Strict TypeScript with comprehensive Kubernetes resource types defined in `web/src/types/kube.ts`

Key frontend types hierarchy:

```
KubeCluster (root)
├── ControlPlane (nodes)
├── Nodes (worker nodes)
└── Namespaces
    ├── Deployments
    ├── Services
    └── Pods
        ├── Containers
        └── Volumes
```

### Backend Architecture

- **Framework**: Gin web framework
- **Kubernetes Client**: k8s.io/client-go v0.34.1
- **Configuration**: Environment variables via godotenv (optional .env file)

Backend structure:

```
api/
├── cmd/
│   └── api/main.go                              # Entry point, server setup
├── internal/
│   ├── config/config.go                         # ApiConfig with HTTP timeouts
│   ├── routes/routes.go                         # Route definitions + CORS
│   ├── handlers/
│   │   └── clusters/clusters.go                 # Cluster HTTP handlers
│   ├── services/
│   │   └── kubernetes/kubernetes.go             # K8s client + data fetching
│   └── models/kubernetes.go                     # Go struct definitions
```

### API Endpoints

REST API (port 8080):

- `GET /api/clusters` - List all clusters
- `GET /api/clusters/:id` - Get cluster details
- `GET /api/clusters/:id/nodes` - Get cluster nodes
- `GET /api/clusters/:id/pods` - Get all pods
- `GET /api/clusters/:id/services` - Get all services
- `GET /api/clusters/:id/deployments` - Get all deployments
- `GET /api/clusters/:id/namespaces` - Get namespaces with resources
- `GET /health` - Health check

### Kubernetes Client Initialization

The backend initializes the Kubernetes client in this order:

1. Use explicit KUBECONFIG env var if provided
2. Try in-cluster config (for running inside K8s)
3. Fall back to default kubeconfig (~/.kube/config)

The client is initialized once at startup in `cmd/api/main.go` via `kubernetes.InitClient()`.

### Data Flow

1. Frontend makes HTTP requests to `/api/*` endpoints
2. Backend handlers (in `handlers/clusters` package) call `kubernetes` service functions
3. Services use `k8s.io/client-go` to query Kubernetes API
4. Data is transformed to internal models and returned as JSON

## Environment Configuration

### Backend (.env in api/)

Create `api/.env` (optional, falls back to defaults):

```
ENVIRONMENT=development
HOST=localhost
PORT=8080
LOG_LEVEL=info
KUBECONFIG=  # Leave empty to use default ~/.kube/config
HTTP_READ_TIMEOUT=10  # HTTP read timeout in seconds (default: 10)
HTTP_WRITE_TIMEOUT=10  # HTTP write timeout in seconds (default: 10)
HTTP_IDLE_TIMEOUT=30  # HTTP idle timeout in seconds (default: 30)
```

### Frontend CORS

The backend allows CORS from any localhost or 127.0.0.1 origin on any port (for local development flexibility).

## Code Style Guidelines

### Frontend

See `web/AGENTS.md` for detailed frontend code style:

- Use `@/*` alias for src/ imports
- React imports first, then third-party, then local
- PascalCase for components, camelCase for functions/hooks
- forwardRef components should include displayName
- Use `cn()` helper from `@/lib/utils` for class merging
- Tailwind CSS for styling with CSS variables for theming

### Backend

- Package organization: `internal/` for private packages
- Error handling: Return errors, log at entry points
- Context usage: Use `context.TODO()` for K8s client calls (consider adding proper context propagation)
- Logging: Use standard `log` package
- Thread safety: Use mutexes for shared state (see WebSocket hub)

## Important Notes

- The frontend uses sample data in `web/src/data/sampleClusters.ts` for development
- All Kubernetes timestamps are converted to Go `time.Time` and serialized as ISO strings

---
> Source: [justinbehncodes/kubey](https://github.com/justinbehncodes/kubey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
