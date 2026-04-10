## erp-eleven

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is an ERP system project named "Eleven" for managing clothing store operations. The system handles sales, vendors, currency exchange, orders, and employee data with multi-currency support.

**IMPORTANTE**: Para documentação completa do projeto, consulte o arquivo `README.md` na raiz do projeto que contém todas as informações técnicas, endpoints, configuração e regras de negócio.

## Development Commands

### Backend (FastAPI + PostgreSQL)
```bash
# Setup backend environment
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Copy environment configuration
cp .env.example .env

# Start development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# API documentation available at: http://localhost:8000/docs
# Alternative docs at: http://localhost:8000/redoc
```

### Database Setup
```bash
# Ensure PostgreSQL is running locally
# Create database and run schema
psql -U postgres -c "CREATE DATABASE eleven;"
psql -U postgres -d eleven -f postgres.sql
```

### Environment Configuration
Copy `backend/.env.example` to `backend/.env` and configure:
- **DATABASE_URL**: postgresql://postgres:postgres@localhost:5432/eleven
- **SECRET_KEY**: Generate a secure key for JWT tokens
- **ALGORITHM**: HS256 (for JWT)
- **ACCESS_TOKEN_EXPIRE_MINUTES**: 720

## Architecture

### Backend Structure (FastAPI)
The backend follows a clean architecture pattern:

```
backend/app/
├── main.py              # FastAPI application entry point
├── config.py            # Configuration management with environment variables
├── database.py          # SQLAlchemy database setup and session management
├── models/              # SQLAlchemy ORM models
│   ├── vendedor.py      # Vendor model with commissions and metrics
│   ├── venda.py         # Sales model with multi-currency support
│   ├── cambista.py      # Currency exchanger model
│   ├── pedido.py        # Orders model
│   ├── usuario.py       # User authentication model
│   ├── funcionario.py   # Employee model
│   └── comprovante.py   # Receipt/document model
├── schemas/             # Pydantic models for request/response validation
│   ├── vendedor.py      # Vendor schemas (Create, Response)
│   ├── venda.py         # Sales schemas
│   ├── cambista.py      # Currency exchanger schemas
│   └── pedido.py        # Order schemas
└── api/
    └── endpoints/       # API route handlers
        ├── auth.py      # Authentication endpoints
        ├── vendedores.py # Vendor management endpoints
        ├── vendas.py    # Sales tracking endpoints
        ├── cambistas.py # Currency exchange endpoints
        ├── pedidos.py   # Order management endpoints
        └── dashboard.py # Dashboard analytics endpoints
```

### Key Architecture Patterns
- **Dependency Injection**: Uses FastAPI's dependency system for database sessions
- **Schema Validation**: Pydantic models ensure type safety and validation
- **Separation of Concerns**: Clear separation between models, schemas, and endpoints
- **UUID Primary Keys**: All entities use UUID for better scalability
- **Timestamping**: Automatic created_at/updated_at tracking

## Database Schema

### Core Entities
The PostgreSQL database includes the following main entities:

- **usuarios**: User authentication and authorization (roles: ADMIN, GERENTE, VENDEDOR, etc.)
- **vendedores**: Vendor management with commission rates and weekly targets
- **vendas**: Multi-currency sales tracking (G$, R$, U$, EUR)
- **cambistas**: Currency exchange rate management
- **pedidos**: Order processing with status tracking and shipping
- **comprovantes**: Document management and approval workflows
- **funcionarios**: Employee management with scheduling

### Multi-Currency Support
- **G$**: Primary system currency
- **R$**: Brazilian Real
- **U$**: US Dollar  
- **EUR**: Euro

All financial calculations support these currencies with exchange rate conversion.

## API Endpoints

### Current Available Endpoints
- **Authentication**: `/api/auth/*` - User login, logout, token management
- **Vendors**: `/api/vendedores/*` - Vendor CRUD operations and metrics
- **Sales**: `/api/vendas/*` - Sales tracking and commission calculations  
- **Currency Exchange**: `/api/cambistas/*` - Exchange rate management
- **Orders**: `/api/pedidos/*` - Order processing and logistics
- **Dashboard**: `/api/dashboard/*` - Analytics and reporting data
- **Health Check**: `/health` - Service health monitoring
- **API Root**: `/` - Basic API information

