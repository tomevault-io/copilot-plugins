## omnistate

> This file provides guidance for agentic coding agents operating in the DML V4 repository.

# AGENTS.md

This file provides guidance for agentic coding agents operating in the DML V4 repository.

## Build/Lint/Test Commands

### Backend Commands

#### Development Server
```bash
cd backend
python -m app.main                    # Start FastAPI server (port 8000)
```

#### Database Initialization
```bash
cd backend
python app/init_mongodb.py             # Initialize MongoDB with workflow configs
python scripts/init_rbac.py            # Initialize RBAC (roles/permissions)
python scripts/create_user.py          # Create admin user
```

#### Testing (pytest)
```bash
cd backend

# All tests
pytest

# Specific test file
pytest tests/unit/workflow/test_workflow_service.py -v

# Integration tests
pytest tests/integration/ -v

# With coverage
pytest --cov=app

# Specific module
pytest tests/unit/workflow/

# Single test method
pytest tests/unit/workflow/test_workflow_service.py::test_create_item -v
```

#### Linting (flake8)
```bash
flake8                                  # Max line length: 110, max complexity: 12
flake8 app/modules/workflow/service/    # Lint specific directory
flake8 --select=E,W,F                   # Specific error codes
```

#### Dependencies
```bash
pip install -r requirements.txt         # Install Python dependencies
```

### Frontend Commands

#### Development Server
```bash
cd frontend
npm run dev                             # Start Vite dev server (port 3000)
```

#### Build & Preview
```bash
cd frontend
npm run build                           # Production build
npm run preview                         # Preview production build
npm run clean                           # Clean dist directory
```

#### Linting & Type Checking
```bash
cd frontend
npm run lint                            # TypeScript type check (tsc --noEmit)
```

#### Dependencies
```bash
cd frontend
npm install                             # Install Node.js dependencies
```

## Code Style Guidelines

### Backend (Python)

#### General Rules
- **Max line length**: 110 characters
- **Max complexity**: 12 per function/method
- **Python version**: 3.10+ (Python 3.13 compatible)
- **Async patterns**: Use async/await for I/O operations
- **Logging**: Use `from app.shared.core.logger import log` for structured logging

#### Architecture & Layering
- **Strong layering**: API → Service → Repository/Domain (no upward dependencies)
- **API Layer**: FastAPI route handlers in `app/modules/*/api/`
- **Service Layer**: Business logic in `app/modules/*/service/`
- **Repository Layer**: Data access in `app/modules/*/repository/`
- **Domain Layer**: Business rules in `app/modules/*/domain/`

#### Import Organization
```python
# Standard library imports
import re
from typing import Dict, Any, Optional, List

# Third-party imports
from beanie import PydanticObjectId
from pymongo import AsyncMongoClient

# Local application imports (absolute imports from app/)
from app.modules.workflow.repository.models import (
    SysWorkflowConfigDoc,
    BusWorkItemDoc,
)
from app.shared.core.logger import log as logger
```

#### Type Hints & Documentation
```python
def create_item(
    self,
    type_code: str,
    creator_id: str,
    form_data: Optional[Dict[str, Any]] = None
) -> BusWorkItemDoc:
    """
    Create new work item in DRAFT state.

    Args:
        type_code: Business item type (REQUIREMENT, TEST_CASE, etc.)
        creator_id: User ID of the item creator
        form_data: Optional form data for initial item state

    Returns:
        Created work item document

    Raises:
        WorkItemCreationError: If creation fails
    """
```

#### Error Handling
- Use domain-specific exceptions in `app/modules/*/domain/exceptions.py`
- Handle MongoDB-specific errors (DuplicateKeyError, OperationFailure)
- Use proper async exception handling with try/catch blocks
- Log errors with context using structured logger

#### Naming Conventions
- **Classes**: PascalCase (`AsyncWorkflowService`)
- **Functions/methods**: snake_case (`create_item`, `handle_transition`)
- **Variables**: snake_case (`current_state`, `type_code`)
- **Constants**: UPPER_SNAKE_CASE (`MONGO_URI`)
- **Private methods**: prefix with underscore (`_internal_method`)

