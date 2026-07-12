## enterpriseaicontrolplane

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **EnterpriseAIControlPlane** repository - a production-grade GenAI + Agentic AI application for Enterprise Customer Support. The detailed CODE PLAN is saved in `memory/enterprise-ai-customer-support-plan.md`.

## Architecture (From Code Plan)

### High-Level Layers
1. **Frontend**: React + TypeScript (Chat UI, Role Selector, Metrics Dashboard)
2. **API Layer**: Node.js + Express (REST endpoints, middleware)
3. **AI Gateway**: Enterprise governance (Auth/RBAC, model routing, token limits, guardrails)
4. **Agentic AI Layer**: LangGraph orchestration (Planner → Workers → Critic → Memory)
5. **Data Layer**: PostgreSQL (STM, RBAC), Pinecone (LTM, RAG), Redis (caching)

### Key Modules
- `backend/src/gateway/` - Authentication, RBAC, model routing, audit logging
- `backend/src/memory/` - STM (Postgres) and LTM (Pinecone) implementations
- `backend/src/rag/` - Document ingestion, hybrid search, role-based retrieval
- `backend/src/agents/` - Planner, FAQ/RAG/Summary/Reasoning workers, Critic, Memory agents
- `backend/src/services/` - Groq SDK, CostCalculator, OrchestrationService

## Development Commands

```bash
# Backend development (from backend/)
npm install          # Install dependencies
npm run dev          # Start development server (tsx watch)
npm run build        # Build production (tsc)
npm test             # Run all tests
npm run lint         # Run ESLint

# Frontend development (from frontend/)
npm install          # Install dependencies
npm run dev          # Start Vite dev server
npm run build        # Build for production
npm run lint         # Run ESLint

# Full stack (from root)
docker-compose up    # Start all services
```

## LLM Provider: Groq

This project uses **Groq** as the LLM provider with these models:
- `llama-3.1-8b-instant` - Fast responses (small)
- `llama-3.1-70b-instant` - Standard queries (medium)
- `llama-3.1-70b-versatile` - Complex reasoning (large)

## Vector Database: Pinecone

- **Index**: `mortgageindex`
- **Host**: `https://mortgageindex-96hwyzx.svc.aped-4627-b74a.pinecone.io`
- **Namespaces**: RAG documents, user memories, conversation context

## Database & Services

- **PostgreSQL**: `ai_apps_db_nzf4` on Render (Oregon)
- **Redis**: `redis://red-d836e2t7vvec73938sl0:6379`
- **LangSmith**: Tracing & observability

## Role-Based Access

Supported roles: `support_engineer`, `mortgage_analyst`, `compliance_officer`, `product_owner`

### Environment Variables

Create `backend/.env` from `backend/.env.example`:

```bash
DATABASE_URL=postgresql://...
GROQ_API_KEY=your-groq-key
PINECONE_API_KEY=your-pinecone-key
REDIS_URL=redis://...
JWT_SECRET=your-secret
LANGSMITH_API_KEY=your-key
```

---
> Source: [prakashpujari/EnterpriseAIControlPlane](https://github.com/prakashpujari/EnterpriseAIControlPlane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
