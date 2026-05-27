## pixel-detective

> This document provides comprehensive guidelines for AI agents working on the Pixel Detective project, which consists of two production-ready applications:

# AI Agent Guidelines - Pixel Detective Project

## 🎯 **Overview**

This document provides comprehensive guidelines for AI agents working on the Pixel Detective project, which consists of two production-ready applications:

1. **Pixel Detective** - AI-powered media search engine
2. **Dev Graph** - Temporal knowledge graph for code evolution

**Key Principle**: This entire codebase was built through AI-assisted development. Every change should maintain this philosophy and code quality standards.

---

## 📋 **Quick Navigation Guide**

This root-level AGENTS.md serves as a **navigation hub**. For detailed development guidelines, consult the appropriate subdirectory documentation:

### **🎨 Pixel Detective (Media Search)**

**When working on:** Media search, image ingestion, CLIP/BLIP models, vector search, UMAP visualization

**Primary Documentation:**
- [`backend/AGENTS.md`](backend/AGENTS.md) - Backend service development (Ingestion, ML Inference, GPU-UMAP)
- [`frontend/AGENTS.md`](frontend/AGENTS.md) - Frontend development (Next.js, React, Chakra UI)
- [`backend/ARCHITECTURE.md`](backend/ARCHITECTURE.md) - Backend system design
- [`frontend/ARCHITECTURE.md`](frontend/ARCHITECTURE.md) - Frontend architecture

**Quick Reference:**
- Backend services: 3 FastAPI microservices (ports 8001, 8002, 8003)
- Frontend: Next.js 14 with App Router (port 3000)
- Database: Qdrant vector database (port 6333)
- AI Models: CLIP (embeddings), BLIP (captions), RAPIDS cuML (UMAP)

### **🗺️ Dev Graph (Code Knowledge Graph)**

**When working on:** Git history ingestion, knowledge graph, Neo4j, code symbol extraction, relationship derivation

**Primary Documentation:**
- [`developer_graph/AGENTS.md`](developer_graph/AGENTS.md) - Dev Graph module development
- [`developer_graph/architecture.md`](developer_graph/architecture.md) - System architecture and data model
- [`developer_graph/routes/AGENTS.md`](developer_graph/routes/AGENTS.md) - API route handlers
- [`dev_graph_audit/AGENTS.md`](dev_graph_audit/AGENTS.md) - Data quality and auditing

**Quick Reference:**
- Backend: FastAPI service (port 8080)
- Frontend: Next.js UI at `tools/dev-graph-ui` (port 3001)
- Database: Neo4j graph database (ports 7687/7474)
- Pipeline: 8-stage unified ingestion system

### **📚 Cross-Cutting Documentation**

**For project-wide concerns:**
- [`README.md`](README.md) - Project overview, quick start, features
- [`Architecture.MD`](Architecture.MD) - Legacy architecture reference
- [`docs/architecture.md`](docs/architecture.md) - Current system architecture for both apps
- [`DEVELOPER_GUIDE.md`](DEVELOPER_GUIDE.md) - Developer onboarding
- [`.cursor/rules/`](.cursor/rules/) - Context-specific coding guidelines

---

## 🏗️ **Repository Structure**

```
pixel-detective/
├── 🎨 Pixel Detective (Media Search)
│   ├── frontend/                          # Next.js UI (port 3000)
│   │   ├── src/app/                      # App router pages
│   │   ├── src/components/               # React components
│   │   ├── AGENTS.md                     # ← Frontend development guide
│   │   └── ARCHITECTURE.md
│   │
│   └── backend/                          # FastAPI services
│       ├── ingestion_orchestration_fastapi_app/  # Port 8002
│       ├── ml_inference_fastapi_app/              # Port 8001
│       ├── gpu_umap_service/                      # Port 8003
│       ├── AGENTS.md                     # ← Backend development guide
│       └── ARCHITECTURE.md
│
├── 🗺️ Dev Graph (Code Knowledge Graph)
│   ├── tools/dev-graph-ui/              # Next.js UI (port 3001)
│   │   └── src/
│   │
│   ├── developer_graph/                  # FastAPI API (port 8080)
│   │   ├── api.py                        # Main application
│   │   ├── routes/                       # API endpoints
│   │   │   └── AGENTS.md                # ← Route handler guide
│   │   ├── AGENTS.md                     # ← Dev Graph module guide
│   │   └── architecture.md               # ← Data model & pipeline
│   │
│   └── dev_graph_audit/                  # Analysis reports
│       └── AGENTS.md                     # ← Audit guide
│
├── 📚 Shared Resources
│   ├── docs/                            # Project documentation
│   │   ├── architecture.md              # ← System architecture (both apps)
│   │   ├── sprints/                     # Sprint planning & retrospectives
│   │   └── reference_guides/            # Technical guides
│   ├── .cursor/rules/                   # AI coding guidelines
│   │   ├── projectrules.mdc
│   │   ├── backend/                     # Backend-specific rules
│   │   └── frontend/                    # Frontend-specific rules
│   ├── utils/                           # Shared Python utilities
│   ├── database/                        # Database connectors
│   └── config.py                        # Global configuration
│
└── 🔧 DevOps & Scripts
    ├── docker-compose.yml               # Service orchestration
    ├── start_pixel_detective.{ps1,bat}  # Launch media search
    ├── start_dev_graph.{ps1,bat}        # Launch knowledge graph
    └── scripts/                         # Automation scripts
```

