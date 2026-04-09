## dora

> **Dora** is the codename for **DocAssist's UpToDate/NotebookLM Killer** - a premium, Apple-quality medical knowledge platform that doctors will love to use and pay for. This is NOT just another RAG system. This is the future of medical knowledge access.

# CLAUDE.md - DocAssist Dora Project Directives

## 🎯 Project Vision

**Dora** is the codename for **DocAssist's UpToDate/NotebookLM Killer** - a premium, Apple-quality medical knowledge platform that doctors will love to use and pay for. This is NOT just another RAG system. This is the future of medical knowledge access.

**The Goal:** Build a system so good that doctors would "literally die to use it" - seamlessly integrated across EMR, academic writing, appointments, and real-time medical knowledge.

---

## 🧰 MANDATORY DEVELOPMENT TOOLKIT

**YOU MUST USE THESE TOOLS FOR ALL DEVELOPMENT IN THIS PROJECT:**

### 1. Spec-Kit (Specification-Driven Development)
**Repository:** https://github.com/github/spec-kit

**What it is:** A framework that makes specifications executable - transforming requirements into working code through AI agents.

**How to use:**
```bash
# Initialize (already done)
uvx --from git+https://github.com/github/spec-kit.git specify init . --ai claude

# Workflow commands:
/speckit.constitution  # Define project principles
/speckit.specify       # Transform requirements to PRD
/speckit.clarify       # Resolve ambiguities
/speckit.plan          # Create technical architecture
/speckit.tasks         # Generate executable tasks
/speckit.analyze       # Validate consistency
/speckit.implement     # Execute with test-first methodology
```

**Why we use it:**
- Medical AI requires regulatory compliance (HIPAA, FDA) - constitution enforces this
- Multi-stakeholder alignment (clinicians, engineers, compliance)
- Documentation for regulatory approval
- Specifications become living architecture guides

### 2. Ralph-Wiggum (Autonomous Iteration Loop)
**Repository:** https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum

**What it is:** An autonomous iteration loop that enables self-correcting, continuous development until success criteria are met.

**How to use:**
```bash
/ralph-loop "Your task description" --max-iterations N --completion-promise "SUCCESS_SIGNAL"
```

**Why we use it:**
- Autonomous refinement of complex medical algorithms
- Cost-effective development ($50K contracts for <$300 API costs)
- Continuous verification ensures medical accuracy
- Quality assurance built into development process

**Best practices:**
- Always include medical domain validation in completion criteria
- Use strict iteration limits (medical workflows need bounds)
- Build comprehensive test suites BEFORE running ralph loops
- Log all iterations for compliance audit

---

## 🏗️ PROJECT ECOSYSTEM

### Integrated Products (All Must Work Together Seamlessly)

| Product | Repository | Purpose | Status |
|---------|-----------|---------|--------|
| **Dora** (this repo) | drshailesh88/Dora | Medical Knowledge Platform (UpToDate killer) | Planning |
| **DocAssist EMR** | drshailesh88/emr | Electronic Medical Records | Active |
| **Academic Writing** | drshailesh88/cursor_for_academic_writing | AI-powered medical writing | 90% Complete |
| **Practice Manager** | drshailesh88/appointment_system | Appointment & billing | Active |

### Shared Components Across Products

1. **Patient Registry** - Unified across EMR ↔ Appointments ↔ Dora
2. **Authentication** - Single sign-on for all products
3. **Medical Knowledge Base** - Shared embeddings and retrieval
4. **Voice Agent** - "Hey DocAssist" works across all products
5. **Notification System** - Unified SMS/WhatsApp/Email

---

## 📐 ARCHITECTURAL PRINCIPLES

### Constitution (Non-Negotiable)

1. **Privacy-First**: All patient data processed locally. No cloud uploads without explicit consent.
2. **Offline-Capable**: Must work without internet (rural India = 40% of users)
3. **HIPAA/DISHA Compliant**: All features validated for healthcare compliance
4. **Draft-Mode AI**: All LLM outputs require physician confirmation before persistence
5. **Zero Commission**: No hidden fees on patient payments
6. **Data Ownership**: Doctors own their data, can export anytime
7. **Evidence-Based**: All recommendations cite sources with confidence scores
8. **Hallucination Prevention**: Strict grounding in retrieved documents only

### Technical Mandates

