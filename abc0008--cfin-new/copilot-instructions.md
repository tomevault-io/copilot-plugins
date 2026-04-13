## cfin-new

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CFIN (Financial Document Analysis System) is an AI-powered platform for analyzing financial documents. It combines PDF processing, conversational AI, and dynamic data visualization using Claude API as its core engine.

## Tech Stack

### Backend
- **Framework**: FastAPI (Python 3.8+)
- **Database**: SQLAlchemy with PostgreSQL/SQLite
- **AI Integration**: Anthropic Claude API (primary), LangChain/LangGraph (secondary)
- **Storage**: Local filesystem or AWS S3
- **Key Dependencies**: anthropic, langchain, fastapi, sqlalchemy, pydantic
- **Backend Start Terminal Command**: cd /Users/alexcardell/AlexCoding_Local/cfin/backend && PYTHONPATH=$PYTHONPATH:/Users/alexcardell/AlexCoding_Local/cfin/backend python -m uvicorn app.main:app --reload --port 8000

### Frontend
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **UI Components**: Tailwind CSS, shadcn/ui
- **Charts**: Recharts
- **State Management**: React hooks and context

## Development Setup

### Backend Setup
```bash
cd cfin/backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Set environment variables
export ANTHROPIC_API_KEY="your-api-key"
export BACKEND_URL="http://localhost:8000"
export CORS_ORIGINS="http://localhost:3000"

# Initialize database
python create_db.py

# Run development server
python run.py  # or uvicorn app.main:app --reload --port 8000
```

### Frontend Setup
```bash
cd cfin/nextjs-fdas
npm install
npm run dev  # Runs on http://localhost:3000
```

## Key Development Commands

### Backend Commands
```bash
# Run backend server
python run.py
python debug_server.py  # Debug mode with verbose logging
./restart_server.sh     # Restart server (kills existing process)

# Run tests
cd backend/tests
./run_tests.sh          # Run all tests
pytest test_specific.py  # Run specific test file
pytest -k "test_name"   # Run specific test by name
pytest -v              # Verbose output

# Database operations
python create_db.py     # Initialize/reset database
python migrate_claude_fields.py  # Run migrations

# API testing scripts
./test_api.sh          # Test all endpoints
./test_document_api_only.sh  # Test document endpoints
./test_document_endpoints.sh  # Document endpoint tests
```

### Frontend Commands
```bash
# Development
npm run dev            # Start development server
npm run build         # Build for production
npm run start         # Start production server
npm run lint          # Run ESLint
npm run test          # Run Jest tests
npm test -- --watch   # Run tests in watch mode
```

## Project Structure

```
cfin/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app entry point
│   │   └── routes/              # API endpoints
│   │       ├── document.py      # Document management
│   │       ├── conversation.py  # Chat interface
│   │       └── analysis.py      # Analysis operations
│   ├── services/
│   │   ├── analysis_service.py  # Analysis orchestration
│   │   ├── conversation_service.py
│   │   └── analysis_strategies/ # Analysis implementations
│   ├── pdf_processing/
│   │   ├── claude_file_client.py # Claude Files API integration
│   │   └── financial_agent.py    # Financial analysis logic
│   ├── models/                   # Pydantic models & DB schemas
│   ├── repositories/             # Data access layer
│   └── utils/                    # Utilities and helpers
├── nextjs-fdas/
│   └── src/
│       ├── app/                  # Next.js app router pages
│       │   ├── dashboard/        # Document library
│       │   └── workspace/        # Main analysis interface
│       ├── components/           # React components
│       │   ├── chat/            # Chat interface components
│       │   ├── charts/          # Visualization components
│       │   └── document/        # Document handling
│       └── lib/api/             # API client functions
└── uploads/                     # Document storage (local mode)
```

## API Endpoints

### Document Management
- `POST /api/documents/upload` - Upload PDF with Claude processing
- `GET /api/documents/{id}` - Get document details
- `GET /api/documents/` - List all documents
- `DELETE /api/documents/{id}` - Delete document
- `GET /api/documents/{id}/citations` - Get citations for highlighting

### Conversation/Chat
- `POST /api/conversation/message` - Send message with context
- `POST /api/conversation/create` - Create new conversation
- `GET /api/conversation/{id}` - Get conversation history
- `GET /api/conversation/history` - List all conversations

### Analysis
- `POST /api/analysis/run` - Execute analysis on documents
- `GET /api/analysis/{id}` - Get analysis results
- `GET /api/analysis/types` - List available analysis types

## Claude API Configuration

The system uses different Claude models based on the operation:
- **Document Processing**: Claude 3 Haiku (fast, cost-effective)
- **Analysis & Chat**: Claude 3 Sonnet (balanced performance)
- **Complex Analysis**: Claude 3 Opus (highest capability)

Key settings in `backend/settings.py`:
- `DEFAULT_MODEL`: "claude-3-5-sonnet-20241022"
- `MAX_TOKENS`: 4096
- `TEMPERATURE`: 0 (deterministic outputs)

## Key Features

### PDF Processing
- Native PDF support via Claude API (no OCR required)
- Automatic extraction of text, tables, and financial data
- Citation mapping for precise document references

### Analysis Types
1. **Comprehensive Analysis** - Full financial analysis with visualizations
2. **Financial Template** - Structured analysis using predefined templates
3. **Basic Financial** - Quick financial overview
4. **Sentiment Analysis** - Tone and sentiment assessment

