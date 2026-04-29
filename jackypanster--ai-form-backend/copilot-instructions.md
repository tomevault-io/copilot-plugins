## ai-form-backend

> This project follows a modern web application architecture with the following key components:

# Technical Architecture Guide

This project follows a modern web application architecture with the following key components:

## Frontend (Next.js)
The frontend is built with Next.js and uses Shadcn UI for the component library. Key files and directories:

- Frontend components use Shadcn UI for a clean, modern interface
- State management uses React hooks (useState, useEffect)
- API communication handled through fetch/axios

## Backend (FastAPI)
The backend is built with FastAPI and provides the core API endpoints. Key components:

### API Endpoints
Main endpoint: `POST /api/v1/fill-form`
- Accepts form fields and source content
- Returns structured filled form data

### Data Models
Pydantic models for request/response validation:
- `FillFormRequest`
- `FilledDataResponse`
- `FillFormSuccessResponse`
- `ErrorResponse`

## External Services
The system integrates with external LLM APIs (e.g., DeepSeek API) for form filling functionality.

## Deployment
The application is containerized using Docker:
- [Dockerfile](mdc:Dockerfile) defines the container setup
- Environment variables used for sensitive configuration
- Supports both development and production deployments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackypanster) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