1. **Library-First**: All features as reusable libraries (spec-kit Article I)
2. **Test-First**: TDD mandatory (spec-kit Article III)
3. **API-First**: Every feature exposed via REST/GraphQL
4. **Simplicity**: Max 3 projects per feature (spec-kit Article VII)
5. **No Over-Abstraction**: Use frameworks directly (spec-kit Article VIII)

---

## 🔧 TECHNOLOGY STACK

### Core Infrastructure

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **LLM (Cloud)** | Claude 3.5 Sonnet / GPT-4o | Best medical reasoning |
| **LLM (Local)** | Ollama + Qwen 2.5 (3B/7B) | Offline capability |
| **Embeddings** | PubMedBERT / Clinical ModernBERT | Medical domain optimization |
| **Vector DB** | Qdrant (primary) + ChromaDB (local) | Hybrid search, HIPAA-compliant |
| **Graph DB** | Neo4j | Medical knowledge graphs, UMLS |
| **Relational DB** | PostgreSQL (cloud) / SQLite (local) | Structured data |
| **Document Parsing** | RAGFlow / MinerU | Complex medical PDFs |
| **Reranking** | Cohere rerank / ColBERT | High-precision retrieval |

### Application Layer

| Component | Technology | Rationale |
|-----------|------------|-----------|
| **Desktop UI** | Flet (Python) | Cross-platform, rapid dev |
| **Web UI** | Next.js 14 + React 18 | Modern, responsive |
| **Mobile** | Flutter | iOS + Android from single codebase |
| **Backend API** | FastAPI | Async, type-safe, fast |
| **Voice** | Whisper (STT) + Piper (TTS) | Fully local, no cloud |

### RAG Pipeline (Portable RAG Enhanced)

```
Query Pipeline:
Query → Multi-Query Gen → HyDE Embedding → Dense + Sparse Retrieval → RRF Fusion
      → Cohere Rerank → Contextual Compression → LLM Synthesis → Citation Generation

Ingestion Pipeline:
PDFs → RAGFlow Parsing → Adaptive Chunking → Metadata Extraction → PubMedBERT Embeddings
     → Qdrant Vector Storage → Neo4j Graph Indexing
```

---

## 🎯 WHAT MAKES US DIFFERENT FROM UPTODATE

| Feature | UpToDate | Dora |
|---------|----------|------|
| **Cost** | $559/year | ₹999/month (~$12/month) |
| **Offline** | No | Yes (critical for India) |
| **EMR Integration** | Third-party | Native (DocAssist suite) |
| **Voice Control** | No | "Hey DocAssist" |
| **Patient Context** | None | RAG over patient records |
| **Academic Writing** | None | Built-in with PubMed |
| **Local Data** | Cloud-only | Doctor owns data |
| **Indian Guidelines** | Limited | Native support |
| **Personalization** | None | Learns doctor's specialty |

---

## 📋 DEVELOPMENT WORKFLOW

### For Every Feature:

1. **Specify** → Write PRD in `.specify/spec.md`
2. **Plan** → Generate architecture in `.specify/plan.md`
3. **Tasks** → Break down in `.specify/tasks.md`
4. **Implement** → Use ralph-loop with test-first TDD
5. **Review** → Clinical validation before merge
6. **Document** → Update CLAUDE.md and sync to AGENTS.md, CODEX.md, GEMINI.md, GROK.md

### Commit Message Format:
```
<type>(<scope>): <description>

Types: feat, fix, docs, refactor, test, chore
Scopes: rag, emr, voice, ui, api, auth, billing
```

---

## 🚨 REMINDERS FOR EVERY SESSION

1. **ALWAYS** check this file at session start
2. **USE** spec-kit commands for any new feature
3. **RUN** ralph-loop for complex implementations
4. **TEST** before committing (80% coverage minimum)
5. **SYNC** documentation across all AI instruction files
6. **CITE** sources in all medical content
7. **VALIDATE** with clinical test cases

---

## 📁 Key Files to Reference

- `/CLAUDE.md` - This file (AI development directives)
- `/MEDICAL_RAG_RESEARCH.md` - State-of-the-art RAG research
- `/.specify/spec.md` - Current feature specifications
- `/.specify/plan.md` - Technical implementation plan
- `/.specify/constitution.md` - Project principles
- `/.claude/commands/` - Custom slash commands

---

*Last Updated: January 2026*
*Project Codename: Dora*
*Vision: The Apple of Medical Knowledge Platforms*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drshailesh88)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/drshailesh88)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