### Tool System
The backend implements a tool-based system for Claude interactions:
- Tool definitions in `models/tools.py`
- Tool processing logic in `utils/tool_processing.py`
- Supports: charts, tables, metrics, financial calculations
- **Critical**: Chart tools require `xAxisKey` in config and transform data to `{x, y}` format
- Multi-series charts automatically detected and processed differently than single-series

## Testing Strategy

### Backend Testing
```bash
# Unit tests
pytest backend/tests/unit/

# Integration tests
pytest backend/tests/integration/

# Performance tests
pytest backend/tests/performance/

# Run with coverage
pytest --cov=app --cov=services --cov=repositories
```

### Frontend Testing
```bash
# Component tests
npm test

# Specific component
npm test -- ChartRenderer

# E2E tests (if configured)
npm run test:e2e
```

## Development Tips

1. **Claude API Rate Limits**: The system implements automatic retry logic and rate limit handling
2. **Document Caching**: Uploaded documents are cached using Claude Files API for 7 days
3. **Error Handling**: All API endpoints return standardized error responses
4. **Logging**: Use `secure_logging` utility to avoid exposing sensitive data
5. **Database Migrations**: Always test migrations on a copy before applying to production
6. **Schema Alignment**: All Pydantic models use `ConfigDict(alias_generator=to_camel, populate_by_name=True)` for consistent camelCase API responses
7. **Visualization Data**: Use `List[Dict[str, Any]]` for visualization fields (`monetary_values`, `percentages`) - not dictionaries
8. **Tool Processing**: Charts require `xAxisKey` in config; data is automatically transformed to `{x, y}` format for frontend consumption

## Common Issues & Solutions

1. **CORS Errors**: Ensure `CORS_ORIGINS` env var matches your frontend URL
2. **Claude API Errors**: Check API key and rate limits
3. **Database Connection**: Verify `DATABASE_URL` is set correctly
4. **File Upload Issues**: Check `MAX_FILE_SIZE` setting (default: 50MB)
5. **Pydantic Validation Errors**: Check that `model_dump(by_alias=True)` is used for API responses to ensure camelCase output
6. **Chart Data Errors**: Missing `xAxisKey` in chart config will cause validation failures; all chart data needs x/y structure
7. **Dict vs List Type Errors**: Visualization helpers return lists, not dicts - update models accordingly

## Deployment Considerations

- Set proper environment variables for all services
- Enable HTTPS for all endpoints
- Configure proper CORS origins for production domains

<Gemini_CLI_Usage_For_Long_Context_Analysis>

# Using Gemini CLI for Large Codebase Analysis

When analyzing large codebases or multiple files that might exceed context limits, use the Gemini CLI with its massive
context window. Use `gemini -p` to leverage Google Gemini's large context capacity.

## File and Directory Inclusion Syntax

Use the `@` syntax to include files and directories in your Gemini prompts. The paths should be relative to WHERE you run the
  gemini command:

### Examples:

**Single file analysis:**
gemini -p "@src/main.py Explain this file's purpose and structure"

Multiple files:
gemini -p "@package.json @src/index.js Analyze the dependencies used in the code"

Entire directory:
gemini -p "@src/ Summarize the architecture of this codebase"

Multiple directories:
gemini -p "@src/ @tests/ Analyze test coverage for the source code"

Current directory and subdirectories:
gemini -p "@./ Give me an overview of this entire project"

# Or use --all_files flag:
gemini --all_files -p "Analyze the project structure and dependencies"

Implementation Verification Examples

Check if a feature is implemented:
gemini -p "@src/ @lib/ Has dark mode been implemented in this codebase? Show me the relevant files and functions"

Verify authentication implementation:
gemini -p "@src/ @middleware/ Is JWT authentication implemented? List all auth-related endpoints and middleware"

Check for specific patterns:
gemini -p "@src/ Are there any React hooks that handle WebSocket connections? List them with file paths"

Verify error handling:
gemini -p "@src/ @api/ Is proper error handling implemented for all API endpoints? Show examples of try-catch blocks"

Check for rate limiting:
gemini -p "@backend/ @middleware/ Is rate limiting implemented for the API? Show the implementation details"

Verify caching strategy:
gemini -p "@src/ @lib/ @services/ Is Redis caching implemented? List all cache-related functions and their usage"

Check for specific security measures:
gemini -p "@src/ @api/ Are SQL injection protections implemented? Show how user inputs are sanitized"

Verify test coverage for features:
gemini -p "@src/payment/ @tests/ Is the payment processing module fully tested? List all test cases"

When to Use Gemini CLI

Use gemini -p when:
- Analyzing entire codebases or large directories
- Comparing multiple large files
- Need to understand project-wide patterns or architecture
- Current context window is insufficient for the task
- Working with files totaling more than 100KB
- Verifying if specific features, patterns, or security measures are implemented
- Checking for the presence of certain coding patterns across the entire codebase

Important Notes

- Paths in @ syntax are relative to your current working directory when invoking gemini
- The CLI will include file contents directly in the context
- No need for --yolo flag for read-only analysis
- Gemini's context window can handle entire codebases that would overflow Claude's context
- When checking implementations, be specific about what you're looking for to get accurate results
</Gemini_CLI_Usage_For_Long_Context_Analysis>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abc0008) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
