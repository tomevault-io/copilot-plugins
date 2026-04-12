## mutlifamily-ai-valuation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Development & Build
```bash
npm run dev          # Start development server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript type checking
```

### Docker Commands
```bash
npm run docker:dev   # Start development with Docker Compose
npm run docker:prod  # Start production with Docker Compose  
npm run docker:build # Build Docker images
npm run docker:down  # Stop all Docker services
npm run docker:logs  # View Docker logs
```

### One-click Installation
```bash
./install.sh        # Complete Docker setup with all services
```

## Architecture Overview

This is a **Next.js 15** multifamily property valuation application using the **App Router** architecture. The system processes real estate documents (rent rolls, T12 statements, offering memorandums) and generates comprehensive financial analyses.

### Tech Stack
- **Frontend**: Next.js 15, TypeScript, Tailwind CSS, Shadcn/ui, Radix UI
- **Backend**: Next.js API Routes, NextAuth.js, WebSockets
- **Database**: PostgreSQL (production), Redis (sessions/cache)
- **Infrastructure**: Docker, Docker Compose, Nginx (reverse proxy)
- **AI/Processing**: Python AI processing system, OpenAI integration, Document analysis, Financial modeling

### Key Directories
```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API endpoints (20+ routes)
│   │   ├── auth/          # NextAuth.js authentication
│   │   ├── properties/    # Property CRUD operations
│   │   ├── process/       # Document processing pipeline
│   │   ├── export/        # Multi-format exports (Excel, PDF, PowerPoint)
│   │   ├── websocket/     # Real-time updates
│   │   └── mcp/          # AI processing integration
│   ├── dashboard/         # Property dashboard pages
│   ├── properties/        # Property management UI
│   └── calculator/        # Financial calculators
├── components/            # React components (15+ components)
│   ├── ui/               # Shadcn/ui base components
│   ├── Phase3Dashboard.tsx # Advanced analytics dashboard
│   ├── DocumentPreview.tsx # Document viewer with AI annotations
│   ├── ExportPanel.tsx    # Multi-format export interface
│   └── ProcessingStatusPanel.tsx # Real-time status updates
├── hooks/                 # Custom React hooks (4 hooks)
├── lib/                   # Utility libraries (8 files)
├── types/                 # TypeScript definitions
└── globals.css           # Tailwind CSS with custom CSS variables

ai_processing/             # Python AI processing system
├── src/                   # Python source code
│   ├── main.py           # Main processing pipeline entry point
│   ├── document_processor.py # PDF/Excel document parsing
│   ├── ai_analyzer.py    # AI-powered document analysis
│   ├── financial_modeler.py # Financial modeling and projections
│   ├── report_generator.py # Multi-format report generation
│   └── utils/            # Utility modules (logging, config)
├── config.json           # Processing configuration
├── requirements.txt      # Python dependencies
└── venv/                 # Python virtual environment
```

## Core API Patterns

### Authentication & Users
- `/api/auth/[...nextauth]` - NextAuth.js (Google OAuth + credentials)
- `/api/user/profile` - User profile management
- `/api/user/portfolio` - User property portfolio

### Property Management
- `/api/properties` - CRUD operations for properties
- `/api/properties/[id]` - Individual property management
- `/api/properties/[id]/analysis` - Property financial analysis

### Document Processing
- `/api/process` - Main document processing pipeline
- `/api/files` - File upload/management
- `/api/document-preview` - Generate document previews
- `/api/mcp/process-documents` - AI processing via MCP

### Exports & Generation
- `/api/export` - Multi-format exports (Excel, PDF, PowerPoint, JSON, CSV)
- `/api/generate-pitch-deck` - Professional pitch deck generation
- `/api/download-pitch-deck/[filename]` - File downloads

### Real-time Features
- `/api/websocket` - WebSocket connections for live updates
- `/api/investor-notifications` - Email notification system

## AI Processing System

The Python AI processing system analyzes multifamily property documents and generates comprehensive reports.

### Processing Pipeline
1. **Document Processing** (`document_processor.py`)
   - PDF parsing using pdfplumber and PyPDF2
   - Excel parsing with pandas and openpyxl
   - Rent roll, T12, and offering memo extraction

2. **AI Analysis** (`ai_analyzer.py`)
   - OpenAI integration for document insights
   - Unit mix and rent analysis
   - Occupancy and financial performance analysis
   - Risk assessment and investment grading

