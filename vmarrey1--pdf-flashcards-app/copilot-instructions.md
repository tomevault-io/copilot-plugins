## pdf-flashcards-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PDF Flashcards is a full-stack web application that converts PDF documents (max 5 pages) into interactive flashcards using T5/PyTorch NLP. The frontend is React 19, the backend is Spring Boot 3.2 with Java 17, and NLP processing is handled by a Python FastAPI microservice.

## Supported PDF Types

This application works best with **text-heavy, educational content**. Ideal PDF types include:

### Recommended PDF Content
- **Textbook chapters** - Academic content with clear concepts and definitions
- **Lecture notes** - Structured educational material with key points
- **Study guides** - Summarized content designed for learning
- **Book extracts** - Passages from non-fiction or educational books
- **Research paper abstracts** - Condensed academic summaries
- **Course materials** - Syllabi, handouts, and educational documents
- **Technical documentation** - API docs, tutorials, how-to guides

### PDF Requirements
- **Max 5 pages** - Enforced limit for optimal processing
- **Max 10MB file size** - Upload limit
- **Text-based PDFs** - Must contain selectable/extractable text
- **Clear structure** - Paragraphs, headings, and organized content work best

### Not Recommended
- Scanned images without OCR
- PDFs with primarily tables, charts, or images
- Legal contracts or forms
- PDFs with complex multi-column layouts
- Content in languages other than English

## Build and Run Commands

### Frontend (from `pdf-flashcards-app/`)
```bash
npm install          # Install dependencies
npm start            # Start dev server on http://localhost:3000
npm test             # Run tests with Jest
npm run build        # Production build
```

### Backend (from `pdf-flashcards-app/backend/`)
```bash
mvn clean install    # Build project
mvn spring-boot:run  # Start server on http://localhost:8080
mvn test             # Run tests
```

### Python NLP Service (from `pdf-flashcards-app/nlp-service/`)
```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

### Full Stack Startup Order
1. Start Python NLP service (port 8000)
2. Start Spring Boot backend (port 8080)
3. Start React frontend (port 3000)

## Architecture

```
React (3000) --> Spring Boot (8080) --> Python FastAPI (8000)
                       |                      |
                       +-- PDFBox             +-- T5/PyTorch (Hugging Face)
```

### Frontend Structure (`src/`)
- `App.js` - Main orchestration, state management
- `components/FileUpload.js` - PDF upload with drag-and-drop (5-page limit)
- `components/FlashcardDeck.js` - Flashcard display with Anki export
- `components/Header.js` - Application header
- `config.js` - API endpoints configuration

### Backend Structure (`backend/src/main/java/com/pdfflashcards/backend/`)
- `controller/FlashcardController.java` - PDF processing endpoints
- `controller/ExportController.java` - Anki export endpoint
- `service/FlashcardService.java` - Core business logic
- `service/PdfProcessingService.java` - PDF extraction + 5-page validation
- `service/NlpService.java` - Calls Python NLP microservice via HTTP

### Python NLP Service (`nlp-service/app/`)
- `main.py` - FastAPI application
- `nlp_service.py` - FlashcardGenerator using T5 model
- `models.py` - Pydantic request/response models

## Key API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/process-pdf` | POST | Process PDF, return flashcards |
| `/api/process-pdf-advanced` | POST | Process with maxCards param (1-100) |
| `/api/export/anki` | POST | Export flashcards to Anki format |
| `/api/health` | GET | Service health check |
| `/api/info` | GET | Service version and status |

## Configuration

### Backend (`backend/src/main/resources/application.properties`)
- `nlp.service.url=http://localhost:8000` - Python NLP service URL
- `pdf.max.pages=5` - Maximum PDF pages allowed
- `pdf.max.file.size.mb=10` - Maximum file size
- `flashcard.default.max.cards=20` - Default flashcards to generate
- `flashcard.max.cards.limit=100` - Maximum flashcards allowed

### Frontend (`src/config.js`)
- `REACT_APP_API_URL` - Environment variable for API base URL (defaults to localhost:8080)

## Docker Deployment

### Quick Start with Docker Compose
```bash
# Build and start all services
docker-compose up --build

# Run in background
docker-compose up -d --build

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Individual Container Build
```bash
# Build backend
docker build -t pdf-flashcards-backend ./backend

# Build NLP service
docker build -t pdf-flashcards-nlp ./nlp-service

# Run NLP service
docker run -d -p 8000:8000 --name nlp-service pdf-flashcards-nlp

# Run backend (after NLP is healthy)
docker run -d -p 8080:8080 \
  -e NLP_SERVICE_URL=http://host.docker.internal:8000 \
  --name backend pdf-flashcards-backend
```

### Docker Services
| Service | Port | Health Check |
|---------|------|--------------|
| nlp-service | 8000 | `/health` |
| backend | 8080 | `/api/health` |

### Environment Variables
- `NLP_SERVICE_URL` - URL to NLP service (default: `http://nlp-service:8000`)
- `SPRING_PROFILES_ACTIVE` - Set to `docker` for container deployment

## Deployment

### Frontend (Vercel)
The React frontend is configured for Vercel deployment. See `vercel.json` for configuration.

Set `REACT_APP_API_URL` environment variable in Vercel to your backend URL.

### Backend + NLP Service (Docker)
Deploy containers to any platform:
- **AWS ECS/Fargate** - Use docker-compose or ECS task definitions
- **Railway/Render** - Connect GitHub repo, auto-deploy from Dockerfile
- **DigitalOcean App Platform** - Deploy from docker-compose.yml
- **Self-hosted** - Run docker-compose on any VM

## Constraints

- PDFs must be 5 pages or fewer
- File uploads limited to 10MB
- CORS configured for localhost (update for production)
- Best results with English text content

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vmarrey1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
