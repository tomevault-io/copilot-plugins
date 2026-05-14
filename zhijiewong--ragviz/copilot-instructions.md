## ragviz

> This file provides comprehensive guidance to Claude Code when working with the RAGViz platform - a Graph RAG Visualization Platform that combines knowledge graphs with vector retrieval for local LLM applications.

# CLAUDE.md - RAGViz Development Guide

This file provides comprehensive guidance to Claude Code when working with the RAGViz platform - a Graph RAG Visualization Platform that combines knowledge graphs with vector retrieval for local LLM applications.

## 🎯 Project Overview

**RAGViz** is a developer-oriented Graph Retrieval-Augmented Generation (Graph RAG) visualization and parameter tuning platform. It combines knowledge graphs with vector retrieval to provide contextual support for local large language models, enabling interactive document uploads, knowledge graph construction, and real-time parameter adjustments while visualizing their impact on answers and subgraph structures.

### Core Philosophy

- **KISS (Keep It Simple, Stupid)**: Choose straightforward solutions over complex ones
- **YAGNI (You Aren't Gonna Need It)**: Implement features only when needed
- **Developer-First**: Built for research and data science teams with debugging visualization
- **Local-First**: Privacy-focused with local LLM integration (Ollama/llama.cpp)

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │   Database      │
│   (Next.js)     │◄──►│   (FastAPI)     │◄──►│   (Neo4j)       │
│                 │    │                 │    │                 │
│ • File Upload   │    │ • Document Proc │    │ • Knowledge     │
│ • Chat UI       │    │ • Entity Extract│    │   Graph         │
│ • Graph Viz     │    │ • Vector Search │    │ • Vector Index  │
│ • Parameters    │    │ • LLM Inference │    │ • HNSW Search   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                ▲
                                │
                       ┌─────────────────┐
                       │   Local LLM     │
                       │ (Ollama/llama.cpp)│
                       └─────────────────┘
```

**Data Flow**:
1. User uploads documents → Frontend → Backend
2. Backend processes: chunking, entity extraction, embedding → Neo4j
3. User queries via chat → Backend retrieves via vector search → LLM generates response
4. Frontend displays answer + highlights retrieved subgraph

## 📁 Project Structure

```
ragviz/
├── frontend/                 # Next.js React Application
│   ├── src/
│   │   ├── app/             # Next.js App Router pages
│   │   ├── components/      # React components
│   │   │   ├── graph/       # Graph visualization components
│   │   │   ├── ui/          # Reusable UI components (shadcn/ui)
│   │   │   └── chat/        # Chat interface components
│   │   └── lib/             # Utilities and API clients
│   ├── package.json         # Node.js dependencies
│   ├── next.config.ts       # Next.js configuration
│   └── CLAUDE.md           # Frontend-specific guide
├── backend/                 # FastAPI Python Application
│   ├── app/
│   │   ├── api/            # API route handlers
│   │   ├── core/           # Configuration and settings
│   │   ├── services/       # Business logic (Neo4j, LLM, etc.)
│   │   └── models/         # Pydantic data models
│   ├── requirements.txt    # Python dependencies
│   ├── venv/              # Virtual environment
│   └── CLAUDE.md          # Backend-specific guide (detailed)
├── docker-compose.yml      # Container orchestration
├── .env.example           # Environment variables template
├── README.md              # Project documentation
├── PROJECT.md             # Detailed technical overview (Chinese)
└── CLAUDE.md              # This comprehensive guide
```

## 🛠️ Development Environment Setup

### Prerequisites

- **Node.js** 18+ (for frontend)
- **Python** 3.9+ (for backend)
- **Neo4j** 5.15+ (database) - Choose one:
  - **Recommended**: [Neo4j Aura Free](https://neo4j.com/cloud/aura-free/) (cloud, zero setup)
  - Local: Docker, Neo4j Desktop, or self-hosted
- **Ollama** (for local LLM)
- **Docker** & Docker Compose (optional, for all-in-one setup)

### Quick Start

#### Option 1: Cloud Setup with Neo4j Aura (Recommended) ☁️

Best for beginners - no local database installation needed!

```bash
# 1. Get Neo4j Aura Free (2 minutes)
# Visit: https://neo4j.com/cloud/aura-free/
# Create free instance, save credentials

# 2. Clone and setup
git clone <repository-url>
cd ragviz

# 3. Environment setup
cp .env.example .env
# Edit .env:
# NEO4J_URI=neo4j+s://xxxxx.databases.neo4j.io  # Your Aura URI
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=your_aura_password

# 4. Install Ollama
# Visit https://ollama.ai/download
ollama pull qwen2.5:7b-instruct

# 5. Run Backend
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# 6. Run Frontend (new terminal)
cd frontend
npm install
npm run dev  # Uses Turbopack for fast development (2s vs 30s+)
```

#### Option 2: Docker (All-in-One Local Setup) 🐳

```bash
# 1. Clone and setup
git clone <repository-url>
cd ragviz

# 2. Environment setup
cp .env.example .env
# Edit .env with local settings (already configured for Docker)

# 3. Start everything
docker-compose up -d

# Access: http://localhost:3000
```

### Environment Variables

```bash
# Backend (.env)
# Option 1: Neo4j Aura (Recommended)
NEO4J_URI=neo4j+s://xxxxx.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_aura_password

# Option 2: Local Neo4j
# NEO4J_URI=bolt://localhost:7687
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=your_password

# LLM Configuration
OLLAMA_BASE_URL=http://localhost:11434
OPENAI_API_KEY=your_key_if_needed  # Optional

# CORS (adjust for your setup)
ALLOWED_ORIGINS=http://localhost:3000

# Frontend (.env.local)
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## 🎨 Frontend Development (Next.js/React)

### Tech Stack

- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS v4
- **UI Components**: shadcn/ui + Radix UI
- **Graph Visualization**: Cytoscape.js
- **State Management**: React Query (@tanstack/react-query)
- **HTTP Client**: Axios
- **Animations**: Framer Motion

### Code Standards

- **Line Length**: 80 characters max
- **Component Structure**: Functional components with hooks
- **Type Safety**: Strict TypeScript, no `any` types
- **Naming**: PascalCase for components, camelCase for functions/variables
- **File Organization**: Group by feature, not by type

### Component Architecture

```typescript
// Component Structure Example
interface ComponentProps {
  required: string;
  optional?: number;
}

export function ComponentName({ required, optional = 0 }: ComponentProps) {
  // 1. Hooks (useState, useEffect, custom hooks)
  const [state, setState] = useState<Type>(initialValue);
  
  // 2. Event handlers
  const handleClick = useCallback(() => {
    // handler logic
  }, [dependencies]);
  
  // 3. Effects
  useEffect(() => {
    // effect logic
  }, [dependencies]);
  
  // 4. Early returns
  if (loading) return <LoadingSpinner />;
  
  // 5. Render
  return (
    <div className="container">
      {/* JSX */}
    </div>
  );
}
```

### Key Components

```
src/components/
├── graph/
│   ├── GraphContainer.tsx      # Main graph wrapper
│   ├── InteractiveGraph.tsx    # Cytoscape integration
│   └── GraphControls.tsx       # Graph manipulation controls
├── chat/
│   ├── ChatInterface.tsx       # Main chat UI
│   ├── MessageList.tsx         # Message display
│   └── QueryInput.tsx          # User input form
├── ui/                         # shadcn/ui components
│   ├── button.tsx
│   ├── input.tsx
│   ├── card.tsx
│   └── dialog.tsx
└── upload/
    └── FileUpload.tsx          # Document upload
```

### Frontend Commands

```bash
# Development
npm run dev          # Start dev server with Turbopack (fast!)
npm run dev:webpack  # Start dev server with webpack (fallback)
npm run build        # Production build
npm run start        # Start production server
npm run lint         # ESLint checking
npm run type-check   # TypeScript checking
```

## 🔧 Backend Development (FastAPI/Python)

### Tech Stack

- **Framework**: FastAPI 0.104+
- **Language**: Python 3.9+
- **Database**: Neo4j 5.15+ with HNSW vector indexing
- **LLM Integration**: Ollama, OpenAI (fallback)
- **Document Processing**: PyPDF, python-docx
- **Embeddings**: sentence-transformers, HuggingFace
- **Async**: uvicorn, asyncio

### Code Standards (Detailed in backend/CLAUDE.md)

- **Line Length**: 100 characters max (ruff configured)
- **Type Hints**: Required for all functions
- **Docstrings**: Google-style for public functions
- **Error Handling**: Custom exceptions with structured logging
- **File Limits**: 500 lines max per file, 50 lines per function

### Virtual Environment

**CRITICAL**: Always use the `venv` virtual environment for Python commands:

```bash
# Activate virtual environment
cd backend
source venv/bin/activate  # Linux/Mac
# OR
venv\Scripts\activate     # Windows

# Run Python commands within venv
python app/main.py
pytest tests/
pip install package_name
```

### API Architecture

```python
# FastAPI Structure Example
from fastapi import APIRouter, HTTPException, Depends
from pydantic import BaseModel

router = APIRouter(prefix="/api/v1", tags=["documents"])

class DocumentUpload(BaseModel):
    filename: str
    content_type: str
    size: int

@router.post("/documents/upload")
async def upload_document(
    file: UploadFile = File(...),
    service: DocumentService = Depends(get_document_service)
) -> DocumentResponse:
    """
    Upload and process document for knowledge graph construction.
    
    Args:
        file: Uploaded document file
        service: Injected document processing service
        
    Returns:
        Processing status and document metadata
        
    Raises:
        HTTPException: If file type not supported
    """
    try:
        result = await service.process_document(file)
        return DocumentResponse(
            id=result.id,
            status="processing",
            graph_nodes=result.node_count
        )
    except UnsupportedFileType as e:
        raise HTTPException(status_code=400, detail=str(e))
```

### Core Services

```
backend/app/services/
├── document_service.py      # File upload & processing
├── graph_service.py         # Neo4j operations
├── embedding_service.py     # Vector embeddings
├── retrieval_service.py     # RAG retrieval logic
├── llm_service.py          # Local LLM integration
└── chat_service.py         # Chat session management
```

### Backend Commands

```bash
# Always run within activated venv
uvicorn app.main:app --reload --port 8000  # Dev server
pytest tests/ -v                           # Run tests
python -m app.main                          # Direct run
ruff check . --fix                         # Lint and fix
ruff format .                              # Format code
mypy app/                                  # Type checking
```

## 🗄️ Database (Neo4j) Integration

### Schema Design

```cypher
// Node Types
(:Document {id, title, upload_date, file_path})
(:Entity {id, name, type, description, embedding})
(:Chunk {id, text, position, document_id, embedding})

// Relationship Types
(:Document)-[:CONTAINS]->(:Chunk)
(:Chunk)-[:MENTIONS]->(:Entity)
(:Entity)-[:RELATED_TO]->(:Entity)
(:Entity)-[:EXTRACTED_FROM]->(:Document)
```

### Vector Indexing

```cypher
// Create vector indexes for similarity search
CREATE VECTOR INDEX chunk_embeddings FOR (c:Chunk) ON c.embedding
OPTIONS {indexConfig: {
  vector.dimensions: 384,
  vector.similarity_function: 'cosine'
}};

CREATE VECTOR INDEX entity_embeddings FOR (e:Entity) ON e.embedding
OPTIONS {indexConfig: {
  vector.dimensions: 384,
  vector.similarity_function: 'cosine'
}};
```

### Query Examples

```cypher
// Vector similarity search for RAG retrieval
CALL db.index.vector.queryNodes('chunk_embeddings', 10, $query_embedding)
YIELD node, score
MATCH (node)-[:MENTIONS]->(entity:Entity)
RETURN node.text, collect(entity.name) as entities, score
ORDER BY score DESC;

// Subgraph expansion for context
MATCH (chunk:Chunk)-[:MENTIONS]->(entity:Entity)
WHERE chunk.id IN $retrieved_chunks
MATCH (entity)-[:RELATED_TO*1..2]-(related:Entity)
RETURN entity, related, collect(chunk) as source_chunks;
```

## 🤖 Local LLM Integration

### Supported Models

```python
# Ollama Integration
OLLAMA_MODELS = {
    "llama3.1": "llama3.1:8b-instruct",
    "mistral": "mistral:7b-instruct", 
    "qwen2.5": "qwen2.5:7b-instruct",  # Current default
    "codellama": "codellama:7b-instruct"
}

# Embedding Models
EMBEDDING_MODELS = {
    "default": "sentence-transformers/all-MiniLM-L6-v2",
    "multilingual": "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2",
    "large": "sentence-transformers/all-mpnet-base-v2"
}
```

### RAG Pipeline

```python
async def rag_query(query: str, params: QueryParams) -> RAGResponse:
    """Complete RAG pipeline execution."""
    # 1. Embed query
    query_embedding = await embedding_service.embed_text(query)
    
    # 2. Vector retrieval from Neo4j
    chunks = await graph_service.vector_search(
        embedding=query_embedding,
        top_k=params.top_k,
        similarity_threshold=params.threshold
    )
    
    # 3. Subgraph expansion
    context_graph = await graph_service.expand_subgraph(
        chunk_ids=[c.id for c in chunks],
        max_hops=params.max_hops
    )
    
    # 4. Context preparation
    context = format_context(chunks, context_graph)
    
    # 5. LLM generation
    response = await llm_service.generate(
        prompt=build_prompt(query, context),
        temperature=params.temperature,
        max_tokens=params.max_tokens
    )
    
    return RAGResponse(
        answer=response.text,
        sources=chunks,
        subgraph=context_graph,
        metadata=response.metadata
    )
```

## 📋 Development Workflow

### Git Strategy

- **Main Branch**: `main` (production-ready)
- **Development**: Feature branches (`feature/*`, `fix/*`, `docs/*`)
- **Naming**: `type/description` (e.g., `feature/graph-visualization`, `fix/neo4j-connection`)

### Commit Standards

```
type(scope): brief description

Detailed explanation of changes made.

- Feature additions or modifications
- Bug fixes or improvements
- Breaking changes noted

Closes #123
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Development Commands

```bash
# Full stack development
docker-compose up -d neo4j     # Start Neo4j only
npm run dev --prefix frontend  # Frontend dev server  
cd backend && source venv/bin/activate && uvicorn app.main:app --reload

# Testing
cd frontend && npm run test
cd backend && pytest tests/ --cov=app

# Production build
docker-compose build
docker-compose up -d
```

### Code Review Checklist

- [ ] Code follows project conventions and style guides
- [ ] All tests pass (frontend + backend)
- [ ] Type checking passes (TypeScript + mypy)
- [ ] Linting passes (ESLint + ruff)
- [ ] API endpoints have proper error handling
- [ ] Components have proper TypeScript types
- [ ] Database queries are optimized
- [ ] Security considerations addressed
- [ ] Documentation updated if needed

## 🧪 Testing Strategy

### Frontend Testing

```bash
# Install testing dependencies
npm install --save-dev @testing-library/react @testing-library/jest-dom vitest

# Test commands
npm run test          # Run all tests
npm run test:watch    # Watch mode
npm run test:coverage # Coverage report
```

### Backend Testing

```python
# pytest configuration in backend/
# Always run in virtual environment

# Test structure
backend/tests/
├── conftest.py           # Shared fixtures
├── test_api/
│   ├── test_documents.py # API endpoint tests
│   └── test_chat.py      # Chat functionality tests
├── test_services/
│   ├── test_graph.py     # Neo4j service tests
│   └── test_llm.py       # LLM integration tests
└── integration/
    └── test_rag_pipeline.py  # End-to-end tests
```

```bash
# Test commands (in venv)
pytest tests/ -v                    # All tests
pytest tests/test_api/ -v           # API tests only
pytest tests/ --cov=app --cov-report=html  # Coverage
```

## 🚀 Deployment & Production

### Docker Deployment

```yaml
# docker-compose.yml
services:
  neo4j:
    image: neo4j:5.15-community
    environment:
      NEO4J_AUTH: neo4j/your_password
    ports: ["7474:7474", "7687:7687"]
    
  backend:
    build: ./backend
    environment:
      NEO4J_URI: bolt://neo4j:7687
    ports: ["8000:8000"]
    depends_on: [neo4j]
    
  frontend:
    build: ./frontend
    environment:
      NEXT_PUBLIC_API_URL: http://backend:8000
    ports: ["3000:3000"]
    depends_on: [backend]
```

### Production Considerations

- **Environment Variables**: Use `.env` files, never commit secrets
- **SSL/TLS**: Configure HTTPS for production deployment
- **Resource Limits**: Set appropriate memory/CPU limits for containers
- **Monitoring**: Add logging, metrics, and health checks
- **Backup**: Regular Neo4j database backups
- **Security**: API rate limiting, input validation, CORS configuration

## 🔧 Troubleshooting Guide

### Common Issues

1. **Neo4j Connection Failed**
   ```bash
   # Check if Neo4j is running
   docker-compose ps
   # Check logs
   docker-compose logs neo4j
   ```

2. **Frontend Build Errors**
   ```bash
   # Clear Next.js cache
   rm -rf .next/
   npm run build
   ```

3. **Python Import Errors**
   ```bash
   # Ensure venv is activated
   which python  # Should show venv path
   pip install -r requirements.txt
   ```

4. **Vector Search Performance**
   ```cypher
   // Check index status
   SHOW INDEXES;
   // Verify vector dimensions match
   MATCH (n:Chunk) WHERE n.embedding IS NOT NULL 
   RETURN size(n.embedding) as embedding_size LIMIT 1;
   ```

### Performance Optimization

- **Frontend**: Use React.memo, useMemo, useCallback for expensive operations
- **Backend**: Implement async/await properly, use connection pooling
- **Database**: Create appropriate indexes, optimize Cypher queries
- **LLM**: Cache embeddings, use batching for multiple requests

## 📚 Learning Resources

### Essential Documentation

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Next.js Documentation](https://nextjs.org/docs)
- [Neo4j Documentation](https://neo4j.com/docs/)
- [Cytoscape.js Documentation](https://js.cytoscape.org/)
- [Ollama Documentation](https://ollama.ai/docs)

### RAG & Knowledge Graphs

- [Microsoft GraphRAG](https://microsoft.github.io/graphrag/)
- [Neo4j Graph Data Science](https://neo4j.com/docs/graph-data-science/)
- [RAG Best Practices](https://docs.llamaindex.ai/en/stable/optimizing/production_rag/)

## ⚠️ Critical Reminders

### For Claude Code

1. **ALWAYS** activate the Python virtual environment before running Python commands:
   ```bash
   cd backend && source venv/bin/activate  # or venv\Scripts\activate on Windows
   ```

2. **ALWAYS** use Turbopack for frontend development (it's now the default):
   ```bash
   cd frontend && npm run dev  # Uses --turbo flag automatically
   ```

3. **NEVER** assume dependencies exist - check package.json/requirements.txt first

4. **ALWAYS** use TypeScript types in frontend components - no `any` types

5. **ALWAYS** include proper error handling in API endpoints

6. **ALWAYS** check if Neo4j connection is working before running queries

7. **NEVER** commit sensitive information (.env files, API keys)

8. **ALWAYS** run tests before suggesting code changes

9. **ALWAYS** use the project's established patterns and conventions

### Development Notes

- The project uses **qwen2.5:7b-instruct** as the default Ollama model
- Frontend runs on port **3000**, backend on **8000**, Neo4j on **7474/7687**
- All API endpoints are prefixed with `/api/v1`
- Graph visualization uses **Cytoscape.js**, not other graph libraries
- Vector embeddings are **384-dimensional** by default (sentence-transformers)

---

This guide is a living document. Update it as the project evolves and new patterns emerge. Always refer to component-specific CLAUDE.md files in `frontend/` and `backend/` directories for detailed implementation guidance.

---
> Source: [zhijiewong/ragviz](https://github.com/zhijiewong/ragviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