#### MongoDB/Beanie Patterns
- Use Beanie ODM for MongoDB operations
- Document models inherit from `Document` class
- Always filter by `is_deleted: False` for soft delete pattern
- Use `PydanticObjectId` for document IDs
- Implement proper indexing in document models

### Frontend (React + TypeScript)

#### General Rules
- **Framework**: React 19 + TypeScript + Vite
- **Styling**: TailwindCSS 4
- **Single-file architecture**: All views in `src/App.tsx`
- **TypeScript**: Strict mode with comprehensive type definitions

#### Component Structure
```typescript
/**
 * Component description
 * Explains what this component does
 */

import React, { useState, useCallback } from 'react';
import { Component } from './Component';

// Types first
interface Props {
  data: DataType;
  onAction: (id: string) => void;
}

// Component implementation
const MyComponent: React.FC<Props> = ({ data, onAction }) => {
  const [state, setState] = useState<Type>();

  const handleClick = useCallback((id: string) => {
    onAction(id);
  }, [onAction]);

  return (
    <div className="...">
      {/* Component JSX */}
    </div>
  );
};

export default MyComponent;
```

#### Import Organization
```typescript
// ========== 类型导入 ==========
import { TestCase, User } from './types';

// ========== Hook导入 ==========
import { useLocalAI } from './hooks/useLocalAI';

// ========== API服务导入 ==========
import { api } from './services/api';

// ========== 组件导入 ==========
import { Component } from './components';

// ========== 图标库导入 ==========
import { Icon } from 'lucide-react';
```

#### TypeScript Patterns
- Use comprehensive interface definitions in `src/types.ts`
- Prefer `React.FC<Props>` for function components
- Use `useCallback` for event handlers to prevent unnecessary re-renders
- Use proper typing for API responses and form data

#### TailwindCSS Patterns
- Use consistent utility classes
- Follow mobile-first responsive design
- Use Tailwind's color palette consistently
- Group related styles together

#### Error Handling
- Handle API errors gracefully with user feedback
- Use try/catch for async operations
- Display loading states during async operations
- Implement proper error boundaries

## Project-Specific Patterns

### Workflow System
- Workflow configurations defined in JSON files (`app/configs/*.json`)
- Business rules: `type_code` + `from_state` + `action` → `to_state`
- Owner strategies: `KEEP`, `TO_CREATOR`, `TO_SPECIFIC_USER`
- Always validate workflow consistency on startup

### API Design
- **Prefix**: All APIs use `/api/v1` prefix
- **Response format**: Unified Envelope `{"code": 0, "message": "ok", "data": {}}`
- **Authentication**: JWT-based with RBAC
- **Error handling**: Use FastAPI exception handlers

### Database Design
- **Soft delete**: All business documents have `is_deleted: false` filter
- **Audit trail**: Use `BusFlowLogDoc` for state transitions
- **Indexing**: Proper indexes on frequently queried fields
- **Transactions**: Use MongoDB transactions when supported

### Environment Configuration
- Backend: `backend/.env` with MongoDB, JWT, CORS settings
- Frontend: `frontend/.env` with API base URL and timeout settings
- Never commit secrets to version control

## Testing Guidelines

### Backend Testing
- **Unit tests**: Test services and domain logic in `tests/unit/`
- **Integration tests**: Test API endpoints in `tests/integration/`
- **Test patterns**: Use fakes for MongoDB operations
- **Async testing**: Use pytest-asyncio for async test cases
- **Coverage**: Aim for high coverage of business logic

### Frontend Testing
- Currently no formal testing setup (npm test shows error)
- Consider adding Jest + React Testing Library for component testing

## Security Guidelines
- **Authentication**: Validate JWT tokens on all protected routes
- **Authorization**: Check RBAC permissions for sensitive operations
- **Input validation**: Use Pydantic models for request validation
- **SQL injection**: Use parameterized queries (Beanie handles this)
- **XSS protection**: Sanitize user inputs in frontend components

## Development Workflow
1. **Feature development**: Follow layered architecture strictly
2. **Testing**: Write tests for new functionality
3. **Linting**: Run flake8 and TypeScript checks before committing
4. **Documentation**: Update relevant documentation files
5. **Database changes**: Update MongoDB initialization scripts if needed

---
> Source: [hankerbiao/omnistate](https://github.com/hankerbiao/omnistate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
