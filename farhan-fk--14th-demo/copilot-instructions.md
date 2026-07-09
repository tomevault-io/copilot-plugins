## 14th-demo

> This is a **Retrieval-Augmented Generation (RAG) Chatbot** system that answers questions about course materials using semantic search and Claude AI.

# CLAUDE.md - AI Context Documentation

## Project Overview
This is a **Retrieval-Augmented Generation (RAG) Chatbot** system that answers questions about course materials using semantic search and Claude AI.

**Repository**: https://github.com/farhan-fk/14th-Demo  
**Original Source**: https://github.com/https-deeplearning-ai/starting-ragchatbot-codebase

---

## Technology Stack

- **Backend Framework**: FastAPI (Python 3.13+)
- **Frontend**: HTML/CSS/JavaScript with Marked.js for markdown rendering
- **Vector Database**: ChromaDB for semantic search and embeddings
- **AI Model**: Anthropic Claude (via API) for response generation
- **Embeddings**: Sentence Transformers for text vectorization
- **Package Manager**: uv (fast Python package installer)

---

## Architecture Overview

### High-Level Components

```
Frontend (Web UI) 
    ↓
FastAPI Server (/api/query, /api/courses)
    ↓
RAG System (Orchestrator)
    ↓
├── Document Processor (Chunking & Parsing)
├── AI Generator (Claude API Integration)
├── Tool Manager (Search Tool)
├── Session Manager (Conversation History)
└── Vector Store (ChromaDB Interface)
    ↓
ChromaDB Collections:
├── course_catalog (metadata)
└── course_content (text chunks)
```

---

## Key Files & Their Responsibilities

### Backend (`backend/`)

| File | Purpose |
|------|---------|
| `app.py` | FastAPI server, defines API endpoints, serves frontend |
| `rag_system.py` | Main orchestrator coordinating all components |
| `vector_store.py` | ChromaDB wrapper for storing/retrieving course chunks |
| `ai_generator.py` | Anthropic Claude API client for response generation |
| `document_processor.py` | Processes course documents into structured chunks |
| `search_tools.py` | Tool-based search system for RAG (used by Claude) |
| `session_manager.py` | Manages conversation history per session |
| `models.py` | Pydantic data models (Course, Lesson, CourseChunk) |
| `config.py` | Configuration settings and environment variables |

### Frontend (`frontend/`)

| File | Purpose |
|------|---------|
| `index.html` | Web interface structure |
| `script.js` | Handles user interactions, API calls, message display |
| `style.css` | UI styling |

### Data (`docs/`)

Contains 4 course transcript files:
- `course1_script.txt` - Building Towards Computer Use with Anthropic
- `course2_script.txt` - MCP: Build Rich-Context AI Apps with Anthropic
- `course3_script.txt` - Advanced Retrieval for AI with Chroma
- `course4_script.txt` - Prompt Compression and Query Optimization

---

## Query Processing Flow

### Step-by-Step User Query Journey

1. **User Input** → Frontend captures query from input field
2. **API Request** → POST to `/api/query` with `{query, session_id}`
3. **RAG System** → Receives request, retrieves conversation history
4. **AI Generator** → Calls Claude API with:
   - System prompt
   - User query
   - Conversation history
   - Available tools (search_course_content)
5. **Tool Execution** (if Claude decides to search):
   - Search Tool → Vector Store
   - ChromaDB performs semantic search
   - Returns top N relevant chunks with metadata
6. **Response Generation** → Claude uses retrieved context to generate answer
7. **Session Update** → Conversation history saved
8. **API Response** → Returns `{answer, sources, session_id}`
9. **Frontend Display** → Shows formatted answer with collapsible sources

---

## Document Processing Pipeline

### Input Format
Course documents follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson N: [lesson title]
Lesson Link: [lesson url]
[lesson content...]
```

### Processing Steps

1. **Metadata Extraction**: Parse first 3-4 lines for course info
2. **Lesson Parsing**: Identify lesson markers (`Lesson N: title`)
3. **Text Chunking**: Split content using sentence-based chunking
   - Configurable chunk size (e.g., 1000 chars)
   - Overlap between chunks for context continuity
   - Preserves sentence boundaries
4. **Context Enhancement**: Prepend metadata to chunks
   - Format: `"Course {title} Lesson {N} content: {chunk}"`
5. **Vectorization**: Generate embeddings using Sentence Transformers
6. **Storage**: Save to ChromaDB with metadata (`course_title`, `lesson_number`, `chunk_index`)

---

## ChromaDB Collections

### 1. `course_catalog` (Metadata)
Stores course-level information for semantic course name matching:
- **ID**: Course title (unique identifier)
- **Document**: Course title text
- **Metadata**: `{title, instructor, course_link, lessons_json, lesson_count}`

### 2. `course_content` (Text Chunks)
Stores searchable course content:
- **ID**: `{course_title}_{chunk_index}`
- **Document**: Chunk text with context
- **Metadata**: `{course_title, lesson_number, chunk_index}`

---

## API Endpoints

### POST `/api/query`
**Purpose**: Process user questions and return AI-generated answers

**Request Body**:
```json
{
  "query": "What is RAG?",
  "session_id": "session_1" // optional
}
```

**Response**:
```json
{
  "answer": "RAG (Retrieval-Augmented Generation) is...",
  "sources": ["Advanced Retrieval for AI - Lesson 2"],
  "session_id": "session_1"
}
```

### GET `/api/courses`
**Purpose**: Get course statistics

**Response**:
```json
{
  "total_courses": 4,
  "course_titles": ["Course 1", "Course 2", ...]
}
```

---

## Environment Setup

### Prerequisites
- Python 3.13+
- uv package manager
- Anthropic API key

### Installation
```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Create .env file
echo "ANTHROPIC_API_KEY=your-key-here" > .env

