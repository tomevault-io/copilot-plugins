## promtree

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**S13P31S307**: Samsung Electronics Production Technology Research Institute collaboration project
**Goal**: Extract material property information from TDS/MSDS PDF files with 99%+ accuracy, store in databases, and enable RAG-based chatbot for material property prediction and Q&A.

### Team Structure

6-person team (Team PromTree):
- **RAG Development**: 양현지, 이석철, 전해지, 최인혁
- **Parser Development**: 유승현, 조은경

### Technology Stack

**Backend:**
- **Python 3.12**: Primary language
- **FastAPI**: Web API server
- **LangChain**: RAG framework
- **MongoDB**: Document storage (markdown, chunks)
- **PostgreSQL**: Structured property data storage
- **Elasticsearch**: Full-text search and indexing
- **Neo4j**: Knowledge graph storage
- **Qdrant**: Vector database
- **Ollama**: Local LLM inference
- **Google Gemini**: Cloud LLM API
- **Docker**: Database containerization

**Frontend:**
- **React 19.1**: UI framework
- **TypeScript**: Type-safe JavaScript
- **Vite**: Build tool and dev server
- **Tailwind CSS**: Utility-first CSS framework
- **Lucide React**: Icon library

## Repository Setup

- **Remote**: https://lab.ssafy.com/s13-final/S13P31S307.git
- **Main Branch**: `master`
- **Development Branch**: `develop`
- **Commit Convention**: `[S13P31S307-<issue-number>] <Type>: <description>` (Korean descriptions supported)
- **Branch Convention**: `S13P31S307-<issue-number>-<description>`

## Environment Setup

### Package Manager

This project uses **uv** for Python dependency management. Install uv if not already installed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Docker Services

Start all database services from the project root:

```bash
# Start all services
docker-compose up -d

# Or start specific services
docker-compose up -d mongodb qdrant
```

Services:
- **MongoDB** (port 27017): Document storage (no web UI)
- **Qdrant** (ports 6333, 6334): Vector database
- **Elasticsearch** (port 9200): Full-text search with Korean support (Nori)
- **Neo4j** (ports 7474, 7687): Knowledge graph database

Environment variables required in root `.env`:
```
MONGO_INITDB_ROOT_USERNAME=promtree
MONGO_INITDB_ROOT_PASSWORD=ssafy13s307
MONGO_HOST=localhost
MONGO_PORT=27017

# Google Gemini API for RAG
GOOGLE_API_KEY=your_api_key_here
```

**Note**: PostgreSQL, Mongo Express, Kibana, and Adminer are no longer in the default docker-compose. The project primarily uses MongoDB with Beanie ODM.

### Virtual Environment Setup

The project uses **uv** for unified dependency management via `pyproject.toml`. Individual modules may still have their own venvs for compatibility.

**Primary application** (using uv):
```bash
# Sync all dependencies from pyproject.toml
uv sync

# Activate the virtual environment
source .venv/bin/activate  # macOS/Linux
```

**Legacy modules** (if working on specific extraction pipelines):

**db3** (TDS property extraction):
```bash
cd db3
source venv/bin/activate
pip install -r requirements.txt
```

**retriever** (RAG system - legacy):
```bash
cd retriever
source venv/bin/activate
pip install -r rag_requirements.txt
```

## Quick Start (Full Stack Development)

To run the complete application (frontend + backend + databases):

### 1. Start Database Services

```bash
# From project root
docker-compose up -d
```

Verify services:
- MongoDB: Connect via `mongosh mongodb://promtree:ssafy13s307@localhost:27017`
- Qdrant: http://localhost:6333/dashboard
- Neo4j: http://localhost:7474 (browser UI, credentials: neo4j/ssafy13s307)
- Elasticsearch: `curl -u elastic:ssafy13s307 http://localhost:9200`

### 2. Start Backend API

```bash
# Install dependencies with uv (first time)
uv sync

# Activate virtual environment
source .venv/bin/activate  # macOS/Linux

# Configure environment (first time)
# Edit .env and add MONGO credentials + GOOGLE_API_KEY

# Start server (from project root)
python main.py
```

Backend will be available at:
- API: http://localhost:8000
- Swagger Docs: http://localhost:8000/docs
- Health Check: http://localhost:8000/

