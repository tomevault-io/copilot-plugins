## oms-agent

> Stai lavorando su un **OMS (Order Management System)** per un'azienda con ecommerce.

# CLAUDE.md — Agente IA Servizio Clienti → RAG Economico

## Contesto progetto

Stai lavorando su un **OMS (Order Management System)** per un'azienda con ecommerce.
Stack OMS esistente: Next.js 14+, React 19, Tailwind/DaisyUI, MongoDB (Mongoose), MSSQL/MySQL, IBM AS400, JWT auth, Winston logs.

Questo progetto Python è un **modulo esterno** all'OMS, collegato tramite API REST.
L'obiettivo finale è doppio:
1. **Breve termine:** agente IA servizio clienti che risponde su stato ordini/spedizioni/resi
2. **Lungo termine:** sistema RAG per analisi complesse di dati economici aziendali

Il developer conosce bene JavaScript/Node.js ma ha basi limitate di Python.
Spiega sempre il codice Python quando differisce dai pattern JS equivalenti.

---

## Stack tecnologico scelto

| Layer | Tecnologia | Motivo |
|---|---|---|
| Linguaggio | Python 3.11+ | Ecosistema LangChain/RAG superiore a JS |
| Agent framework | LangChain | Astrazione provider LLM + futuro RAG |
| LLM provider default | Anthropic Claude Haiku | Miglior tool calling, economico |
| LLM provider alternativo | OpenAI GPT-4o-mini | Flessibilità provider |
| Vector store (fase 2) | ChromaDB | Locale, zero infrastruttura |
| Embeddings (fase 2) | OpenAI text-embedding-3-small | Standard, economico |
| API server | FastAPI | Async nativo, simile a Express |
| Env management | python-dotenv | Equivalente di dotenv in Node |
| Package manager | pip + venv | Standard Python |

---

## Roadmap — 3 fasi progressive

### FASE 1 — Agente servizio clienti ✅ COMPLETATA

**Goal:** agente funzionante con tool calling verso API OMS reale.

#### Step 1.1 — Ambiente Python ✅
- venv creato con Python 3.12.3
- Dipendenze installate: langchain, langchain-anthropic, fastapi, uvicorn, python-dotenv, pymongo, langgraph-checkpoint-mongodb, python-json-logger

#### Step 1.2 — Agente minimale ✅
- `tools.py`: tool fittizi `get_order_status` e `get_orders_by_email`
- `agent.py`: AgentExecutor LangChain con Claude Haiku (`claude-haiku-4-5-20251001`)

#### Step 1.3 — Tool reali OMS ✅
- Contratto API definito e ricevuto dall'OMS team
- Autenticazione: system user AD esistente (bearer token, ~10h TTL, retry su 401)
- Tool implementati: `get_oms_agent_api_info`, `get_order_status`, `get_shipping_status`
- Rimosso `get_orders_by_email` per motivi di sicurezza (enumerazione ordini per email)
- Classe `OMSAgentAPI` con login automatico e gestione 401/404/429
- Sicurezza: entrambi `increment_id` + `email` obbligatori per ogni lookup

#### Step 1.4 — Memoria conversazione ✅
- `MemorySaver` LangGraph → poi migrato a `MongoDBSaver` (stesso MongoDB OMS)
- `memory.py`: `get_checkpointer()` ritorna `MongoDBSaver` connesso tramite `MONGO_URI`
- Collezione MongoDB: `agent_checkpoints` (creata automaticamente da LangGraph)
- Limite sessione: `MAX_SESSION_TURNS=10` turni (20 messaggi) via `trim_messages` pre_model_hook

#### Step 1.5 — API FastAPI ✅
- `main.py`: POST `/chat` con validazione Pydantic, logging strutturato, token tracking
- Endpoint statistiche token:
  - `GET /stats/total` — cumulativo dall'avvio
  - `GET /stats/session/{session_id}` — per sessione
  - `GET /stats/daily?date=YYYY-MM-DD` — per giorno
  - `GET /stats/monthly?month=YYYY-MM` — per mese

#### Step 1.6 — Logging e Token Tracking ✅
- `logger.py`: log JSON strutturati su file rotante `logs/oms_agent.log` (10MB × 5) + console
- `token_tracker.py`: estrae `usage_metadata` dai messaggi AIMessage, salva su MongoDB
  - Collezione: `agent_token_usage` con indici su `session_id`, `date`, `timestamp`
  - Prezzi Haiku: input $0.00080/1K token, output $0.00240/1K token

---

### FASE 1 — Struttura file attuale