3. **Financial Modeling** (`financial_modeler.py`)
   - Multi-year investment projections
   - Cash flow analysis with growth assumptions
   - Returns analysis (cap rates, NOI, appreciation)
   - Sensitivity analysis for key variables

4. **Report Generation** (`report_generator.py`)
   - Excel workbooks with financial models
   - PDF executive summaries
   - PowerPoint pitch decks
   - JSON data exports

### Configuration
- Set `OPENAI_API_KEY` environment variable for AI features
- Configure financial assumptions in `ai_processing/config.json`
- Install dependencies: `pip install -r ai_processing/requirements.txt`

### Usage
```bash
cd ai_processing
source venv/bin/activate
python3 src/main.py --rent-roll path/to/rent_roll.xlsx \
                    --t12 path/to/t12.pdf \
                    --om path/to/offering_memo.pdf \
                    --output-dir /path/to/output \
                    --job-id unique-job-id \
                    --generate-pitch-deck \
                    --include-analysis
```

## Data Models

### Core Types (src/types/)
- **Property**: Complete property data with financial metrics
- **ProcessingJob**: Document processing job tracking
- **PropertyAnalysis**: Financial analysis results and projections
- **ProcessingFile**: File metadata and processing status
- **User**: User management with roles and portfolio data

## Component Architecture

### Advanced Components
- **Phase3Dashboard**: Real-time analytics dashboard with WebSocket integration
- **ProcessingStatusPanel**: Live job status tracking with progress indicators
- **DocumentPreview**: Interactive document viewer with AI annotations
- **ExportPanel**: Multi-format export interface with customization options

### Custom Hooks
- **useWebSocket**: Real-time connection management
- **useProperties**: Property data fetching and management
- **useMCP**: AI processing integration
- **useGoogleDrive**: Google Drive API integration

## Development Workflow

### File Processing Pipeline
1. **Upload** → Validation → File system storage
2. **Processing** → AI extraction → Data analysis
3. **Real-time Updates** → WebSocket → UI components
4. **Results** → Multiple export formats → Download links
5. **Notifications** → Email templates → Investor communications

### Authentication Flow
- NextAuth.js with Google OAuth and credentials providers
- JWT sessions with 30-day expiration
- Role-based access control (admin, user, investor)
- User preferences and portfolio management

## Docker Architecture

### Services
- **app**: Next.js application (port 3000)
- **redis**: Session storage and WebSocket management (port 6379)
- **postgres**: Production database (port 5432)
- **nginx**: Reverse proxy and SSL termination (ports 80/443)

### File Persistence
- `./uploads/` - Document uploads
- `./outputs/` - Processing results
- `./storage/` - Export files
- `./init-db/` - Database initialization scripts

## Key Features

### Document Processing
- Supports rent rolls, T12 statements, offering memorandums
- AI-powered text extraction and financial analysis
- Real-time processing status with WebSocket updates
- Interactive document preview with annotations

### Export System
- Professional PowerPoint pitch decks
- Excel financial models with charts
- PDF executive summaries
- JSON/CSV raw data exports

### Real-time Features
- WebSocket-based live status updates
- Progress tracking for long-running analyses
- Instant notifications for job completion
- Live dashboard updates

## Environment Configuration

### Required Variables
```env
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secure-32-character-secret
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

### Optional Variables
```env
DATABASE_URL=postgresql://user:pass@localhost:5432/db
REDIS_URL=redis://localhost:6379
SMTP_HOST=smtp.gmail.com
OPENAI_API_KEY=your-openai-key
ANTHROPIC_API_KEY=your-anthropic-key
```

## Path Aliases
- `@/*` maps to `./src/*` (configured in tsconfig.json)

## Build Configuration
- **Standalone output** for Docker deployment
- **Health check endpoint** at `/health` → `/api/health`
- **TypeScript strict mode** enabled
- **ESLint** configuration for Next.js

## Testing & Quality
- TypeScript type checking: `npm run type-check`
- ESLint linting: `npm run lint`
- Production build validation: `npm run build`

## Security Considerations
- File path validation and user ownership checks
- API endpoint authentication verification
- Secure file storage and transmission
- Environment variable configuration for secrets
- Role-based access control throughout the application

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kmransom56)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kmransom56)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