### Common Endpoint Patterns
- `GET /{resource}/` - List all active records
- `POST /{resource}/` - Create new record
- `GET /{resource}/{id}` - Get specific record by UUID
- `PUT /{resource}/{id}` - Update existing record
- `DELETE /{resource}/{id}` - Soft delete (set ativo=False)

## Deployment

### Local Development
1. Set up PostgreSQL database and run `postgres.sql` schema
2. Configure backend environment variables in `.env`
3. Start backend: `uvicorn app.main:app --reload`
4. Frontend to be created with Vue.js 3 + TypeScript

### Production Deployment (Render)
The project includes `render.yaml` for easy deployment:
- **Database**: PostgreSQL service with automatic backups
- **Backend**: FastAPI service with auto-deploy from Git
- **Frontend**: Static site deployment (when created)

See `DEPLOYMENT.md` for detailed deployment instructions.

## Development Patterns

### Backend Development
- **Models**: Use SQLAlchemy with UUID primary keys and automatic timestamps
- **Schemas**: Create separate Pydantic models for Create/Response operations
- **Endpoints**: Follow REST conventions with proper HTTP status codes
- **Database Sessions**: Use dependency injection with `Depends(get_db)`
- **Error Handling**: Raise HTTPException with appropriate status codes

### Code Examples

#### Model Pattern
```python
# models/example.py
from sqlalchemy import Column, String, Boolean, DateTime, DECIMAL
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.sql import func
import uuid
from ..database import Base

class Example(Base):
    __tablename__ = "examples"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    nome = Column(String(100), nullable=False)
    ativo = Column(Boolean, default=True)
    created_at = Column(DateTime, default=func.current_timestamp())
    updated_at = Column(DateTime, default=func.current_timestamp(), onupdate=func.current_timestamp())
```

#### Schema Pattern
```python
# schemas/example.py
from pydantic import BaseModel
from typing import Optional
from datetime import datetime
import uuid

class ExampleBase(BaseModel):
    nome: str
    ativo: Optional[bool] = True

class ExampleCreate(ExampleBase):
    pass

class ExampleResponse(ExampleBase):
    id: uuid.UUID
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True
```

#### Endpoint Pattern
```python
# api/endpoints/examples.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List
from ...database import get_db
from ...models.example import Example
from ...schemas.example import ExampleCreate, ExampleResponse

router = APIRouter()

@router.get("/", response_model=List[ExampleResponse])
def listar_examples(db: Session = Depends(get_db)):
    return db.query(Example).filter(Example.ativo == True).all()

@router.post("/", response_model=ExampleResponse)
def criar_example(example: ExampleCreate, db: Session = Depends(get_db)):
    db_example = Example(**example.dict())
    db.add(db_example)
    db.commit()
    db.refresh(db_example)
    return db_example
```

## Frontend Guidelines (When Created)

### Planned Frontend Structure (Vue.js 3 + TypeScript)
- **Framework**: Vue.js 3 with Composition API
- **Language**: TypeScript for type safety
- **State Management**: Pinia
- **HTTP Client**: Axios with interceptors
- **Styling**: Tailwind CSS
- **Build Tool**: Vite

### Vue.js Development Patterns
- Use `<script setup>` with TypeScript
- Implement proper error handling and loading states
- Follow consistent naming conventions
- Use composables for reusable logic
- Implement proper form validation for financial data

## Important Development Notes

- **Currency Handling**: Always use Decimal for financial calculations, never float
- **UUID Validation**: Ensure proper UUID format validation in endpoints
- **Multi-tenancy**: Consider vendor isolation in data access patterns
- **Security**: Implement proper authentication and authorization
- **Performance**: Use database indexes for frequently queried fields
- **Testing**: Write tests for financial calculation logic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ferreiraluc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ferreiraluc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
