## playground-guidance-agent

> **Pension Guidance Chat**: FCA-compliant AI agent with advanced agentic learning (Generative Agents + MedAgent-Zero/SEAL framework).

# CLAUDE.md - AI Assistant Guide

**Pension Guidance Chat**: FCA-compliant AI agent with advanced agentic learning (Generative Agents + MedAgent-Zero/SEAL framework).

## Quick Start

```bash
# Backend
uv sync && uv run uvicorn guidance_agent.api.main:app --reload  # http://localhost:8000

# Frontend
cd frontend && npm install && npm run dev  # http://localhost:3000

# Docker (recommended)
docker-compose up -d postgres phoenix backend  # Backend stack only
docker-compose up -d                           # Full stack

# Tests
pytest                      # Backend (214 tests)
cd frontend && npm test     # Frontend (203 tests)
```

## Key Architecture Patterns

### Dual Agent System
- **Advisor Agent** (`src/guidance_agent/advisor/agent.py`): Provides FCA-compliant guidance with learning capabilities
- **Customer Agent** (`src/guidance_agent/customer/agent.py`): Simulated customers for testing/training
- **Compliance Validator** (`src/guidance_agent/compliance/validator.py`): LLM-as-judge real-time validation

### Learning System
- **Memory Stream**: Observations + reflections with importance scoring (0-1) and temporal decay
- **Case-Based Learning**: Extracts successful consultation patterns
- **Rule Learning**: Discovers and refines guidance principles over time
- **Multi-faceted RAG**: Retrieves from 5 sources (memories, cases, rules, FCA knowledge, pension knowledge)

### Conversational Quality System
- **Natural Dialogue Flow**: Signposting, transitions, varied phrasing to avoid repetitive language
- **Conversation Phase Awareness**: Detects opening/middle/closing phases for appropriate tone
- **Emotional Intelligence**: Assesses customer state (anxious, confident, confused) and adapts responses
- **Quality Tracking**: Calculates 0-1 conversational quality score per consultation (variety, signposting, personalization, engagement)
- **Dialogue Learning**: Captures successful conversational patterns in case base for retrieval
- **FCA Neutrality (Phase 7)**: Strict distinction between process warmth (compliant) and circumstantial evaluation (prohibited)

### Tech Choices
- **Backend**: FastAPI + SQLAlchemy (async) + pgvector + LiteLLM + Jinja2 templates
- **Frontend**: Nuxt 3 + Vercel AI SDK (SSE streaming) + Pinia
- **Database**: PostgreSQL + pgvector (1536-dim embeddings)
- **Observability**: Arize Phoenix (LLM tracing) - http://localhost:6006
- **Package Manager**: uv (Python)

## Database Schema (7 Core Models)

1. **Memory**: Agent memory stream with vector embeddings, importance scoring, temporal decay
2. **Case**: Successful consultations with task categorization, vector similarity, **dialogue_techniques (JSONB)**
3. **Rule**: Learned guidance principles with confidence scores, evidence tracking
4. **Consultation**: Full conversation history (JSONB), outcome metrics, compliance scores, **conversational_quality (Float 0-1)**, **dialogue_patterns (JSONB)**
5. **FCAKnowledge**: Categorized compliance content with vector search
6. **PensionKnowledge**: Domain knowledge (74KB) with category/subcategory structure
7. **SystemSettings**: Single-row config (ID=1) for model params, compliance toggles

**All embedding fields use pgvector with 1536 dimensions**

## Critical: Phoenix Tracing Setup

**IMPORTANT**: Import order matters! Phoenix must instrument LiteLLM BEFORE it's imported.

**Correct** (`src/guidance_agent/api/main.py`):
```python
from fastapi import FastAPI

# MUST BE FIRST guidance_agent import
from guidance_agent.core import llm_config  # noqa: F401

# Now safe to import routers (which use advisor/litellm)
from guidance_agent.api.routers import consultations
```

**Incorrect** (will break tracing):
```python
from guidance_agent.api.routers import consultations  # ❌ Imports litellm first!
from guidance_agent.core import llm_config  # Too late
```

**Verification**: After startup, check logs for:
```
✓ Phoenix tracing enabled (BatchSpanProcessor): http://localhost:4317
  LiteLLM instrumentation: ACTIVE
  ✓ Test span created - should appear in Phoenix UI
```