```
oms-agent/
├── .env                  ← ANTHROPIC_API_KEY, OMS_*, MONGO_URI, MONGO_DB_NAME, MAX_SESSION_TURNS
├── agent.py              ← build_agent(checkpointer): create_agent + trim_messages hook
├── logger.py             ← get_logger(): JSON logger con file rotante + console
├── main.py               ← FastAPI: POST /chat + GET /stats/*
├── memory.py             ← get_checkpointer(): MongoDBSaver
├── token_tracker.py      ← TokenTracker: record(), get_by_session/day/month/total()
├── tools.py              ← OMSAgentAPI + 3 tool LangChain
├── requirements.txt      ← dipendenze fissate con pip freeze
└── logs/
    └── oms_agent.log     ← log JSON rotante (creato automaticamente)
```

---

### FASE 2 — RAG FAQ ed ecommerce ✅ COMPLETATA

#### Step 2.1 — Dipendenze RAG ✅
- Installati: `chromadb`, `langchain-chroma`, `langchain-openai`, `openai`, `beautifulsoup4`

#### Step 2.2 — Scraping e indicizzazione ✅
- `rag/scraper.py`: scraping HTML statico con BeautifulSoup, pulizia tag rumorosi
- `rag/ingest.py`: chunking (500 chars, overlap 50) + OpenAI embeddings + ChromaDB locale
- `rag/retriever.py`: singleton retriever, k=3 chunk per query
- Indice in `chroma_db/` (gitignored — riproducibile con `python -m rag.ingest`)
- Embeddings: OpenAI `text-embedding-3-small`

#### Step 2.3 — Tool RAG integrato ✅
- `tools.py`: aggiunto `search_faq_and_policies(question)` che chiama `rag/retriever.search_faq()`
- `agent.py`: tool aggiunto alla lista, system prompt aggiornato con istruzioni di routing
- Routing automatico: domande generali → RAG, ordini specifici → OMS API

#### Configurazione .env necessaria ✅
```
OPENAI_API_KEY=sk-...
ECOMMERCE_BASE_URL=https://www.esempio.it
ECOMMERCE_PAGES=/faq,/spedizioni,/resi,/pagamenti,/metodi-pagamento
```

#### Comando per aggiornare l'indice
```bash
source venv/bin/activate
python -m rag.ingest
```

---

### FASE 3 — Catalogo prodotti SQL (pianificata)

#### Step 1.1 — Ambiente Python pulito

```bash
python -m venv venv
source venv/bin/activate          # Mac/Linux
# oppure: venv\Scripts\activate   # Windows

pip install langchain langchain-anthropic langchain-openai python-dotenv fastapi uvicorn
```

File `.env`:
```
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...             # opzionale, per flessibilità
LLM_PROVIDER=anthropic
LLM_MODEL=claude-haiku-4-5-20251001
```

#### Step 1.2 — Primo agente minimale (obiettivo immediato)

Struttura file:
```
agent/
├── .env
├── main.py          ← entry point, avvia FastAPI
├── agent.py         ← logica agente LangChain
├── tools.py         ← tool che chiamano OMS API (fittizie ora, reali dopo)
└── memory.py        ← gestione sessione conversazione
```

**`tools.py`** — definisce i tool che l'agente può usare:
```python
from langchain.tools import tool

# Tool fittizio per sviluppo — sostituire con chiamata reale a OMS API
@tool
def get_order_status(order_id: str) -> str:
    """Recupera lo stato di un ordine dato il suo ID."""
    # TODO: sostituire con requests.get(f"{OMS_BASE_URL}/api/agent/orders/{order_id}")
    return f"Ordine {order_id}: In spedizione, previsto domani."

@tool
def get_orders_by_email(email: str) -> str:
    """Recupera tutti gli ordini associati a un'email cliente."""
    # TODO: chiamata reale OMS
    return f"Trovati 2 ordini per {email}: #1001 consegnato, #1002 in spedizione."
```

**`agent.py`** — logica centrale:
```python
from langchain_anthropic import ChatAnthropic
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from tools import get_order_status, get_orders_by_email

SYSTEM_PROMPT = """Sei un assistente del servizio clienti per il nostro ecommerce.
Rispondi sempre in italiano, sii cordiale e preciso.
Usa i tool disponibili per recuperare informazioni reali sugli ordini.
Non inventare mai dati — se non trovi informazioni, dillo chiaramente."""

def build_agent():
    llm = ChatAnthropic(model="claude-haiku-4-5-20251001")
    tools = [get_order_status, get_orders_by_email]

    prompt = ChatPromptTemplate.from_messages([
        ("system", SYSTEM_PROMPT),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
        MessagesPlaceholder("agent_scratchpad"),
    ])

    agent = create_tool_calling_agent(llm, tools, prompt)
    return AgentExecutor(agent=agent, tools=tools, verbose=True)
```

#### Step 1.3 — Aggiungere tool reali OMS

Creare su OMS Next.js le route dedicate all'agente:
```
GET  /api/agent/orders/:id
GET  /api/agent/orders?email=xxx
GET  /api/agent/shipping/:id
GET  /api/agent/returns/:id
```

