## devops-lab-02

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SmartSplit** is a full-stack expense splitting application with:
- **Backend**: Spring Boot 3.5.5 (Java 17) REST API with JWT authentication, MySQL database, and Flyway migrations
- **Frontend**: React 19 with TypeScript, TailwindCSS, and React Router
- **Infrastructure**: Kubernetes deployment on Minikube with Docker containerization

## Core Architecture

### Backend Structure (Spring Boot)
Located in `backend/src/main/java/com/smartsplit/smartsplitback/`:
- **controller/**: REST API endpoints
- **service/**: Business logic layer
- **repository/**: JPA repository interfaces
- **model/**: JPA entities and DTOs (in `model/dto/`)
- **security/**: JWT authentication and Spring Security configuration
- **config/**: Application configuration classes
- **Database**: Flyway migrations in `backend/src/main/resources/db/migration/`

The backend uses:
- Spring Security with JWT tokens (configured via `APP_JWT_SECRET` and `APP_JWT_EXPIRATION_SECONDS`)
- MySQL 8 database with JPA/Hibernate
- Testcontainers for integration tests (files ending in `*IT.java`)
- Actuator for health checks and monitoring
- OpenAPI/Swagger UI for API documentation

### Frontend Structure (React)
Located in `frontend/src/`:
- **pages/**: Full page components (Login, SignUp, HomePage, GroupPage, BillDetail, etc.)
- **components/**: Reusable UI components (Navbar, BottomNav, FAB, TransactionCard, etc.)
- **contexts/**: React context providers (AuthContext for authentication state)
- **utils/**: Utility functions and helpers
- **types/**: TypeScript type definitions

The frontend communicates with the backend through a proxy configuration (set in `frontend/package.json` to `http://127.0.0.1:16048`).

### Authentication Flow
- JWT-based authentication managed via AuthContext
- Protected routes use `ProtectedRoute` component
- Public routes (login/signup) use `PublicRoute` component
- Tokens stored and managed client-side

## Development Commands

### Backend (from `backend/` directory)
```bash
# Run unit tests only (skip integration tests)
mvn test -DskipITs=true

# Run integration tests only (files matching *IT.java)
mvn verify -DskipTests

# Run all tests
mvn verify

# Build the application
mvn clean package

# Run locally (requires MySQL running)
mvn spring-boot:run

# Run with specific profile
SPRING_PROFILES_ACTIVE=prod mvn spring-boot:run
```

### Frontend (from `frontend/` directory)
```bash
# Install dependencies
npm install

# Start development server (port 3000)
npm start

# Build for production
npm build

# Run unit tests with Jest
npm test

# Run Cypress E2E tests (headless)
npm run test:e2e

# Open Cypress test runner
npm run cypress:open
```

### E2E Testing (from project root)
```bash
# Run Cypress tests
npx cypress run

# Open Cypress interactive mode
npx cypress open
```

Cypress is configured in `cypress.config.js` with base URL `http://localhost:3003/` (override with `CYPRESS_BASE_URL` env var).

### Docker Compose (Local Development)
```bash
# Start all services (MySQL, backend, frontend)
docker-compose up

# Start in detached mode
docker-compose up -d

# Stop all services
docker-compose down

# Rebuild and start
docker-compose up --build
```

Services exposed:
- Frontend: http://localhost:8080
- Backend API: http://localhost:8081
- MySQL: localhost:8082

### Kubernetes Deployment (Minikube)

**Linux/Mac:**
```bash
# Deploy to Minikube
./deploy-minikube.sh
```

**Windows:**
```powershell
# Deploy to Minikube
.\deploy-minikube.ps1
```

**Manual K8s operations:**
```bash
# Apply configurations
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/mysql/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
kubectl apply -f k8s/ingress.yaml

# Check pod status
kubectl get pods -n smartsplit

# View logs
kubectl logs -n smartsplit <pod-name>

# Port forwarding (scripts available in scripts/)
kubectl port-forward -n smartsplit svc/frontend 3003:80
kubectl port-forward -n smartsplit svc/backend 16048:8081
kubectl port-forward -n smartsplit svc/mysql 8082:3306
```

Port forwarding scripts in `scripts/` directory:
- `start-port-forward.ps1`: Start all port forwards
- `stop-port-forward.ps1`: Stop all port forwards
- `port-forward.ps1`: Main port forwarding script

## Environment Variables

Required environment variables (see `.env` file):
```
MYSQL_ROOT_PASSWORD=<password>
MYSQL_DATABASE=smartsplit-db
APP_JWT_SECRET=<long-secure-secret>
APP_JWT_EXPIRATION_SECONDS=86400
```

For backend tests, set:
```
SPRING_PROFILES_ACTIVE=test
APP_JWT_SECRET=<test-secret>
```

## CI/CD Pipeline

The project uses GitHub Actions (`.github/workflows/ci-cd.yml`) with four stages:

1. **Test**: Runs backend unit tests with Maven and installs frontend dependencies
2. **E2E-Test**: Runs Cypress end-to-end tests with full application stack (MySQL, Backend, Frontend)
3. **Build-and-Push**: Builds Docker images and pushes to Docker Hub with dual tags:
   - `:latest` - Convenience tag for pulling latest image
   - `:${git-sha}` - Immutable tag tied to git commit (used for deployments)
4. **Deploy**: Updates Kubernetes deployments using git SHA tags for immutable, auditable deployments

Pipeline runs on `self-hosted` runner using PowerShell on Windows.

**Required GitHub Secrets:**
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `APP_JWT_SECRET`

### Docker Image Tagging Strategy

**Production (CI/CD)**:
- Images are tagged with both `:latest` and `:${git-sha}` during build
- **Deployments use `:${git-sha}` tags exclusively** for:
  - Immutable deployments (each deployment tied to specific git commit)
  - Easy rollbacks to previous versions
  - Clear audit trail of deployed code
  - Guaranteed consistency across environments

**Local Development (Minikube)**:
- Uses `:latest` tag by default for simplicity
- Can optionally use specific versions for testing:
  ```powershell
  # Windows - deploy with custom tag
  .\deploy-minikube.ps1 -ImageTag "v1.2.3"
  ```
  ```bash
  # Linux/Mac - deploy with custom tag
  IMAGE_TAG=v1.2.3 ./deploy-minikube.sh
  ```

**Image Pull Policy**:
- Both backend and frontend use `imagePullPolicy: Always` to ensure fresh images are pulled
- Prevents stale image issues in Kubernetes

**Checking Deployed Version**:
```bash
# View current image tags in production
kubectl describe deployment backend -n smartsplit | grep Image:
kubectl describe deployment frontend -n smartsplit | grep Image:

# View deployment history
kubectl rollout history deployment/backend -n smartsplit
kubectl rollout history deployment/frontend -n smartsplit
```

**Rolling Back to Previous Version**:
```bash
# Rollback to previous deployment
kubectl rollout undo deployment/backend -n smartsplit
kubectl rollout undo deployment/frontend -n smartsplit

# Rollback to specific revision
kubectl rollout undo deployment/backend -n smartsplit --to-revision=2

# Deploy specific git SHA version
kubectl set image deployment/backend backend=sakanaisreal/smartsplit-backend:<git-sha> -n smartsplit
kubectl set image deployment/frontend frontend=sakanaisreal/smartsplit-frontend:<git-sha> -n smartsplit
```

### E2E Testing Stage Details

The E2E test stage:
- Spins up a temporary MySQL database using Docker (port 3307)
- Builds and starts the backend (port 16048)
- Starts the frontend dev server (port 3000)
- Waits for all services to be healthy using actuator endpoints
- Runs Cypress tests from the **root directory** using `npx cypress run`
- Uploads test artifacts (screenshots on failure, videos always) to GitHub Actions
- Cleans up all processes and Docker containers after completion

**Cypress Configuration:**
- Tests located in: `cypress/e2e/`
- Config file: `cypress.config.js` (root directory)
- Base URL: Configurable via `CYPRESS_BASE_URL` environment variable (default: `http://localhost:3003/`, CI uses `http://localhost:3000`)
- Artifacts saved to: `cypress/screenshots/`, `cypress/videos/`, `cypress/downloads/` (all gitignored)

**Test Coverage:**
The E2E test suite (`cypress/e2e/SmartSplit-E2E.cy.js`) covers:
- User authentication (sign up, sign in, sign out)
- Profile editing
- Group management (CRUD operations)
- Expense management (equal split and manual split)
- Payment processing and verification

## Network Ports

SmartSplit uses different port configurations for local development and Kubernetes deployment. **For complete details, see [docs/PORT_CONFIGURATION.md](docs/PORT_CONFIGURATION.md)**.

### Quick Port Reference

| Port | Environment | Service | Access URL |
|------|-------------|---------|------------|
| **3000** | Local Dev, CI/CD | Frontend Dev Server | http://localhost:3000 |
| **3003** | Minikube | Frontend (Port Forward) | http://localhost:3003 |
| **8080** | Docker Compose | Frontend (nginx) | http://localhost:8080 |
| **8081** | Docker Compose | Backend (Spring Boot) | http://localhost:8081/api |
| **16048** | Minikube, CI/CD | Backend (Port Forward) | http://localhost:16048/api |
| **8082** | Docker, Minikube | MySQL | localhost:8082 |
| **3306** | Internal Only | MySQL (Container) | - |

### Port Diagnostic Tools

Check which ports are in use:

```powershell
# Windows
.\scripts\check-ports.ps1
```

```bash
# Linux/Mac
./scripts/check-ports.sh
```

These scripts show:
- Which SmartSplit ports are currently in use
- What processes are using each port
- Active Docker containers and kubectl port-forwards
- Helpful commands for managing services

### Common Port Issues

**Frontend cannot reach backend:**
- Ensure backend is running on the expected port
- Docker Compose backend: `localhost:8081`
- Minikube backend (port-forward): `localhost:16048`
- Frontend `package.json` proxy should match backend port

**Port already in use:**
- Run diagnostic script to identify conflicting process
- Kill conflicting process or use alternative port
- Docker Compose conflicts: Stop with `docker-compose down`
- Kubernetes port-forward conflicts: Run `.\scripts\stop-port-forward.ps1`

**Known Issue:** `frontend/package.json` proxy points to `localhost:16048`, but Docker Compose backend uses `localhost:8081`. This breaks local frontend development with Docker Compose backend. See [docs/PORT_CONFIGURATION.md](docs/PORT_CONFIGURATION.md#known-issues--mismatches) for details and workarounds.

For detailed port configuration flows, troubleshooting, and environment-specific setups, refer to the comprehensive [Network Port Configuration Guide](docs/PORT_CONFIGURATION.md).

## Key Technical Details

### Backend Testing Strategy
- **Unit tests**: Standard test files in `src/test/java` (run with `mvn test -DskipITs=true`)
- **Integration tests**: Files ending in `*IT.java` using Testcontainers with MySQL (run with `mvn verify`)
- **Code coverage**: JaCoCo plugin generates reports after tests

### API Base Path
The backend API uses `/api` as the base path for all endpoints (configured in Kubernetes ingress and mentioned in recent commits).

### Database Migrations
Flyway handles database schema versioning. Migration files are in `backend/src/main/resources/db/migration/` following the pattern `V<version>__<description>.sql`.

## Branch Strategy

- **main**: Production branch (triggers CI/CD deployment)
- **develop**: Development branch (current working branch)

When creating pull requests, target the `develop` branch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SakanaIsReal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SakanaIsReal)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
