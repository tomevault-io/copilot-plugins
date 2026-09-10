## replyon-customer-agent

> A production-grade AI customer support system that handles queries automatically across multiple input types (text, voice, image) with no human needed for most tickets.

# Multi-modal Customer Support Agent — Project Context

## What We're Building
A production-grade AI customer support system that handles queries automatically across multiple input types (text, voice, image) with no human needed for most tickets.

## How It Works
1. Customer submits a ticket via text, voice recording, or image/screenshot
2. Channel Adapter normalizes it into a unified Ticket schema
3. Classifier Agent detects intent, priority, and category
4. RAG Agent searches the product knowledge base and generates an answer
5. Escalation Agent hands off to a human if confidence is low or issue is complex
6. Response sent back to customer + dashboard updated

## Agent Flow
```
Customer Input (text / voice / image)
        ↓
  Channel Adapter  →  unified Ticket schema
        ↓
  Classifier Agent  →  intent + priority + category + confidence_score
        ↓
  RAG Agent  →  search knowledge base → generate answer
        ↓
  Escalation Agent  →  human handoff if confidence < threshold
        ↓
  Response → Customer + Dashboard updated
```

## Tech Stack

### Backend
- **Python 3.12** — runtime
- **FastAPI** — API gateway, webhooks, WebSocket
- **Celery + Redis** — background task processing
- **Claude Sonnet API (claude-sonnet-4-6)** — classifier + RAG answer generation + vision
- **LangChain** — RAG pipeline, document loaders
- **OpenAI Whisper API** — speech-to-text for voice input
- **Claude Vision** — understand screenshots and images

### Data & Storage
- **FAISS** (dev) → **Pinecone** (prod) — vector store for knowledge base
- **PostgreSQL** — tickets, users, human agent queue
- **Redis** — FAQ cache, session state
- **text-embedding-3-small** — embedding model for docs

### Frontend
- **Next.js 14** — customer chat UI + agent dashboard
- **shadcn/ui + Tailwind CSS** — UI components
- **Clerk** — auth for human agents
- **WebSockets** — real-time chat
- **Recharts** — metrics dashboard charts

### Monitoring & Evaluation
- **RAGAS** — RAG answer quality evaluation
- **Langfuse** — LLM observability, cost tracking
- **Sentry** — error tracking
- **Prometheus + Grafana** — system metrics

### Infrastructure
- **Docker + Docker Compose** — containerization
- **AWS EC2** — production hosting
- **Nginx** — reverse proxy
- **GitHub Actions** — CI/CD

### Inbound Channels
- Web chat — WebSocket (native FastAPI)
- Email — SendGrid Inbound Parse
- WhatsApp — Twilio WhatsApp API
- Voice — Twilio Voice + Whisper STT

## Project Structure
```
Customer_agent_project/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI entrypoint
│   │   ├── api/
│   │   │   └── routes/
│   │   │       ├── tickets.py       # POST /tickets
│   │   │       ├── escalation.py    # Human queue endpoints
│   │   │       └── health.py
│   │   ├── agents/
│   │   │   ├── classifier.py        # Intent/priority classifier
│   │   │   ├── rag_agent.py         # LangChain RAG chain
│   │   │   ├── faq_agent.py         # Cache-backed FAQ resolver
│   │   │   └── escalation_agent.py  # Human handoff logic
│   │   ├── pipelines/
│   │   │   ├── stt.py               # Whisper speech-to-text
│   │   │   ├── vision.py            # Claude vision processing
│   │   │   └── embeddings.py        # Embedding + vector store
│   │   ├── knowledge_base/
│   │   │   ├── ingest.py            # Chunk, embed, upsert docs
│   │   │   └── retriever.py         # Similarity search wrapper
│   │   ├── monitoring/
│   │   │   ├── metrics.py           # Prometheus metrics
│   │   │   └── eval.py              # RAGAS eval framework
│   │   ├── models/
│   │   │   └── schemas.py           # Pydantic schemas
│   │   └── config.py                # Settings, env vars
│   ├── tests/
│   ├── Dockerfile
│   └── requirements.txt
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ChatWidget.jsx
│   │   │   ├── VoiceRecorder.jsx
│   │   │   ├── ImageUploader.jsx
│   │   │   └── TicketStatus.jsx
│   │   └── dashboard/
│   │       ├── MetricsDashboard.jsx
│   │       ├── TicketQueue.jsx
│   │       └── EvalPanel.jsx
│   ├── Dockerfile
│   └── package.json
│
├── infra/
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   ├── nginx.conf
│   └── ec2-setup.sh
│
├── knowledge_base/
│   └── docs/                        # Product docs, FAQs, manuals
│
└── .github/
    └── workflows/
        └── deploy.yml
```

## Build Order (day by day)
- **day 1** — FastAPI gateway + Ticket schema + Classifier Agent
- **day 2** — RAG pipeline (ingest docs → embed → FAISS → retrieval chain)
- **day 3** — Multi-modal inputs (Whisper STT + Claude Vision)
- **day 4** — Escalation Agent + human queue + fallback logic
- **day 5** — Monitoring dashboard (Langfuse + RAGAS + Prometheus)
- **day6** — Docker + EC2 deploy + GitHub Actions CI/CD

## Key Design Decisions
- All inbound channels normalize to a single `Ticket` Pydantic schema before hitting the classifier
- Confidence threshold < 0.7 triggers escalation to human queue
- Use prompt caching on Claude API for repeated knowledge base context (cost saving)
- FAISS locally, migrate to Pinecone before production deploy
- Every LLM call logged to Langfuse with latency + token usage

## Current Status
- Project just started
- Next step: scaffold full folder structure + write FastAPI entrypoint + Ticket schema + Classifier Agent

---
> Source: [Bhupatishyam55/Replyon_customer_agent](https://github.com/Bhupatishyam55/Replyon_customer_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