### 3. Start Frontend

```bash
cd frontend

# Install dependencies (first time)
npm install

# Start dev server
npm run dev
```

Frontend will be available at: http://localhost:5173

### Troubleshooting

**Port already in use:**
```bash
# Kill backend (port 8000)
lsof -ti:8000 | xargs kill -9

# Kill frontend (port 5173)
lsof -ti:5173 | xargs kill -9
```

**Database connection issues:**
```bash
# From project root
docker-compose restart mongodb
docker-compose ps  # Check status
docker logs mongodb  # Check MongoDB logs
```

**RAG initialization timeout:**
- First message query may take 30-60 seconds to initialize FAISS embeddings
- Subsequent queries will be faster

## Project Architecture

### High-Level Data Flow

```
PDF Files → PromTree → Markdown (MongoDB)
                ↓
         Property Extraction (db3 for TDS, db1/db1_2py for MSDS)
                ↓
         PostgreSQL Storage
                ↓
    Hybrid RAG System (retriever) → Chatbot UI
         ↓              ↓
    Vector Search   Graph Search
    (FAISS/Qdrant)  (Neo4j)
         ↓              ↓
         └──── LLM ─────┘
```

### Application Stack

```
┌─────────────────────────────────────────────┐
│  Frontend (React + TypeScript + Vite)      │
│  - Chat Interface                           │
│  - Collection Management                    │
│  - Document Upload                          │
└─────────────────┬───────────────────────────┘
                  │ REST API
┌─────────────────▼───────────────────────────┐
│  Backend (FastAPI)                          │
│  - Chat/Message Management                  │
│  - Collection/Document APIs                 │
│  - RAG Service Integration                  │
└─────────────────┬───────────────────────────┘
                  │
       ┌──────────┴──────────┐
       ▼                     ▼
  MongoDB (Chats)      PostgreSQL (Users)
       │
       ▼
  RAG System (retriever/)
       │
  ┌────┴────┬────────┬──────────┐
  ▼         ▼        ▼          ▼
FAISS  Elasticsearch Neo4j  Qdrant
```

### Module Responsibilities

#### 0. `app/` - Modern FastAPI Backend (Primary)

**The main backend implementation using Beanie ODM for MongoDB.**

**Architecture**:
- **MongoDB with Beanie**: Async ODM (Object-Document Mapper) for type-safe database operations
- **FastAPI**: Modern async web framework
- **Integrated RAG**: Contains RAG system, chunking, and knowledge graph code
- **Integrated Parsers**: TDS/MSDS extraction pipelines (`core/tds.py`, `core/msds.py`)

**Key Structure**:
```
app/
├── app.py              # FastAPI application entry point
├── db.py               # MongoDB connection with Beanie initialization
├── routers/            # API route handlers
│   ├── users.py       # User authentication and management
│   ├── chats.py       # Chat and message endpoints
│   └── collections.py # Collection and document management
├── models/             # Beanie Document models (MongoDB schemas)
│   ├── user.py        # User model
│   ├── chat.py        # Chat model
│   ├── message.py     # Message model
│   └── collection.py  # KnowledgeCollection and CollectionDocument
├── schemas/            # Pydantic request/response schemas
├── services/           # Business logic layer
├── core/               # TDS/MSDS extraction pipelines
│   ├── tds.py         # TDS property extraction
│   └── msds.py        # MSDS property extraction
├── rag/                # RAG system components
│   ├── chunking.py    # Document chunking
│   ├── embedding.py   # Embedding generation
│   ├── neo4j_knowledge_graph.py  # Knowledge graph operations
│   └── parsing.py     # Document parsing utilities
├── promtree/           # PDF processing utilities
│   ├── parsing.py     # PDF to markdown conversion
│   ├── metadata.py    # Metadata extraction
│   └── unpivot.py     # Table unpivoting
└── utils/              # Utility functions
    └── auth.py        # Authentication helpers
```

**Development Commands**:
```bash
# Install dependencies
uv sync

# Activate environment
source .venv/bin/activate

# Start server (from project root)
python main.py

# Access API docs
# Swagger UI: http://localhost:8000/docs
# Health: http://localhost:8000/
```

