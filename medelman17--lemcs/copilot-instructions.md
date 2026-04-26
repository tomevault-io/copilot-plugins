## lemcs

> FastAPI development guidelines for legal document processing endpoints


# FastAPI Development Guidelines

## API Architecture

The LeMCS API provides endpoints for legal document processing, analysis, and consolidation using FastAPI with async patterns.

### Core API Structure
```
api/
├── routes/
│   ├── documents.py          # Document upload and processing
│   ├── citations.py          # Citation extraction endpoints
│   ├── consolidation.py      # Document consolidation
│   ├── health.py            # Health check endpoints
│   └── documents_simple.py  # Simplified document endpoints
```

## Endpoint Development Patterns

### Document Processing Endpoint
```python
from fastapi import APIRouter, UploadFile, File, Depends, HTTPException
from nlp.hybrid_legal_nlp import HybridLegalNLP
from typing import Dict, Any
import asyncio

router = APIRouter(prefix="/documents", tags=["documents"])

@router.post("/analyze")
async def analyze_legal_document(
    document: UploadFile = File(...),
    nlp_service: HybridLegalNLP = Depends(get_hybrid_nlp)
) -> Dict[str, Any]:
    """
    Analyze uploaded legal document with comprehensive NLP processing.
    
    Returns:
        - Document type classification
        - Entity extraction results
        - Legal concept analysis
        - Confidence scores
    """
    try:
        # Validate file type
        if not document.content_type.startswith('text/'):
            raise HTTPException(
                status_code=400, 
                detail="Only text files are supported"
            )
        
        # Extract text content
        content = await document.read()
        text = content.decode('utf-8')
        
        # Process with legal NLP
        loop = asyncio.get_event_loop()
        analysis = await loop.run_in_executor(
            None, nlp_service.comprehensive_analysis, text
        )
        
        return {
            "status": "success",
            "filename": document.filename,
            "document_type": analysis["document_analysis"]["type"],
            "confidence": analysis["document_analysis"]["confidence"],
            "analysis": analysis
        }
        
    except UnicodeDecodeError:
        raise HTTPException(
            status_code=400,
            detail="File encoding not supported. Please use UTF-8."
        )
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Document analysis failed: {str(e)}"
        )
```

### Batch Processing Endpoint
```python
from typing import List

@router.post("/analyze-batch")
async def analyze_documents_batch(
    documents: List[UploadFile] = File(...),
    nlp_service: HybridLegalNLP = Depends(get_hybrid_nlp)
) -> Dict[str, Any]:
    """
    Analyze multiple legal documents in parallel.
    
    Args:
        documents: List of uploaded legal documents
        
    Returns:
        Batch analysis results with individual document analyses
    """
    if len(documents) > 10:
        raise HTTPException(
            status_code=400,
            detail="Maximum 10 documents per batch"
        )
    
    try:
        # Extract text from all documents
        texts = []
        filenames = []
        
        for doc in documents:
            content = await doc.read()
            text = content.decode('utf-8')
            texts.append(text)
            filenames.append(doc.filename)
        
        # Process in parallel
        async def analyze_document(text: str) -> Dict[str, Any]:
            loop = asyncio.get_event_loop()
            return await loop.run_in_executor(
                None, nlp_service.comprehensive_analysis, text
            )
        
        analyses = await asyncio.gather(*[
            analyze_document(text) for text in texts
        ])
        
        # Format results
        results = []
        for filename, analysis in zip(filenames, analyses):
            results.append({
                "filename": filename,
                "document_type": analysis["document_analysis"]["type"],
                "confidence": analysis["document_analysis"]["confidence"],
                "entity_count": len(analysis["entities"]["combined_unique"]),
                "analysis": analysis
            })
        
        return {
            "status": "success",
            "processed_count": len(results),
            "results": results
        }
        
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Batch processing failed: {str(e)}"
        )
```

## Dependency Injection

### NLP Service Dependencies
```python
from functools import lru_cache

@lru_cache(maxsize=1)
def get_hybrid_nlp() -> HybridLegalNLP:
    """Get cached hybrid NLP instance."""
    return HybridLegalNLP()

async def get_nlp_service() -> HybridLegalNLP:
    """FastAPI dependency for NLP service."""
    return get_hybrid_nlp()

# Example usage in endpoint
@router.get("/nlp-status")
async def get_nlp_status(
    nlp_service: HybridLegalNLP = Depends(get_nlp_service)
) -> Dict[str, Any]:
    """Get status of NLP services."""
    status = nlp_service.get_service_status()
    return {
        "nlp_status": status,
        "model_info": nlp_service.bert_service.get_model_info()
    }
```

