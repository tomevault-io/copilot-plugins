## rag-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Objective
- Developing a project to store documents (csv or pdf) into a pgvector database

1. Data Loading:
- Content is stored in userguide.csv with fields: headingTrace, pageTrace, page_id, section_id,
content, and enhancedContent
2. Vector Database Storage:
- When the application runs, the content is loaded into PGVector (PostgreSQL with vector extensions)
- Text is embedded using the all-mpnet-base-v2 model for semantic search
3. Query Processing:
- User questions are analyzed using two methods:
    - TF-IDF retrieval (keyword matching)
    - Vector similarity search (semantic matching)
- Both approaches are combined to retrieve the most relevant documentation
4. Response Generation:
- The system finds the best matching documentation sections
- Uses a language model to generate a helpful response based on the retrieved context
- Formats the response with "Learn more" links to related sections
- Returns a structured JSON object with the answer and optional secondary links

Integration with Main System:

- User questions are routed to the User Guide executor via the Router module
- The system uses it as a fallback for unclassified questions
- It's fully integrated with the conversation management system to maintain context

## Commands
- Setup: `pip install -r requirements.txt`
- Run server: `uvicorn app.main:app --reload`
- Lint: `flake8 . && black --check .`
- Type check: `mypy .`
- Tests: `pytest`
- Single test: `pytest tests/path_to_test.py::test_function_name -v`

## Style Guidelines
- Imports: Group standard library, FastAPI, SQLAlchemy, pgvector, then local imports
- Formatting: Black with default 88 character line length
- Types: Use Pydantic models for API schemas; type hints for all functions
- FastAPI: Use dependency injection for DB connections and shared resources
- Database: Use SQLAlchemy async ORM with pgvector for vector operations
- Error handling: Return appropriate HTTP status codes with detailed error responses
- Naming: snake_case for variables/functions, CamelCase for classes/Pydantic models
- API endpoints: RESTful structure with consistent URL patterns

---
> Source: [homesh33p/rag_project](https://github.com/homesh33p/rag_project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