**Key Features**:
- **Beanie ODM**: Type-safe async MongoDB operations with Pydantic models
- **Unified codebase**: RAG, extraction pipelines, and API in one place
- **Modern async**: Full async/await throughout the stack
- **Integrated processing**: PDF upload → parsing → extraction → RAG indexing in single workflow

**API Endpoints** (see Swagger docs at `/docs`):
- **Users**: `POST /users/register`, `POST /users/login`, `POST /users/logout`, `DELETE /users/delete`, `GET /users/info`, `PATCH /users/settings`
- **Chats**: `GET /chats`, `POST /chats`, `GET /chats/{chat_id}`, `POST /chats/{chat_id}` (send message), `PATCH /chats/{chat_id}`, `DELETE /chats/{chat_id}`
- **Collections**: `GET /collections`, `GET /collections/search?q={query}`, `POST /collections`, `PATCH /collections/{collection_id}`, `DELETE /collections/{collection_id}`
- **Documents**: `GET /collections/{collection_id}` (list docs), `POST /collections/{collection_id}` (upload), `DELETE /collections/{collection_id}/{document_id}`

**Important API Conventions**:
- All identifiers use **snake_case** (e.g., `chat_id`, `collection_id`, `document_id`, `created_at`, `updated_at`)
- Collection creation requires `type` field (`"msds"` or `"tds"`)
- Message contents use `contents` field (not `content` or `message`)

**Database Models** (Beanie Documents in `models/`):
```python
# User model with email authentication
class User(Document):
    email: str
    password: str  # hashed
    name: str
    createdAt: datetime

# Chat with messages
class Chat(Document):
    userId: str
    title: str
    createdAt: datetime

# Message with RAG context
class Message(Document):
    chatId: str
    role: str  # "user" or "assistant"
    content: str
    createdAt: datetime

# Knowledge collection for RAG
class KnowledgeCollection(Document):
    userId: str
    name: str
    description: str
    createdAt: datetime
```

#### 1. `frontend/` - React Web Application

Modern React web UI for chatting with RAG system and managing document collections.

**Key Features**:
- ChatGPT-like interface with sidebar navigation
- Multi-collection document management
- File upload with progress tracking
- Dark/light theme support
- Toast notifications
- User authentication (login/signup)

**Tech Stack**:
- React 19.1 with TypeScript
- Vite for dev server and builds
- Tailwind CSS for styling
- Custom hooks for state management (useToast, useUpload)

**Project Structure**:
```
frontend/
├── src/
│   ├── components/          # React components
│   │   ├── Chat.tsx        # Main chat interface
│   │   ├── Sidebar.tsx     # Navigation sidebar with chat history
│   │   ├── Collections.tsx # Collection list view
│   │   ├── CollectionDetail.tsx  # Single collection detail
│   │   ├── Login.tsx       # Login/signup form
│   │   ├── LandingPage.tsx # Landing page for logged-out users
│   │   ├── ThemeToggle.tsx # Dark mode toggle
│   │   ├── Toast.tsx       # Toast notification component
│   │   └── UploadProgress.tsx  # File upload progress tracker
│   ├── hooks/              # Custom React hooks
│   │   ├── useToast.ts     # Toast notification hook
│   │   └── useUpload.ts    # File upload state hook
│   ├── lib/
│   │   ├── api.ts          # API client (REST calls to backend)
│   │   └── utils.ts        # Utility functions
│   ├── App.tsx             # Main app component (routing logic)
│   └── main.tsx            # Entry point
├── package.json
├── vite.config.ts
└── tailwind.config.js
```

**Development Commands**:
```bash
cd frontend

# Install dependencies
npm install

# Start dev server (http://localhost:5173)
npm run dev

# Build for production
npm run build

# Lint
npm run lint

# Preview production build
npm run preview
```

**Key Components**:
- **App.tsx**: Client-side routing logic (collections, collection-detail, chat pages)
- **Chat.tsx**: Chat interface with message history and RAG responses
- **Sidebar.tsx**: Navigation with chat history list, new chat button
- **Collections.tsx**: Grid view of document collections
- **CollectionDetail.tsx**: Document list within a collection, upload files
- **lib/api.ts**: Centralized API client for all backend calls

**State Management**:
- User state stored in localStorage
- Chat/collection data fetched from backend API
- Custom hooks for toast notifications and upload progress