Autenticazione: JWT service account dedicato, scope read-only, salvato in `.env`.

Tool list completa da implementare:
- `get_order_by_id` — stato ordine per ID
- `get_orders_by_email` — ordini per email cliente
- `get_shipping_status` — tracking spedizione
- `get_return_status` — stato reso
- `search_orders` — ricerca per parametri (data, stato, ecc.)

#### Step 1.4 — Memoria conversazione

Obiettivo: l'agente ricorda il contesto nella stessa sessione.

```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

# Dizionario in memoria — per produzione spostare su MongoDB
sessions = {}

def get_session_history(session_id: str):
    if session_id not in sessions:
        sessions[session_id] = ChatMessageHistory()
    return sessions[session_id]
```

Per produzione: sostituire con `MongoDBChatMessageHistory` da `langchain_community`.

#### Step 1.5 — API FastAPI

```python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel
from agent import build_agent

app = FastAPI()
agent_executor = build_agent()

class ChatRequest(BaseModel):
    session_id: str
    message: str

@app.post("/chat")
async def chat(req: ChatRequest):
    result = agent_executor.invoke({
        "input": req.message,
        "chat_history": []
    })
    return {"response": result["output"]}
```

Avvio: `uvicorn main:app --reload`

---

### FASE 2 — Primo RAG (dopo che la fase 1 funziona)

**Goal:** l'agente può rispondere anche su documenti non strutturati (PDF, note, report).

Nuove dipendenze:
```bash
pip install chromadb langchain-chroma
```

Concetti da imparare in questa fase:
- **Document loaders:** caricano testo da PDF, CSV, database
- **Text splitters:** dividono documenti lunghi in chunk
- **Embeddings:** trasformano testo in vettori numerici
- **Vector store:** salva e cerca vettori per similarità semantica
- **Retrieval chain:** combina ricerca vettoriale + LLM

Struttura aggiuntiva:
```
agent/
├── rag/
│   ├── ingest.py    ← carica documenti nel vector store
│   ├── retriever.py ← interroga il vector store
│   └── chain.py     ← chain RAG completa
└── data/            ← documenti da indicizzare
```

---

### FASE 3 — RAG economico complesso (obiettivo finale)

**Goal:** analisi avanzate su dati economici aziendali.

Fonti dati da integrare:
- **MongoDB** — ordini, spedizioni, resi (già nell'OMS)
- **MSSQL/MySQL** — dati ERP legacy
- **PDF/Excel** — report finanziari, bilanci
- **AS400** — dati storici magazzino/ordini

Capacità target:
- Query ibride: dati strutturati (SQL) + ricerca semantica (vector)
- Analisi multi-step: "confronta margini Q3 vs Q2 considerando resi e costi logistica"
- Aggregazioni complesse con contesto documentale
- Report automatici in linguaggio naturale

Tool aggiuntivi da costruire:
- `query_mongodb` — aggregazioni su ordini/spedizioni
- `query_mssql` — dati ERP
- `search_documents` — ricerca RAG su documenti economici
- `generate_report` — sintesi strutturata di analisi

---

## Architettura finale target

```
Chat UI (React widget)
        │
        ▼
FastAPI /chat endpoint
        │
        ▼
LangChain AgentExecutor
        │
   ┌────┴─────────────────┐
   │                      │
   ▼                      ▼
OMS API tools        RAG tools
(ordini/spedizioni)  (documenti economici)
   │                      │
   ▼                      ▼
OMS Next.js API      ChromaDB / MongoDB
        │
        ▼
   MongoDB (dati operativi)
```

---

## Note importanti per lo sviluppo

**Differenze Python vs JavaScript da tenere a mente:**
- I virtual environment (`venv`) sono l'equivalente di `node_modules` — sempre attivare prima di lavorare
- `pip install` = `npm install`, ma senza `package.json` automatico — usare `pip freeze > requirements.txt`
- Indentazione significativa: 4 spazi, mai tab
- `async/await` esiste ma non è il default come in Node — FastAPI lo gestisce automaticamente
- `dict` = oggetto JS, `list` = array JS, `None` = null

**Sicurezza:**
- Il service account JWT per l'agente deve essere read-only su OMS
- Mai loggare contenuto dei messaggi utente in chiaro (GDPR)
- Rate limiting su `/chat`: max 20 req/min per IP
- Ogni chiamata tool va loggata con session_id (audit trail)

**Costi LLM stimati con Claude Haiku:**
- Conversazione media servizio clienti: ~2.000 token totali = ~$0.004
- 1.000 conversazioni/mese = ~$4/mese
- Ampiamente nei free credits iniziali per tutta la fase di sviluppo

---

## Primo comando da eseguire adesso

```bash
mkdir oms-agent && cd oms-agent
python -m venv venv
source venv/bin/activate
pip install langchain langchain-anthropic python-dotenv fastapi uvicorn
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunofedev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brunofedev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