## Jinja2 Prompt Templates

**All prompts are Jinja2 templates** in `src/guidance_agent/templates/`:

**Advisor**:
- `advisor/system.jinja2`: System instructions
- `advisor/guidance.jinja2`: Main guidance prompt
- `advisor/guidance_reasoning.jinja2`: With chain-of-thought
- `advisor/reflection.jinja2`: Reflection generation
- `advisor/rule_extraction.jinja2`: Rule learning

**Compliance**:
- `compliance/validation.jinja2`: LLM-as-judge validation

**Customer**:
- `customer/system.jinja2`: Customer persona
- `customer/message.jinja2`: Message generation

**Loading templates** (`src/guidance_agent/templates/loader.py`):
```python
template = env.get_template("advisor/guidance.jinja2")
prompt = template.render(
    customer_message="...",
    context_memories=[...],
    context_cases=[...],
    context_rules=[...],
    fca_guidelines=[...]
)
```

**Template best practices**:
- Add comments: `{# Purpose, Inputs, Output #}`
- Test with `tests/templates/test_template_rendering.py`
- All 20 templates have regression tests

## Multi-Faceted RAG Retrieval

**5-source retrieval** (`src/guidance_agent/retrieval/retriever.py`):

1. **Memories**: Recent + important observations/reflections (temporal decay + importance weighting)
2. **Cases**: Similar consultations (vector similarity)
3. **Rules**: Relevant learned principles (vector similarity)
4. **FCA Knowledge**: Compliance guidelines (vector similarity)
5. **Pension Knowledge**: Domain expertise (vector similarity)

**Retrieval strategy**:
- Recency: Temporal decay function
- Relevance: Cosine similarity (pgvector)
- Importance: 0-1 scoring for memories

## LiteLLM Multi-Provider

**Supported providers** (auto-detected from model name):
```bash
# OpenAI
LLM_MODEL=gpt-4-turbo-preview

# Anthropic
LLM_MODEL=claude-3-5-sonnet-20241022

# AWS Bedrock
LLM_MODEL=bedrock/anthropic.claude-3-5-sonnet-20241022-v2:0

# Azure OpenAI
LLM_MODEL=azure/gpt-4

# LM Studio (local)
OPENAI_API_BASE=http://localhost:1234/v1
LLM_MODEL=local-model

# Ollama (local)
LLM_MODEL=ollama/llama2
```

**Prompt caching** (Anthropic/OpenAI):
```python
messages = [
    {
        "role": "system",
        "content": [
            {
                "type": "text",
                "text": long_system_prompt,
                "cache_control": {"type": "ephemeral"}  # Cache this
            }
        ]
    },
    {"role": "user", "content": user_message}
]
```

## Environment Variables

**Critical non-obvious settings**:

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/guidance_agent

# LLM (choose one provider)
OPENAI_API_KEY=sk-...              # OpenAI
ANTHROPIC_API_KEY=sk-ant-...       # Anthropic
AWS_ACCESS_KEY_ID=...              # AWS Bedrock
AZURE_API_KEY=...                  # Azure OpenAI
LLM_MODEL=claude-3-5-sonnet-20241022

# Embeddings (MUST match database Vector dimensions)
EMBEDDING_MODEL=text-embedding-3-small
EMBEDDING_DIMENSIONS=1536  # ⚠️ If changed, need migration + re-embed all data

# Phoenix Tracing
PHOENIX_COLLECTOR_ENDPOINT=http://localhost:4317
PHOENIX_AUTO_SETUP=true
ENVIRONMENT=development  # Required for auto-setup