#### 2. ~~`backend/`~~ - Removed (Legacy)

**⚠️ This directory has been removed.** The actual backend is `app/` (see above).

#### 3. `retriever/` - Hybrid RAG System

Advanced RAG system combining vector search, graph search, and Elasticsearch.

**Architecture**:
- **Vector Search**: FAISS, Qdrant, Weaviate support
- **Graph Search**: Neo4j knowledge graph (ApeRAG/LightRAG approach)
- **Keyword Search**: Elasticsearch with Korean analyzer (Nori)
- **LLM**: Google Gemini 2.5 Flash

**Key Files**:
- `rag_system.py`: Main RAG class with FAISS retrieval
- `embedding.py`: Sentence transformer embeddings (legacy)
- `embedding_model/`: Modular embedding system (OpenAI, sentence-transformers)
- `chunker/markdown_chunker.py`: Markdown chunking with table handling
- `chunker/html2row.py`: HTML table unpivoting
- `indexer/elasticsearch_indexer.py`: Elasticsearch indexing and BM25 search
- `lightrag/lightrag_hybrid_rag.py`: Hybrid RAG orchestrator
- `lightrag/lightrag_entity_extractor.py`: Knowledge graph entity extraction
- `lightrag/lightrag_graph_storage.py`: Neo4j graph storage
- `vector_store/`: Vector database abstraction (Qdrant, Weaviate)
- `knowledge_graph/neo4j_knowledge_graph.py`: Neo4j operations

**Usage - Basic RAG**:
```bash
cd retriever
source venv/bin/activate

# Process documents into chunks
python chunker/markdown_chunker.py

# Initialize embeddings and FAISS index
python embedding.py

# Run RAG system
python rag_system.py
```

**Usage - Hybrid RAG (Vector + Graph)**:
```python
from retriever.lightrag.lightrag_hybrid_rag import HybridRAG
from retriever.rag_system import RAGSystem
from retriever.lightrag.lightrag_graph_storage import GraphStorage

# Initialize components
vector_rag = RAGSystem()
graph_storage = GraphStorage()

# Create hybrid RAG
hybrid_rag = HybridRAG(vector_rag=vector_rag, graph_storage=graph_storage)

# Query
answer = hybrid_rag.query("What is the Tg of polymer X?")
```

**MongoDB Chunks Schema**:
```python
{
    "vector_id": int,           # FAISS/Qdrant index ID
    "content": str,             # Chunk text
    "source_file_name": str,    # Original PDF name
    "page_num": int,            # Page number
    "chunk_index": int          # Chunk position in page
}
```

**Knowledge Graph Schema (Neo4j)**:
- **Nodes**: Entities extracted from documents (materials, properties, companies, etc.)
- **Relationships**: Connections between entities (HAS_PROPERTY, MANUFACTURED_BY, etc.)
- **Properties**: Entity attributes (name, type, source_document, etc.)

#### 4. `promtree/` - PDF to Markdown Conversion

Python package for converting TDS/MSDS PDFs to structured Markdown format.

**Key Features**:
- OCR-based layout detection using Surya
- Preserves document structure (sections, images, tables)
- Outputs clean Markdown for property extraction

**Usage**:
```bash
cd promtree
promtree path/to/document.pdf --output-md output.md
```

**Output Structure**:
```
document/
├── images/          # Extracted images
├── layouts/         # Layout detection results
├── outputs/         # PNG images of each page
├── output.txt       # Text extraction
└── output.md        # Final Markdown
```

#### 5. `db3/` - TDS Material Property Extraction Pipeline

Extracts structured material properties from TDS Markdown documents using regex + LLM hybrid approach.

**Architecture**:
- **MongoDB**: Stores markdown documents (`markdown_collection`) and temporary extraction results (`temp_extraction`)
- **PostgreSQL**: Stores structured property data (`tds_properties` table with dynamic columns)

**Key Files**:
- `core/db_connection.py`: Database connection management
- `core/pipeline_smart.py`: Main extraction pipeline (regex + Ollama LLM)
- `core/extractor.py`: Regex-based property extraction (23 property types)
- `core/llm_agent_ollama.py`: Ollama-based LLM extraction fallback
- `core/llm_agent_langchain.py`: LangChain-based LLM agent (alternative)
- `core/properties_dict.py`: Property type definitions
- `core/create_pg_tables.py`: PostgreSQL schema management
- `utils/mock_data.py`: Mock data generation for testing
- `utils/verify_results.py`: Extraction result verification