### Database Dependencies
```python
from sqlalchemy.ext.asyncio import AsyncSession
from db.database import get_database

async def get_db() -> AsyncSession:
    """Get database session."""
    async with get_database() as session:
        yield session

@router.post("/documents/store")
async def store_legal_document(
    content: str,
    analysis: Dict[str, Any],
    db: AsyncSession = Depends(get_db),
    nlp_service: HybridLegalNLP = Depends(get_nlp_service)
) -> Dict[str, Any]:
    """Store legal document with analysis in database."""
    try:
        # Get semantic embedding
        embedding = nlp_service.get_text_embedding(content)
        
        # Store in database
        document = Document(
            content=content,
            document_type=analysis["document_analysis"]["type"],
            embedding=embedding.numpy().tolist(),
            metadata=analysis
        )
        
        db.add(document)
        await db.commit()
        await db.refresh(document)
        
        return {
            "status": "success",
            "document_id": document.id,
            "document_type": document.document_type
        }
        
    except Exception as e:
        await db.rollback()
        raise HTTPException(
            status_code=500,
            detail=f"Failed to store document: {str(e)}"
        )
```

## Request/Response Models

### Pydantic Models
```python
from pydantic import BaseModel, validator
from typing import Optional, List, Dict, Any
from enum import Enum

class DocumentType(str, Enum):
    LEASE_AGREEMENT = "lease_agreement"
    LEGAL_COMPLAINT = "legal_complaint"
    LEGAL_MEMORANDUM = "legal_memorandum"
    CONTRACT = "contract"
    UNKNOWN = "unknown"

class AnalysisRequest(BaseModel):
    """Request model for document analysis."""
    content: str
    document_type: Optional[DocumentType] = None
    
    @validator('content')
    def validate_content(cls, v):
        if not v.strip():
            raise ValueError('Document content cannot be empty')
        if len(v) > 1_000_000:  # 1MB text limit
            raise ValueError('Document too large')
        return v

class EntityResponse(BaseModel):
    """Response model for extracted entities."""
    text: str
    label: str
    confidence: Optional[float] = None
    start: Optional[int] = None
    end: Optional[int] = None

class AnalysisResponse(BaseModel):
    """Response model for document analysis."""
    status: str
    document_type: DocumentType
    confidence: float
    entities: List[EntityResponse]
    legal_concepts: Dict[str, List[str]]
    complexity_score: float
    processing_time: Optional[float] = None

# Example endpoint with Pydantic models
@router.post("/analyze-text", response_model=AnalysisResponse)
async def analyze_text(
    request: AnalysisRequest,
    nlp_service: HybridLegalNLP = Depends(get_nlp_service)
) -> AnalysisResponse:
    """Analyze legal text with structured request/response."""
    import time
    
    start_time = time.time()
    
    try:
        analysis = nlp_service.comprehensive_analysis(request.content)
        processing_time = time.time() - start_time
        
        # Convert to response model
        entities = [
            EntityResponse(**entity) 
            for entity in analysis["entities"]["combined_unique"]
        ]
        
        return AnalysisResponse(
            status="success",
            document_type=analysis["document_analysis"]["type"],
            confidence=analysis["document_analysis"]["confidence"],
            entities=entities,
            legal_concepts=analysis["entities"]["legal_concepts"],
            complexity_score=analysis["document_analysis"]["complexity_score"],
            processing_time=processing_time
        )
        
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Analysis failed: {str(e)}"
        )
```

## Error Handling

### Custom Exception Handlers
```python
from fastapi import Request
from fastapi.responses import JSONResponse

class LegalProcessingException(Exception):
    """Custom exception for legal processing errors."""
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code

@app.exception_handler(LegalProcessingException)
async def legal_processing_exception_handler(
    request: Request, 
    exc: LegalProcessingException
):
    """Handle legal processing exceptions."""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": "Legal Processing Error",
            "message": exc.message,
            "request_id": getattr(request.state, "request_id", None)
        }
    )

# Usage in endpoints
async def process_legal_document(text: str):
    try:
        return nlp.comprehensive_analysis(text)
    except Exception as e:
        raise LegalProcessingException(
            f"Failed to process legal document: {str(e)}",
            status_code=422
        )
```

### Validation Error Handling
```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(
    request: Request, 
    exc: RequestValidationError
):
    """Handle request validation errors with legal context."""
    return JSONResponse(
        status_code=422,
        content={
            "error": "Validation Error",
            "message": "Invalid legal document data provided",
            "details": exc.errors()
        }
    )
```

## Middleware and Logging

### Request Logging Middleware
```python
import structlog
import time
import uuid

logger = structlog.get_logger()

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    """Log all requests with legal document context."""
    request_id = str(uuid.uuid4())
    request.state.request_id = request_id
    
    start_time = time.time()
    
    logger.info(
        "Request started",
        request_id=request_id,
        method=request.method,
        url=str(request.url),
        client_ip=request.client.host
    )
    
    try:
        response = await call_next(request)
        
        logger.info(
            "Request completed",
            request_id=request_id,
            status_code=response.status_code,
            duration=time.time() - start_time
        )
        
        return response
        
    except Exception as e:
        logger.error(
            "Request failed",
            request_id=request_id,
            error=str(e),
            duration=time.time() - start_time
        )
        raise
```