---

## 🚀 **Getting Started as an AI Agent**

### **Step 1: Identify Your Task Context**

| If User Asks About... | Navigate To... |
|----------------------|----------------|
| **Media search, images, CLIP, BLIP, vector search** | [`backend/AGENTS.md`](backend/AGENTS.md) + [`frontend/AGENTS.md`](frontend/AGENTS.md) |
| **Frontend UI, React, Next.js, Chakra UI** | [`frontend/AGENTS.md`](frontend/AGENTS.md) |
| **Backend APIs, FastAPI, microservices** | [`backend/AGENTS.md`](backend/AGENTS.md) |
| **Git history, Neo4j, knowledge graph** | [`developer_graph/AGENTS.md`](developer_graph/AGENTS.md) |
| **Dev Graph API routes** | [`developer_graph/routes/AGENTS.md`](developer_graph/routes/AGENTS.md) |
| **Data quality, auditing, graph validation** | [`dev_graph_audit/AGENTS.md`](dev_graph_audit/AGENTS.md) |
| **Architecture, system design** | [`docs/architecture.md`](docs/architecture.md) |
| **Deployment, Docker, setup** | This file (Deployment section below) |
| **Sprint planning, retrospectives** | [`docs/sprints/`](docs/sprints/) |

### **Step 2: Follow the Relevant AGENTS.md**

Each subdirectory's AGENTS.md contains:
- **Focused guidelines** for that specific area
- **Development patterns** with code examples
- **Common tasks** and how to accomplish them
- **Debugging guides** for typical issues
- **Architecture patterns** specific to that component

### **Step 3: Check Cursor Rules (`.cursor/rules/`)**

For detailed coding patterns:
- **Always applicable**: `projectrules.mdc`
- **Backend work**: `.cursor/rules/backend/*.mdc`
- **Frontend work**: `.cursor/rules/frontend/*.mdc`
- **Debugging**: `.cursor/rules/debugging.mdc`
- **Sprint work**: `.cursor/rules/sprint-planning.mdc`

---

## 🔧 **Quick Start Commands**

### **Pixel Detective (Media Search)**
```powershell
# Full stack
.\start_pixel_detective.ps1

# Individual services (development)
docker compose up -d qdrant_db
uvicorn backend.ingestion_orchestration_fastapi_app.main:app --reload --port 8002
uvicorn backend.ml_inference_fastapi_app.main:app --reload --port 8001
cd frontend && npm run dev
```

**Access:** http://localhost:3000

### **Dev Graph (Knowledge Graph)**
```powershell
# Full stack
.\start_dev_graph.ps1

# Individual services (development)
docker compose up -d neo4j_db
uvicorn developer_graph.api:app --reload --port 8080
cd tools/dev-graph-ui && npm run dev
```

**Access:** http://localhost:3001

---

## 🎯 **Key Architectural Principles**

### **Shared Patterns Across Both Applications**

1. **Microservices Architecture**
   - Each service has a single responsibility
   - Services communicate via REST APIs
   - Independent scaling and deployment

2. **Dependency Injection**
   - Use `Depends()` in FastAPI
   - Never import `main.py` from routers
   - Centralized state in `dependencies.py`

3. **Async/Await Everywhere**
   - All I/O operations are async
   - Use `asyncio.to_thread()` for CPU-bound work
   - Background tasks for long-running operations

4. **React Query for Frontend**
   - All API calls use React Query
   - Never manual `useEffect` + `useState` for API calls
   - Background polling for real-time updates

5. **Type Safety**
   - Python: Type hints everywhere
   - TypeScript: Strict mode enabled
   - Pydantic models for API validation

6. **Error Handling**
   - Structured error responses
   - Graceful degradation
   - User-friendly error messages

### **Application-Specific Patterns**

#### **Pixel Detective**
- **GPU Management**: Exclusive locks with `asyncio.Lock()`
- **Batch Processing**: Dynamic batch sizing based on GPU memory
- **Vector Search**: Qdrant for semantic similarity
- **Image Optimization**: Next.js Image component