**Running the Pipeline**:
```bash
cd db3/core
source ../venv/bin/activate

# Create PostgreSQL tables (first time only)
python create_pg_tables.py

# Run extraction pipeline
python pipeline_smart.py
```

**Extraction Strategy**:
- **Regex-first**: Fast pattern matching for common properties (Tg, Tm, YS, YM, etc.)
- **LLM fallback**: Ollama (Qwen2.5:7b) or LangChain for complex documents
- **Hybrid merge**: Combines results, deduplicates, regex-preferred
- **Dynamic schema**: PostgreSQL columns auto-created for new properties

**Property Types Supported** (23 types):
```
Tg, Tm, Td, DC, Eg, YS, YM, BS, Tensile_Strength, Elongation_Rate,
Hardness, HDT, Thermal_Conductivity, Density, Viscosity,
Thixotropic_index, He_permeability, H2_permeability, O2_permeability,
N2_permeability, CO2_permeability, CH4_permeability
```

**PostgreSQL Schema**:
```sql
CREATE TABLE tds_properties (
    id SERIAL PRIMARY KEY,
    document_id VARCHAR(100) UNIQUE NOT NULL,
    -- Dynamic columns: {property}_value FLOAT, {property}_unit VARCHAR(50)
    tg_value FLOAT, tg_unit VARCHAR(50),
    tm_value FLOAT, tm_unit VARCHAR(50),
    -- ... (columns added dynamically as properties are discovered)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 6. `common/db1/` - MSDS Extraction Pipeline (Version 1)

Extracts product information and ingredient composition from MSDS documents.

**Architecture**:
- LLM-driven section identification and slicing
- Structured extraction of Section 1 (product info) and Section 2/3 (ingredients)
- PostgreSQL storage with `products` and `ingredients` tables

**Key Files**:
- `db_1_py/msds_pipeline.py`: Main pipeline orchestrator
- `db_1_py/msds_db_section1_pipeline.py`: Section 1 extraction
- `db_1_py/msds_db_ingredients_pipeline.py`: Ingredient extraction
- `db_1_py/msds_db_create_pg_tables.py`: PostgreSQL schema management
- `utils/verify.py`: Extraction verification with PDF report generation

**Running the Pipeline**:
```bash
cd common/db1/db_1_py
source ../../venv/bin/activate

# Run pipeline (configure document_id in msds_pipeline.py)
python msds_pipeline.py

# Verify results with PDF report
cd ../utils
python verify.py
```

**Extraction Workflow**:
1. Load MSDS markdown from MongoDB (`msds_markdown_collection`)
2. Extract Section 1: product name, company name
3. Extract Section 2/3: ingredient list with CAS numbers, concentrations
4. Normalize and postprocess extracted data
5. Store in PostgreSQL (`products`, `ingredients` tables)

#### 7. `common/db1_2py/` - MSDS Extraction Pipeline (Version 2)

Alternative MSDS extraction pipeline using markdown-it for header parsing.

**Architecture**:
- markdown-it parser for section structure extraction
- LLM-based header identification
- Flexible regex patterns for section slicing
- Integrated extraction of product + ingredient info

**Key Files**:
- `msds_ETL_pipeline.py`: Main ETL pipeline
- `msds_extract_sections_with_markdown_it.py`: Header extraction
- `msds_get_key_section_headers_with_llm.py`: LLM section identification
- `msds_extract_integrated_sds_info_LLM.py`: Integrated LLM extraction
- `msds_db_create_pg_tables.py`: PostgreSQL schema management

**Running the Pipeline**:
```bash
cd common/db1_2py
source ../venv/bin/activate

