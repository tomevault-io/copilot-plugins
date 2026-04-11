## nestjs-blog

> * **Name**: NestJS Blog (Refactored from Jekyll)

# Project Context: NestJS Blog (Fullstack Refactoring)

## 1. Project Identity
* **Name**: NestJS Blog (Refactored from Jekyll)
* **Goal**: A production-grade, high-performance, containerized full-stack blogging platform.
* **Design Philosophy**: "Build to Learn" & "Data-Driven Architecture".
* **Frontend Style**: "Jable-Style" Immersive Grid Layout (High density, card-based).

## 2. Tech Stack Matrix

| Layer | Technology | Key Details |
| :--- | :--- | :--- |
| **Backend** | **NestJS** (Node.js) | TypeScript, Modular Architecture, Swagger API. |
| **Database** | **PostgreSQL** 16 | JSONB for flexibility, strictly typed via TypeORM. |
| **Cache** | **Redis** | Session storage, API response caching (3ms hit rate). |
| **Frontend** | **Vue 3** + Naive UI | TypeScript, Composition API. |
| **Gateway** | **Nginx** | Reverse Proxy, SSL termination, HTTP/2. |
| **DevOps** | **Docker Compose** | Multi-container orchestration, Layer Caching optimization. |

## 3. Architecture & Infrastructure

### 3.1 Network Topology
* **Entrypoint**: Nginx (Host 80/443) -> Proxies to `blog-backend:3721`.
* **Security (Hidden Ports)**:
    * PostgreSQL (5432) and Redis (6379) are **NOT** exposed to the host network.
    * They communicate strictly within the `blog-network` Docker bridge.
* **Data Migration Strategy**:
    * Uses **SSH Tunneling** via `migration-cli` to access hidden DB ports securely during development/data import.

### 3.2 Deployment Strategy (Hybrid Cloud/Home)
* **Production**: Cloud Server (Ubuntu LTS, AMD64).
* **Build Nodes**:
    * **Primary**: Mac M4 (ARM64) - High IOPS, ultra-fast dev builds.
    * **Secondary**: PVE/i7/i5 (AMD64) - CI Runners.
    * **Edge Test**: Raspberry Pi 5 (ARM64) - IoT/Edge capability test.
* **Cross-Platform Build**:
    * Uses `docker buildx` with `QEMU` to build `linux/amd64` images on `linux/arm64` (Mac) hardware.
    * Command: `DOCKER_DEFAULT_PLATFORM=linux/amd64 docker-compose build`

## 4. Key Engineering Decisions (The "Why")

### 4.1 Docker Optimization (Layer Caching)
* **Rule**: `package.json` and `npm ci` must precede source code `COPY`.
* **Result**: Rebuilds on code changes take < 2s (vs 40s+), leveraging Docker cache for dependencies.

### 4.2 Handling "The Dist Path" Issue
* **Problem**: TypeScript compiler flattens `dist/src/main` to `dist/main` when `.dockerignore` excludes non-src files.
* **Fix**: Dockerfile `CMD` is explicitly set to `["node", "dist/main"]`.

### 4.3 Migration CLI Profile
* **Technique**: The `migration-cli` service in `docker-compose.yml` is tagged with `profiles: ["tools"]`.
* **Benefit**: It does not start automatically with `docker-compose up`, saving resources. Invoked only when needed via `docker-compose run`.

## 5. Performance Benchmarks (Baseline)

### 5.1 Build Time (Clean Install)
* **Mac M4 (NVMe)**: ~4.4s (CPU bound).
* **Intel i7-7700K**: ~4.8s.
* **Raspberry Pi 5 (SD Card)**: ~17m (IOPS bound).
    * *Lesson*: High IOPS (NVMe) is critical for CI/CD `node_modules` installation.

### 5.2 Runtime Resources (Docker Stats)
* **Backend Memory**: ~53MB (Idle).
* **Nginx Memory**: ~4MB.
* **Redis Latency**: < 1ms (Internal Docker Network).

## 6. Future Roadmap
* **IaC**: Implement **Ansible** playbooks to manage Host Nginx and Docker installations (replace manual setup).
* **Observability**: Integrate **Prometheus + Grafana** for real-time monitoring.
* **DB Optimization**: Explore ShardingSphere (Proof of Concept).

## 7. User Persona (For AI Assistance)
* **Role**: Senior Backend/Database Engineer candidate.
* **Focus**: High availability, Performance tuning (IOPS, Latency), Clean Architecture, DevOps automation.
* **Preferences**:
    * Prefer **Self-Hosted** solutions over paid Cloud SaaS.
    * Prefer **Hardcore CLI tools** (stress-ng, fio, btop) over GUI wrappers.
    * Value **Data-driven decisions** (show numbers, not just adjectives).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gdouzwt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gdouzwt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