#### **Dev Graph**
- **Graph Operations**: Batch UNWIND for bulk creates
- **Temporal Tracking**: Git commit history as timeline
- **Relationship Derivation**: Evidence-based linking
- **WebGL Visualization**: DeckGL for large graphs

---

## ⚠️ **Critical Warnings**

### **Never Do These (Universal)**

❌ **Don't**: Import `main.py` from routers (circular imports)  
❌ **Don't**: Block event loop with synchronous operations  
❌ **Don't**: Use browser APIs in React render (hydration errors)  
❌ **Don't**: Skip error handling  
❌ **Don't**: Hardcode environment-specific values  
❌ **Don't**: Create databases without indexes  

### **Always Do These (Universal)**

✅ **Read existing code first** - Understand patterns  
✅ **Follow conventions** - Match existing style  
✅ **Test your changes** - Automated tests required  
✅ **Update documentation** - Keep docs current  
✅ **Handle errors gracefully** - User-friendly messages  
✅ **Log appropriately** - Structured logging  

---

## 📚 **Documentation Hierarchy**

When seeking information, follow this hierarchy:

1. **Specific AGENTS.md** - For focused task guidance
   - `backend/AGENTS.md` for backend work
   - `frontend/AGENTS.md` for frontend work  
   - `developer_graph/AGENTS.md` for Dev Graph work

2. **Architecture docs** - For system design understanding
   - `docs/architecture.md` for overall architecture
   - Component-specific ARCHITECTURE.md files

3. **Cursor rules** - For detailed coding patterns
   - `.cursor/rules/` directory

4. **Code comments** - For implementation details
   - Inline documentation in source files

5. **Sprint docs** - For historical context
   - `docs/sprints/` directory

---

## 🎓 **Learning Path**

### **For New Agents**

**Week 1: Orientation**
- Read this file for navigation
- Read `README.md` for project overview
- Explore `docs/architecture.md` for system design
- Review appropriate subdirectory AGENTS.md

**Week 2: Deep Dive**
- Choose one application (Pixel Detective OR Dev Graph)
- Study its specific AGENTS.md and ARCHITECTURE.md
- Follow development patterns in `.cursor/rules/`
- Make a small documentation improvement

**Week 3: Implementation**
- Implement a small feature or bug fix
- Follow testing guidelines
- Update relevant documentation
- Review your changes against coding standards

---

## 📞 **Getting Help**

### **Where to Look**

1. **This file** - For navigation and general guidelines
2. **Subdirectory AGENTS.md** - For focused technical guidance
3. **Architecture docs** - For system design context
4. **Cursor rules** - For detailed coding patterns
5. **Code examples** - For implementation reference

### **Common Issues by Component**

| Issue Category | Primary Reference |
|---------------|------------------|
| Pixel Detective Backend | `backend/AGENTS.md` |
| Pixel Detective Frontend | `frontend/AGENTS.md` |
| Dev Graph Module | `developer_graph/AGENTS.md` |
| Dev Graph API Routes | `developer_graph/routes/AGENTS.md` |
| Data Quality | `dev_graph_audit/AGENTS.md` |
| System Architecture | `docs/architecture.md` |
| Debugging | `.cursor/rules/debugging.mdc` |

---

## 🎯 **Success Criteria**

An AI agent is successful when:

✅ **Navigation**: Quickly finds relevant documentation  
✅ **Code Quality**: Matches existing patterns and conventions  
✅ **Functionality**: Features work as expected  
✅ **Testing**: Adequate test coverage  
✅ **Documentation**: Keeps docs current  
✅ **Performance**: No regressions  
✅ **Maintainability**: Changes are easy for next agent to understand

---

**Last Updated**: Sprint 11 (October 2025)  
**Version**: 3.0 (Simplified Navigation)  
**Status**: Production Guidelines

**🚀 Built with AI | 🗺️ Follow the Guides | 💡 Navigate with Purpose**

---

## 📖 **Quick Reference Links**

### **Pixel Detective Documentation**
- [Backend AGENTS](backend/AGENTS.md)
- [Frontend AGENTS](frontend/AGENTS.md)
- [Backend Architecture](backend/ARCHITECTURE.md)
- [Frontend Architecture](frontend/ARCHITECTURE.md)

### **Dev Graph Documentation**
- [Dev Graph AGENTS](developer_graph/AGENTS.md)
- [Dev Graph Architecture](developer_graph/architecture.md)
- [Route Handlers AGENTS](developer_graph/routes/AGENTS.md)
- [Audit Guide](dev_graph_audit/AGENTS.md)

### **Project-Wide Documentation**
- [System Architecture](docs/architecture.md)
- [Developer Guide](DEVELOPER_GUIDE.md)
- [Sprint Documentation](docs/sprints/)
- [Cursor Rules Index](.cursor/rules/)

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