# Run on markdown text
python msds_ETL_pipeline.py  # or import main() function
```

**Pipeline Steps**:
1. Extract headers with markdown-it parser
2. LLM identifies key sections (identification, composition)
3. Find next section headers for boundary detection
4. Preprocess markdown (remove images, page markers)
5. Slice sections with flexible regex patterns
6. LLM extracts product + ingredient info
7. Save to PostgreSQL

#### 8. `cleanup/` - Legacy and Experimental Code

Contains older implementations and experimental parsing methods:
- `cleanup/demo/`: Original PDF parsing demo (pdfplumber + PyMuPDF)
- `cleanup/parsing/`: Alternative parsing approaches (PP-DocLayout, advanced PDF to MD)

**Note**: Active development has moved to `db3/`, `retriever/`, `promtree/`, and `common/db1/`. The `cleanup/` directory is kept for reference.

### Database Conventions

**MongoDB** (`promtree` database - primary for `app/` backend):
- **Beanie Collections** (defined in `app/models/`):
  - `user`: User accounts with authentication
  - `chat`: Chat sessions
  - `message`: Chat messages with RAG responses
  - `knowledgecollection`: Document collections for RAG
  - `collectiondocument`: Individual documents within collections

**MongoDB** (`s307_db` database - legacy extraction pipelines):
- `markdown_collection`: TDS markdown documents
- `msds_markdown_collection`: MSDS markdown documents
- `temp_extraction`: Temporary property extraction results
- `chunks`: RAG vector search chunks (older approach)

**MongoDB** (`chunk_db` database):
- `chunk_collection`: Processed chunks for RAG system

**PostgreSQL** (not in default docker-compose, only in legacy `db3` module):
- `tds_properties`: TDS extracted material properties (dynamic schema)
- `products`: MSDS product information
- `ingredients`: MSDS ingredient composition

**Elasticsearch**:
- Index: Document chunks with Korean text support (Nori analyzer)
- BM25 keyword search for hybrid retrieval

**Neo4j**:
- Knowledge graph: Entities and relationships extracted from documents
- Cypher queries for graph-based retrieval

**Qdrant**:
- Vector database: Alternative to FAISS for scalable vector search
- Collection-based organization

## Common Development Tasks

### Working with the Modern Stack (app/ + frontend/)

**Adding a New API Endpoint:**

1. **Define Beanie Model** (if new data structure) in `app/models/`:
```python
from beanie import Document
from datetime import datetime

class NewModel(Document):
    userId: str
    name: str
    createdAt: datetime = datetime.now()

    class Settings:
        name = "newmodel"  # MongoDB collection name
```

2. **Add to Beanie initialization** in `app/db.py`:
```python
await init_beanie(
    database=client.promtree,
    document_models=[User, Chat, Message, KnowledgeCollection,
                     CollectionDocument, NewModel]  # Add here
)
```

3. **Create request/response schemas** in `app/schemas/`:
```python
from pydantic import BaseModel

class CreateNewModelRequest(BaseModel):
    name: str
    userId: str

class NewModelResponse(BaseModel):
    id: str
    name: str
    createdAt: str
```

4. **Define endpoint** in `app/routers/` (e.g., `new_models.py`):
```python
from fastapi import APIRouter, HTTPException
from app.models.new_model import NewModel
from app.schemas.new_model import CreateNewModelRequest

router = APIRouter()

@router.post("/")
async def create_new_model(request: CreateNewModelRequest):
    """Create new model"""
    model = NewModel(
        userId=request.userId,
        name=request.name
    )
    await model.insert()

    return {"success": True, "data": model.dict()}
```

5. **Register router** in `app/app.py`:
```python
from app.routers.new_models import router as new_model_router

app.include_router(new_model_router, prefix="/newmodels", tags=["NewModel"])
```

6. **Update frontend API client** in `frontend/src/lib/api.ts`:
```typescript
export async function createNewModel(data: CreateNewModelData) {
  return apiRequest<NewModel>('/newmodels', {
    method: 'POST',
    body: JSON.stringify(data),
  });
}
```

**Testing the API:**
- Swagger UI: http://localhost:8000/docs (auto-updates when routes change)
- Health check: `curl http://localhost:8000/`
- MongoDB inspection: `mongosh mongodb://promtree:ssafy13s307@localhost:27017/promtree`

**Development Tips:**

**Backend (app/)**:
- Uses `uvicorn` with `reload=False` in `main.py` - manually restart to see changes
- Beanie provides async MongoDB operations: `await Model.find_one()`, `await Model.insert()`
- Check logs in terminal where `python main.py` is running
- All models automatically get ObjectId as `id` field

