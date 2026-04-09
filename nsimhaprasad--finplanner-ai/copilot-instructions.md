## finplanner-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Multi-Agent Financial Advisor web application built with Python on Google Cloud. The system analyzes NSDL CAS PDF files to provide personalized investment recommendations through a multi-agent architecture.

## Architecture

The system follows a 4-phase implementation with specialized agents:

### Phase 1: Core Functionality
- **CAS_Parser_Agent**: Extracts portfolio data from password-protected PDF files
- PDF parsing using `pdfplumber` library
- Structured JSON output for downstream agents

### Phase 2: Portfolio Analysis  
- **Portfolio_Analysis_Agent**: Analyzes asset allocation and sector concentration
- Uses `pandas` for data manipulation and categorization

### Phase 3: Context Integration
- **Market_Outlook_Agent**: Fetches current market data from financial APIs
- **Risk_Profiler_Agent**: Determines user risk tolerance through questionnaire

### Phase 4: Synthesis & Recommendations
- **Orchestrator_Agent**: Manages workflow and data flow between agents
- **Financial_Advisor_Agent**: Generates personalized investment goals and recommendations

## Key Technologies

- **Backend**: FastAPI or Flask for API endpoints
- **PDF Processing**: `pdfplumber` for extracting text and tables from PDFs
- **Data Analysis**: `pandas` for data manipulation
- **Cloud Services**: Google Cloud (Storage, Functions/Run, Secret Manager)
- **Frontend**: HTML forms with Chart.js for visualizations

## Development Setup

### Local Development
```bash
# Backend development
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload

# Frontend development  
cd frontend
npm install
npm start
```

### Docker Development
```bash
# Run both services
docker-compose up --build

# Backend only
docker build -t finplanner-backend ./backend
docker run -p 8000:8000 finplanner-backend

# Frontend only
docker build -t finplanner-frontend ./frontend  
docker run -p 3000:80 finplanner-frontend
```

### Cloud Deployment
```bash
# Deploy to Google Cloud Run
./deploy.sh

# Or deploy individually
gcloud builds submit --config cloudbuild-backend.yaml .
gcloud builds submit --config cloudbuild-frontend.yaml .
```

## Commands

- `uvicorn app.main:app --reload` - Start FastAPI backend in development mode
- `npm start` - Start React frontend development server
- `npm run build` - Build React frontend for production
- `docker-compose up` - Run full stack locally with Docker
- `./deploy.sh` - Deploy to Google Cloud Run

## Data Flow

1. User uploads NSDL CAS PDF file
2. CAS_Parser_Agent extracts portfolio holdings
3. Portfolio_Analysis_Agent analyzes asset allocation
4. Market_Outlook_Agent fetches current market data
5. Risk_Profiler_Agent determines user risk profile
6. Financial_Advisor_Agent generates recommendations
7. Results displayed in dashboard format

## Security Notes

- Store passwords and API keys in Google Secret Manager
- Handle password-protected PDFs with try/except blocks
- Use IAM for secure service communication
- Default CAS password is typically user's PAN in uppercase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nsimhaprasad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nsimhaprasad)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