# Run application
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Access Points
- **Web UI**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs

---

## Current State (as of Nov 14, 2025)

### Loaded Courses
- ✅ **4 courses** indexed
- ✅ **528 text chunks** stored in ChromaDB
- ✅ All courses searchable via semantic search

### Course Details
1. Building Towards Computer Use with Anthropic - 153 chunks
2. MCP: Build Rich-Context AI Apps with Anthropic - 164 chunks
3. Advanced Retrieval for AI with Chroma - 90 chunks
4. Prompt Compression and Query Optimization - 121 chunks

---

## Key Features

### 1. Tool-Based Search
Claude autonomously decides when to search course content using the `search_course_content` tool with parameters:
- `query`: What to search for
- `course_name`: Optional course filter (fuzzy matching supported)
- `lesson_number`: Optional lesson filter

### 2. Conversation Memory
Session Manager maintains conversation history (last 5 exchanges) for context-aware responses.

### 3. Semantic Course Matching
Vector search in `course_catalog` enables fuzzy course name matching (e.g., "MCP" matches "MCP: Build Rich-Context AI Apps").

### 4. Source Attribution
Each response includes sources showing which courses/lessons were used to generate the answer.

---

## Configuration (`config.py`)

Key settings:
- `CHUNK_SIZE`: 1000 characters
- `CHUNK_OVERLAP`: 200 characters
- `MAX_RESULTS`: 5 (top N chunks retrieved)
- `MAX_HISTORY`: 5 (conversation exchanges kept)
- `ANTHROPIC_MODEL`: Claude model identifier
- `EMBEDDING_MODEL`: Sentence Transformer model name
- `CHROMA_PATH`: Vector database storage location

---

## Security & Best Practices

### Gitignore Configuration
The following are excluded from version control:
- `.env` (API keys)
- `.venv/` (virtual environment)
- `__pycache__/` (Python cache)
- `backend/chroma_db/` (vector database)
- IDE configs (`.vscode/`, `.idea/`)

### CORS
Configured in `app.py` to allow cross-origin requests for development.

---

## Development Workflow

### Adding New Courses
1. Place course document in `docs/` folder
2. Follow format: `Course Title: ...` etc.
3. Restart server (auto-loads on startup)
4. Or use RAG System's `add_course_document()` method

### Modifying Prompts
System prompt is in `ai_generator.py` → `AIGenerator.SYSTEM_PROMPT`

### Changing Chunk Size
Update `CHUNK_SIZE` and `CHUNK_OVERLAP` in `config.py`

---

## Troubleshooting

### Server Won't Start
- Verify `.env` has valid `ANTHROPIC_API_KEY`
- Check Python version: `python --version` (need 3.13+)
- Reinstall dependencies: `uv sync`

### No Search Results
- Ensure courses are loaded (check startup logs)
- Verify ChromaDB path exists: `backend/chroma_db/`
- Try clearing and rebuilding: Delete `chroma_db/` folder and restart

### API Key Issues
- Verify key format: `sk-ant-api03-...`
- Check Anthropic console for usage/limits
- Ensure `.env` is in project root

---

## Future Enhancement Ideas

1. **Multi-file upload**: Allow users to upload their own course materials
2. **Advanced filters**: Filter by instructor, course date, difficulty level
3. **Export conversations**: Save chat history as PDF/markdown
4. **Analytics dashboard**: Track popular queries, course engagement
5. **Multi-language support**: Translate course content and responses
6. **Prompt caching**: Reduce latency and costs for repeated queries

---

## Useful Commands

```bash
# Start server
cd backend && uv run uvicorn app:app --reload --port 8000

# Check installed packages
uv pip list

# Run tests (if added)
uv run pytest

# Clear ChromaDB
rm -rf backend/chroma_db/

# View logs
tail -f backend/logs/app.log  # if logging configured

# Git operations
git status
git add .
git commit -m "message"
git push origin main
```

---

## Contact & Resources

- **Repository**: https://github.com/farhan-fk/14th-Demo
- **Anthropic Docs**: https://docs.anthropic.com/
- **ChromaDB Docs**: https://docs.trychroma.com/
- **FastAPI Docs**: https://fastapi.tiangolo.com/

---

*This document is maintained for AI assistants (like Claude) to quickly understand the project context and provide effective assistance.*

---
> Source: [farhan-fk/14th-Demo](https://github.com/farhan-fk/14th-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