**Frontend**:
- Vite hot reload works automatically
- Check browser console for errors
- React DevTools for component inspection
- Toast notifications display API errors

**Debugging Beanie Queries**:
```python
# Find documents
user = await User.find_one(User.email == "test@example.com")

# Find with multiple conditions
chats = await Chat.find(
    Chat.userId == user_id,
    Chat.createdAt >= start_date
).to_list()

# Update document
user.name = "New Name"
await user.save()

# Delete document
await user.delete()
```

### Adding a New TDS Property Type

1. Add pattern to `db3/core/extractor.py`:
```python
PATTERNS = {
    'New_Property': [
        (r'(?:New Property|NP)[\s:]+(\d+\.?\d*)\s*(unit)', 1, 2),
    ]
}
```

2. Add to `db3/core/properties_dict.py` if needed
3. Run pipeline - PostgreSQL columns auto-created

### Testing TDS Property Extraction

```bash
cd db3/core
source ../venv/bin/activate

# Generate mock data
python ../utils/mock_data.py

# Run pipeline on mock data
python pipeline_smart.py  # Filters: document_id REGEX '^MOCK_'

# Verify results
python ../utils/verify_results.py
```

### Testing MSDS Extraction

```bash
cd common/db1/db_1_py
source ../../venv/bin/activate

# Configure document_id in msds_pipeline.py (e.g., 'MOCK_MSDS_007')
python msds_pipeline.py

# Generate verification PDF report
cd ../utils
python verify.py
```

### Working with Ollama LLM

Start Ollama server (GPU environment):
```bash
# GPU selection (see common/GPU.md)
CUDA_VISIBLE_DEVICES=7 nohup ollama serve > ollama.log 2>&1 &

# Pull models
ollama pull qwen2.5:7b
ollama pull gpt-oss:20b  # Used in db1_2py pipeline
```

Enable LLM in db3 pipeline:
```python
# db3/core/pipeline_smart.py, line 309
use_llm_fallback=True  # Enable LLM fallback
```

### Running Hybrid RAG System

```bash
cd retriever
source venv/bin/activate

# 1. Chunk documents
python chunker/markdown_chunker.py

# 2. Build knowledge graph
python lightrag/lightrag_build_graph.py

# 3. Initialize vector embeddings
python embedding.py

# 4. Run hybrid RAG demo
python lightrag/lightrag_demo.py
```

### Working with Elasticsearch

```bash
# Start Elasticsearch and Kibana
cd common
docker-compose up -d elasticsearch kibana

# Wait for services to be ready (check logs)
docker logs -f elasticsearch

# Index documents
cd ../retriever/indexer
source ../venv/bin/activate
python elasticsearch_indexer.py
```

Access Kibana at http://localhost:5601 (username: elastic, password: from .env)

## Development Workflow

### Starting a New Task

1. Pull latest `develop`:
```bash
git checkout develop
git pull origin develop
```

2. Create feature branch:
```bash
git checkout -b S13P31S307-<issue-number>-<description>
```

3. Sync dependencies and activate environment:
```bash
# For main app/ backend
uv sync
source .venv/bin/activate

# OR for legacy modules
cd db3  # or retriever, etc.
source venv/bin/activate
```

4. Start development:
```bash
# Start databases
docker-compose up -d

# Start backend
python main.py  # for app/ backend

# Start frontend (separate terminal)
cd frontend && npm run dev
```

### Before Committing

1. Ensure `.env` files are not tracked (already in `.gitignore`)
2. Test your changes with Docker services running
3. Follow commit convention: `[S13P31S307-<issue>] <Type>: <description>`

### Creating Merge Requests

- Target branch: `develop`
- Provide clear description of changes
- Reference related issue number

## Performance Notes

### LLM Extraction (Ollama)

- **Without LLM** (`use_llm_fallback=False`): ~1 second for 5 documents
- **With LLM** (`use_llm_fallback=True`): ~30-60 seconds per document (GPU-dependent)
- Use regex-only mode for rapid iteration/testing
- Enable LLM for production accuracy

### RAG Search Performance

- **FAISS Search**: Default `top_k=5` for context retrieval
- **Elasticsearch BM25**: Fast keyword search, good for Korean text
- **Graph Search**: Slower but provides entity-relationship context
- **Hybrid approach**: Combine all three for best results

