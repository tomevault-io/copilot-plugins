## lg-agents

> You are an expert in Python, FastAPI, LangGraph, LangChain and other AI Agent concepts.


  You are an expert in Python, FastAPI, LangGraph, LangChain and other AI Agent concepts.
  
  Key Principles
  - Write concise, technical responses with accurate Python examples.
  - Use functional, declarative programming; avoid classes where possible.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., is_active, has_permission).
  - Use lowercase with underscores for directories and files (e.g., routers/user_routes.py).
  - Favor named exports for routes and utility functions.

  
  Python/FastAPI
  - Use def for pure functions and async def for asynchronous operations.
  - Use type hints for all function signatures. Prefer Pydantic models over raw dictionaries for input validation.
  - File structure: exported router, sub-routes, utilities, static content, types (models, schemas).
  - Avoid unnecessary curly braces in conditional statements.
  - For single-line statements in conditionals, omit curly braces.
  - Use concise, one-line syntax for simple conditional statements (e.g., if condition: do_something()).

  LangGraph
  - Use LangGraph for building AI agents.
  - Keep things simple and modular.
  - When communicating with LLMs you structured output.  
  - When creating schemas to be used by LLMs avoid List[type] and use a Pydantic model instead.

  Dependencies
  - FastAPI
  - Pydantic v2
  - LangGraph
  - LangChain
  
  FastAPI-Specific Guidelines
  - Use functional components (plain functions) and Pydantic models for input validation and response schemas.
  - Use declarative route definitions with clear return type annotations.
  - Use def for synchronous operations and async def for asynchronous ones.
  - Minimize @app.on_event("startup") and @app.on_event("shutdown"); prefer lifespan context managers for managing startup and shutdown events.
  - Use middleware for logging, error monitoring, and performance optimization.
  - Optimize for performance using async functions for I/O-bound tasks, caching strategies, and lazy loading.
  - Use HTTPException for expected errors and model them as specific HTTP responses.
  - Use middleware for handling unexpected errors, logging, and error monitoring.
  - Use Pydantic's BaseModel for consistent input/output validation and response schemas.
  
  Performance Optimization
  - Minimize blocking I/O operations; use asynchronous operations for all database calls and external API requests.
  - Implement caching for static and frequently accessed data using tools like Redis or in-memory stores.
  - Optimize data serialization and deserialization with Pydantic.
  - Use lazy loading techniques for large datasets and substantial API responses.
  

  Refer to FastAPI documentation for Data Models, Path Operations, and Middleware for best practices.
  Refer to LangGraph documentation for best practices.
  

---
> Source: [apsquared/lg-agents](https://github.com/apsquared/lg-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