# App
ENVIRONMENT=development
LOG_LEVEL=INFO
```

## Test Structure

**Backend** (214 tests):
- `tests/api/` (146): CRUD for 7 models + SSE streaming
- `tests/integration/` (8): End-to-end advisor flow + learning cycle
- `tests/templates/` (40): All 20 Jinja2 templates
- `tests/regression/` (20): Template migration validation

**Frontend** (203 tests):
- `frontend/tests/` (83): Components, pages, forms
- `frontend/tests/e2e/` (99): Full workflows
- `frontend/tests/compliance-details.spec.ts` (21): Validation reasoning UI

**Key test commands**:
```bash
pytest tests/api/                   # API tests
pytest tests/templates/             # Template tests
cd frontend && npm run test:e2e     # E2E tests
```

## Key Paths

```
src/guidance_agent/
├── api/routers/          # REST endpoints (SSE streaming for chat)
├── advisor/agent.py      # Main advisor agent
├── compliance/validator.py  # LLM-as-judge validation
├── core/
│   ├── llm_config.py     # ⚠️ MUST import first for Phoenix tracing
│   ├── llm.py            # LiteLLM integration
│   └── embeddings.py     # Vector embeddings (1536 dims)
├── learning/             # Case-based + rule learning + reflection
├── retrieval/retriever.py  # Multi-faceted RAG
├── models/               # 7 SQLAlchemy models
└── templates/            # 20 Jinja2 prompt templates

frontend/app/
├── components/admin/ComplianceDetails.vue  # Shows LLM validation reasoning
├── composables/useChat.ts  # SSE streaming integration
└── pages/admin/          # Admin dashboard for memories/cases/rules

tests/                    # Backend tests (214)
frontend/tests/           # Frontend tests (203)
```

## Documentation

**Essential docs**:
- `README.md`: User-facing quick start
- `specs/architecture.md`: System architecture (41KB)
- `specs/fca-compliance-manual.md`: Compliance guidelines (80KB)
- `specs/JINJA_MIGRATION_COMPLETE.md`: Template system details
- `docs/DEPLOYMENT.md`: Production deployment (700+ lines)
- `docs/TESTING.md`: Testing guide

**Research papers**:
- Generative Agents (https://arxiv.org/abs/2304.03442): Memory stream, reflection, importance scoring
- Agent Hospital/MedAgent-Zero (https://arxiv.org/abs/2405.02957): SEAL framework, closed-loop learning

## FCA Neutrality Requirements (Phase 7)

**CRITICAL**: Conversational warmth MUST be about process, NOT circumstances.

**The Distinction**:
- ✅ **Process Warmth (COMPLIANT)**: "Great question!", "I'm glad you're thinking about this", "Let's work through this together"
- ❌ **Circumstantial Evaluation (PROHIBITED)**: "You're doing well!", "That's a solid foundation", "£150k is excellent for your age"

**Prohibited Patterns**:
- Evaluative language: "solid foundation", "doing well", "on track", "good start"
- Social proof + circumstances: "Some people in your situation find it helpful to..."
- Adequacy assessments: "You're ahead/behind where you should be"
- Enthusiastic responses to amounts: "Great! £150k is a strong position"

**Compliant Patterns**:
- Neutral fact-stating: "You have £X in your pension at age Y"
- Factor listing: "Whether this meets your needs depends on [factors]..."
- Exploration offers: "Would you like to explore what you'll need for retirement?"
- Signposting for assessment: "An adviser can assess whether this is adequate for your goals"

**Neutral Response Structure** (4 steps):
1. Acknowledge neutrally: "Thank you for sharing that"
2. State facts: "You have £X at age Y"
3. List adequacy factors: "Whether this meets your needs depends on..."
4. Offer exploration OR signpost: "Would you like to explore..." / "An adviser can assess..."

**Why This Matters**:
- Evaluating adequacy = suitability assessment = regulated advice (FCA violation)
- Process warmth enhances engagement WITHOUT crossing into advice
- The validator enforces this distinction with 4 critical checks

**Test Coverage**: 23 tests in `tests/integration/test_fca_neutrality.py`

## Common Gotchas

1. **Embedding dimension mismatch**: If changing `EMBEDDING_DIMENSIONS`, must:
   - Create Alembic migration to alter Vector columns
   - Run `python scripts/bootstrap_all_knowledge.py` to re-embed

2. **Phoenix not showing traces**: Check import order in `main.py` (llm_config MUST be first)

3. **SSE streaming**: Backend uses `text/event-stream`, frontend uses Vercel AI SDK's `useChat`

4. **SystemSettings is single-row**: Always ID=1, use `UPDATE` not `INSERT`

5. **Template tests**: All 20 templates have tests in `tests/templates/` - update when modifying prompts

6. **FCA Neutrality**: NEVER evaluate customer circumstances - only process. See "FCA Neutrality Requirements" section above

---

**Version**: 1.0.0 | **Status**: ✅ Production Ready | **Tests**: 417 passing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/looksystems)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/looksystems)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