### Docker Resource Management

Monitor container resources:
```bash
docker stats
```

Stop services when not in use:
```bash
# From project root
docker-compose down
```

Stop specific services:
```bash
docker-compose stop es neo4j  # Heavy services
docker-compose stop qdrant     # If not using vector search
```

Restart specific services:
```bash
docker-compose restart mongodb
docker-compose restart es
```

### GPU Memory Management

Check GPU usage (if available):
```bash
nvidia-smi
```

Select specific GPU for Ollama:
```bash
CUDA_VISIBLE_DEVICES=7 ollama serve
```

## Key Architectural Decisions

### Why Two MSDS Pipelines (db1 vs db1_2py)?

- **db1**: Original pipeline with separate Section 1 and Section 2/3 processing
- **db1_2py**: Improved pipeline with markdown-it parser and integrated extraction
- Both are maintained for comparison and fallback

### Vector Database Choice

- **FAISS**: Fast, in-memory, good for prototyping (current default)
- **Qdrant**: Scalable, persistent, better for production
- **Weaviate**: Alternative with built-in hybrid search

### LLM Strategy

- **Local (Ollama)**: Privacy, cost-effective, requires GPU
- **Cloud (Google Gemini)**: Fast, accurate, pay-per-use
- **Hybrid**: Use Ollama for extraction, Gemini for RAG

### Chunking Strategy

- **Recursive text splitter**: Preserves semantic boundaries
- **Table handling**: Unpivot HTML tables to rows for better retrieval
- **Page awareness**: Maintain page numbers for source tracking

## Current Implementation Status

### ✅ Completed Features

**Infrastructure:**
- Docker Compose setup for all databases (MongoDB, Qdrant, Elasticsearch, Neo4j)
- UV package manager integration with `pyproject.toml`
- Unified dependency management across modules

**Backend (`app/`):**
- FastAPI with Beanie ODM for MongoDB
- User authentication (signup/login with password hashing)
- Chat and message management APIs
- Collection and document management APIs
- Integrated RAG, TDS/MSDS extraction pipelines in single codebase
- CORS configuration for frontend
- Swagger API documentation
- Health check endpoint

**Frontend:**
- Modern React 19.1 with TypeScript
- Chat interface with message history
- Collection management (create, list, view, delete)
- Document upload with progress tracking
- Login/signup authentication
- Dark/light theme support
- Toast notifications
- Responsive design with Tailwind CSS

**RAG System (`app/rag/`):**
- Document chunking with markdown support
- Embedding generation
- Neo4j knowledge graph integration
- Elasticsearch with Korean (Nori) analyzer
- Vector search capabilities

**Data Extraction:**
- TDS property extraction (regex + LLM hybrid in `db3/`)
- MSDS property extraction (two pipeline versions in `common/db1/` and `common/db1_2py/`)
- PDF to Markdown conversion (`app/promtree/`)
- Integrated extraction pipelines (`app/core/tds.py`, `app/core/msds.py`)

### 🚧 In Progress / TODO

**Backend Integration:**
- [ ] Complete RAG query integration in chat endpoints
- [ ] Document upload → parsing → extraction → indexing pipeline
- [ ] JWT token-based authentication (currently using simple hash)
- [ ] File upload to Qdrant for vector search
- [ ] Multi-collection RAG filtering

**Frontend:**
- [ ] Real-time chat streaming (SSE or WebSocket)
- [ ] Advanced search filters (date range, collection selection)
- [ ] User settings page
- [ ] Model selection dropdown (Gemini, Ollama, etc.)
- [ ] RAG source citation display

**RAG:**
- [ ] Switch from FAISS to Qdrant for production
- [ ] Hybrid search (vector + keyword + graph) weighting
- [ ] Cache layer for frequently asked queries
- [ ] Chunk relevance scoring improvements

**Deployment:**
- [ ] Docker Compose for full stack (backend + frontend)
- [ ] Production environment configuration
- [ ] CI/CD pipeline
- [ ] Monitoring and logging
- [ ] API rate limiting and authentication middleware

**Documentation:**
- [ ] API integration guide for frontend developers
- [ ] RAG pipeline architecture diagram
- [ ] Deployment guide for production environment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yoo-SeungHyeon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
