## repolens

> This document serves as the primary instructional context for Gemini AI agents interacting with the RepoLens codebase. It defines the architectural mental model, development protocols, and engineering standards of the project.

# GEMINI_INSTRUCTIONAL_CONTEXT // REPOLENS

This document serves as the primary instructional context for Gemini AI agents interacting with the RepoLens codebase. It defines the architectural mental model, development protocols, and engineering standards of the project.

---

## PROJECT_OVERVIEW

RepoLens is a high-performance **Architectural Diagnostic Engine**. It transforms complex software repositories into structured, navigable, and executable knowledge bases.

### CORE_MISSION
Surpass traditional documentation tools by transitioning from a passive code reader to an **Interactive Architectural Operating System**.

### KEY_TECHNOLOGIES
- **Frontend**: Next.js 14 (App Router), ReactFlow (Spatial Mapping), Tailwind CSS.
- **API Gateway**: NestJS, BullMQ (Job Orchestration).
- **Analysis Core**: Python 3.11 (Tree-sitter), transitioning to **Rust (OXC, SCIP)** for performance.
- **Intelligence**: Google Gemini 2.0 Thinking Models.
- **Execution**: Native Docker Sandboxing (on AWS EC2).
- **Data Layer**: Neon (PostgreSQL), Upstash (Redis), Cloudflare (Vectorize).

---

## SYSTEM_ARCHITECTURE

RepoLens uses a distributed analysis pipeline:
1. **Intake**: User submits a GitHub URL via the Next.js frontend.
2. **Orchestration**: The NestJS API validates the request and dispatches a job to BullMQ.
3. **Analysis**: 
   - **Static**: Parses AST and identifies architectural patterns.
   - **System**: Parses CI/CD workflows, Makefiles, and build scripts.
   - **Semantic**: Generates symbolic indexes (SCIP) for cross-file navigation.
4. **Synthesis**: Gemini 2.0 generates structured Wiki chapters and architectural summaries.
5. **Memory**: Embeddings are generated and stored in a vector database for RAG-based queries.
6. **Execution**: Snippets are verified in an isolated Docker sandbox.

---

## ENGINEERING_STANDARDS

The project adheres to strict technical and operational protocols:

### TECHNICAL_PROTOCOL
- **Robustness First**: All components must handle failure modes gracefully (e.g., rate limits, environment constraints).
- **Security**: Native execution must be isolated in sandboxed containers without external network access.
- **Concurrency**: Parallel parsing and asynchronous job handling for high-throughput analysis.
- **Accessibility**: Ensure all interfaces meet WCAG AA standards.

---

## BUILDING_AND_RUNNING

### PREREQUISITES
- Node.js >= 18.17.0
- Python >= 3.11
- Docker & Docker Compose (V2)

### COMMAND_INDEX
| Task | Command |
| :--- | :--- |
| **Initialize** | `pnpm install` |
| **Development** | `pnpm dev` |
| **Build** | `pnpm build` |
| **Database Sync** | `pnpm dlx prisma db push` |
| **Backend Deploy** | `docker compose up -d --build` |
| **Cleanup** | `pnpm clean` |

---

## DEVELOPMENT_CONVENTIONS

- **Monorepo Management**: Powered by **Turborepo**.
- **Data Access**: Use the **Prisma Client** for database operations.
- **AI Integration**: AI-related logic resides in the worker service.
- **Roadmap Alignment**: Consult `BLUEPRINT_PLAN.md` for project milestones.

---

## AGENT_INTERACTION_GUIDELINES

When acting as an agent within this repository:
1. **Maintain Tone**: Be analytical, authoritative, and focused on technical precision.
2. **Verify Stability**: Always run build and lint commands after modifying core logic or shared components.
3. **Security Awareness**: Never expose secrets or introduce insecure execution patterns.

(C) 2026 REPOLENS // ARCHITECTURAL_ARCHIVE_VAULT

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haxllo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/haxllo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