### Security Middleware
```python
@app.middleware("http")
async def security_middleware(request: Request, call_next):
    """Add security headers for legal document processing."""
    response = await call_next(request)
    
    # Add security headers
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    
    # Legal document specific headers
    response.headers["X-Legal-Processing"] = "true"
    response.headers["X-Data-Classification"] = "confidential"
    
    return response
```

## Performance Optimization

### Async Processing Patterns
```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

# Create thread pool for CPU-intensive NLP tasks
nlp_executor = ThreadPoolExecutor(max_workers=4, thread_name_prefix="nlp-")

async def async_nlp_processing(text: str, nlp_service: HybridLegalNLP):
    """Process NLP in thread pool to avoid blocking event loop."""
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(
        nlp_executor, nlp_service.comprehensive_analysis, text
    )

@router.post("/analyze-async")
async def analyze_document_async(
    document: UploadFile = File(...),
    nlp_service: HybridLegalNLP = Depends(get_nlp_service)
):
    """Analyze document asynchronously without blocking."""
    content = await document.read()
    text = content.decode('utf-8')
    
    # Process in thread pool
    analysis = await async_nlp_processing(text, nlp_service)
    
    return {"status": "success", "analysis": analysis}
```

### Response Compression
```python
from fastapi.middleware.gzip import GZipMiddleware

# Add compression for large legal document responses
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

## Health Checks and Monitoring

### Health Check Endpoints
```python
@router.get("/health")
async def health_check(
    nlp_service: HybridLegalNLP = Depends(get_nlp_service)
) -> Dict[str, Any]:
    """Comprehensive health check for legal processing system."""
    health_status = {
        "status": "healthy",
        "timestamp": time.time(),
        "services": {}
    }
    
    try:
        # Check NLP service
        nlp_status = nlp_service.get_service_status()
        health_status["services"]["nlp"] = {
            "status": "healthy" if nlp_status["spacy_available"] else "unhealthy",
            "details": nlp_status
        }
        
        # Check database connectivity
        # Add database health check
        
        # Check model loading
        test_analysis = nlp_service.comprehensive_analysis("Test legal document.")
        health_status["services"]["models"] = {
            "status": "healthy",
            "test_processing_time": time.time() - start_time
        }
        
    except Exception as e:
        health_status["status"] = "unhealthy"
        health_status["error"] = str(e)
    
    return health_status

@router.get("/metrics")
async def get_metrics() -> Dict[str, Any]:
    """Get system metrics for monitoring."""
    return {
        "processed_documents": document_counter.get(),
        "average_processing_time": avg_processing_time.get(),
        "error_rate": error_rate.get(),
        "model_memory_usage": get_model_memory_usage()
    }
```

## API Documentation

### OpenAPI Configuration
```python
from fastapi import FastAPI

app = FastAPI(
    title="LeMCS Legal Document Processing API",
    description="""
    Legal Memoranda Consolidation System API for processing and analyzing legal documents.
    
    Features:
    - Legal document classification
    - Entity extraction with legal-specific models
    - Citation extraction and validation
    - Document consolidation and synthesis
    - Semantic similarity analysis
    """,
    version="1.0.0",
    contact={
        "name": "LeMCS Development Team",
        "email": "dev@lemcs.com"
    },
    license_info={
        "name": "MIT License",
        "url": "https://opensource.org/licenses/MIT"
    }
)

# Add custom OpenAPI tags
tags_metadata = [
    {
        "name": "documents",
        "description": "Legal document upload and analysis operations"
    },
    {
        "name": "citations",
        "description": "Legal citation extraction and validation"
    },
    {
        "name": "consolidation",
        "description": "Multi-document consolidation and synthesis"
    }
]

app.openapi_tags = tags_metadata
```

## Testing FastAPI Endpoints

### Test Client Setup
```python
import pytest
from fastapi.testclient import TestClient
from main_simple import app

client = TestClient(app)

def test_analyze_document_endpoint():
    """Test document analysis endpoint."""
    # Create test file
    test_content = "This is a lease agreement between landlord and tenant."
    files = {"document": ("test.txt", test_content, "text/plain")}
    
    response = client.post("/documents/analyze", files=files)
    
    assert response.status_code == 200
    data = response.json()
    assert data["status"] == "success"
    assert data["document_type"] in ["lease_agreement", "contract", "unknown"]
    assert "analysis" in data

@pytest.mark.asyncio
async def test_batch_processing_endpoint():
    """Test batch document processing."""
    documents = [
        ("doc1.txt", "Lease agreement content", "text/plain"),
        ("doc2.txt", "Legal complaint content", "text/plain")
    ]
    
    files = [("documents", doc) for doc in documents]
    response = client.post("/documents/analyze-batch", files=files)
    
    assert response.status_code == 200
    data = response.json()
    assert data["processed_count"] == 2
    assert len(data["results"]) == 2
```

## Reference Files
@api/routes/documents.py
@api/routes/citations.py
@api/routes/consolidation.py
@main_simple.py
@main.py

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/medelman17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
