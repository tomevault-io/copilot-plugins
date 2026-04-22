## infrax

> InfraX is an intelligent Infrastructure as Code (IaC) orchestration platform powered by "Brahma" - a multi-agent AI system that automates cloud infrastructure management using GPT-4.

# InfraX - Brahma: Intelligent IaC Orchestration Platform

## Project Overview
InfraX is an intelligent Infrastructure as Code (IaC) orchestration platform powered by "Brahma" - a multi-agent AI system that automates cloud infrastructure management using GPT-4.

**Current Version:** 1.0 (Production-Ready for Code Generation)

---

## Brahma Architecture

### Core Components (All Operational ✅)

**1. Intelligent Planner Tool**
- Location: `backend/brahma/tools/intelligent_planner.py`
- Purpose: Analyzes natural language prompts and plans complete infrastructure
- Features:
  - Location-aware cloud provider selection (AWS/Azure/GCP)
  - Automatic region optimization for lowest latency
  - Complete architecture design
  - Service mapping for all components
  - Multi-cloud region knowledge base

**2. Service Selection Agent**
- Location: `backend/brahma/agents/service_selection.py`
- Purpose: Recommends optimal cloud services
- Features:
  - 200+ cloud services across AWS, Azure, GCP
  - Workload-specific recommendations
  - Service comparison and alternatives
  - Best practices application
  - Provider comparison mode

**3. Cost Optimization Agent**
- Location: `backend/brahma/agents/cost_optimization.py`
- Purpose: Analyzes costs and identifies savings (40-60% target)
- Features:
  - AI-powered cost analysis
  - Right-sizing recommendations
  - Reserved instance strategies
  - Storage tiering optimization
  - 12-month cost forecasting
  - Resource scheduling strategies

**4. IaC Generation Agent**
- Location: `backend/brahma/agents/iac_generation.py`
- Purpose: Generates production-ready Terraform code
- Features:
  - Terraform code only (fixed)
  - Security best practices built-in
  - Multi-AZ deployment configurations
  - Auto-scaling setup
  - Monitoring and logging included
  - Supports AWS, Azure, GCP

**5. Diagram Generator Tool**
- Location: `backend/brahma/tools/diagram_generator.py`
- Purpose: Creates visual architecture diagrams
- Features:
  - Mermaid diagram generation (React-compatible)
  - Dark glassmorphism theme matching frontend
  - Network topology visualization
  - Service relationship mapping
  - Data flow diagrams
  - AI-generated detailed service descriptions
  - Interactive hover tooltips with requirements
  - Interactive HTML preview
  - GitHub/GitLab compatible

**6. Cloud Pricing Tool** (NEW ✨)
- Location: `backend/brahma/tools/cloud_pricing.py`
- Purpose: Provides accurate cloud service pricing
- Features:
  - AI-powered pricing estimates (OpenAI GPT-4)
  - Real-time pricing knowledge (up to Jan 2025)
  - AWS, Azure, GCP pricing support
  - Regional pricing variations
  - Instance type recommendations
  - Multi-cloud price comparison
  - Cost breakdown by component (compute, storage, network)
  - Optimization tips included

**7. Unified Orchestrator**
- Location: `backend/brahma/core/orchestrator.py`
- Purpose: Coordinates all agents and tools
- Features:
  - Intelligent workflow mode (auto-planning)
  - Standard workflow mode (manual control)
  - 5-step automated process
  - Workflow history tracking
  - PostgreSQL persistence
  - Complete result packaging

---

## Project Structure

```
InfraX/
├── backend/
│   ├── brahma/                    # Brahma AI System
│   │   ├── agents/
│   │   │   ├── __init__.py
│   │   │   ├── service_selection.py
│   │   │   ├── cost_optimization.py
│   │   │   └── iac_generation.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   └── orchestrator.py
│   │   ├── tools/
│   │   │   ├── __init__.py
│   │   │   ├── intelligent_planner.py
│   │   │   ├── diagram_generator.py
│   │   │   └── cloud_pricing.py
│   │   ├── utils/
│   │   │   └── __init__.py
│   │   └── __init__.py
│   ├── api/
│   │   └── main.py                # FastAPI server + CORS
│   ├── database.py                # PostgreSQL models (SQLAlchemy)
│   ├── templates/
│   │   └── terraform/
│   │       └── aws_base.tf
│   ├── requirements.txt
│   ├── cli.py                     # Demo CLI (not active)
│   └── .env.example
├── frontend/                       # React + TypeScript (NEW ✨)
│   ├── src/
│   │   ├── components/
│   │   │   ├── MermaidDiagram.tsx  # Dark themed diagrams
│   │   │   ├── Sidebar.tsx
│   │   │   └── DashboardLayout.tsx
│   │   ├── pages/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── CreateWorkflow.tsx
│   │   │   ├── Workflows.tsx
│   │   │   └── WorkflowDetails.tsx
│   │   ├── api/
│   │   │   └── brahma.ts          # API client
│   │   ├── types/
│   │   │   └── workflow.ts        # TypeScript definitions
│   │   └── App.tsx
│   ├── package.json
│   └── tailwind.config.js         # Dark glassmorphism theme
├── data/
│   ├── generated_code/            # Terraform files output
│   └── diagrams/                  # Mermaid diagrams + HTML previews
├── PROJECT_VISION.txt             # Complete project vision document
└── CLAUDE.md                      # This file (to be gitignored)
```

---

## Tech Stack

### Backend
- **Runtime:** Python 3.8+
- **Framework:** FastAPI (REST API) + CORS
- **AI:** OpenAI GPT-4 (all agents and tools)
- **IaC Output:** Terraform only
- **Visualization:** Mermaid.js (dark themed)
- **Database:** PostgreSQL (Neon) with SQLAlchemy
- **Storage:** File-based + Database persistence

### Backend Dependencies
```
fastapi
uvicorn
pydantic
openai
python-dotenv
requests
sqlalchemy
psycopg2-binary
alembic
```

### Frontend (NEW ✨)
- **Framework:** React 19 with TypeScript
- **Build Tool:** Vite
- **Visualization:** Mermaid.js with dark theme
- **Styling:** Tailwind CSS v3 (dark glassmorphism)
- **HTTP Client:** Axios
- **Routing:** React Router DOM
- **Fonts:** Space Grotesk + Inter
- **Theme:** Dark mode with glass effects
- **Color Scheme:** Blue primary (#3b82f6), dark backgrounds

### Frontend Dependencies
```
react, react-dom
react-router-dom
axios
mermaid
tailwindcss
typescript
vite
```

---

## Environment Setup

### Required Environment Variables
```bash
# .env file
OPENAI_API_KEY=sk-your-key-here
```

### Setup Commands
```bash
# Backend setup
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Create .env file
cp .env.example .env
# Add your OpenAI API key to .env
```

---

## Workflow Modes

### Mode 1: Intelligent Workflow (Recommended)
**Input:** Natural language prompt + optional location

**Process:**
1. 🧠 Intelligent Planning - Auto-selects cloud provider & region
2. 💰 Cost Optimization - Identifies savings opportunities
3. 📋 Service Refinement - Maps to specific services
4. 🏗️ IaC Generation - Creates Terraform code
5. 🎨 Diagram Generation - Visual architecture (Mermaid)

**Example:**
```python
from brahma.core.orchestrator import BrahmaOrchestrator

brahma = BrahmaOrchestrator()
result = brahma.execute_intelligent_workflow(
    prompt="Build e-commerce platform for Indian users with payment gateway",
    location="India"
)

# Returns:
# - cloud_provider: "aws"
# - region: "ap-south-1"
# - terraform_code: Complete .tf file
# - mermaid_diagram: Architecture visualization
# - estimated_savings: Cost optimization suggestions
```

### Mode 2: Standard Workflow
**Input:** Structured requirements with cloud provider specified

```python
result = brahma.execute_full_workflow({
    "description": "Web application with database",
    "workload_type": "web",
    "scale": "medium",
    "cloud_provider": "aws"  # User specifies provider
})
```

---

## Output Files

### Generated Files Location

**Terraform Code:**
- Path: `/data/generated_code/`
- Format: `{provider}_terraform_{timestamp}.tf`
- Example: `aws_terraform_20241008_143022.tf`

**Mermaid Diagrams:**
- Path: `/data/diagrams/`
- Formats:
  - `.mmd` - Mermaid code (React-compatible)
  - `.html` - Interactive HTML preview
- Example: `aws_diagram_20241008_143022.mmd`

**Workflow Results:**
Complete JSON with all step outputs including:
- Infrastructure plan
- Cost analysis
- Service recommendations
- Generated code
- Diagram code
- Provider selection rationale

---

## Key Features

### ✅ Implemented (V1.5 - Latest)
- Multi-agent AI system with specialized roles
- Natural language infrastructure planning
- Location-aware cloud provider selection
- Automatic region optimization
- Production-ready Terraform generation
- **AI-powered cloud pricing estimates** ✨
- **PostgreSQL database persistence** ✨
- **Dark glassmorphism themed diagrams** ✨
- **Interactive service tooltips with requirements** ✨
- **React + TypeScript frontend** ✨
- Cost optimization (40-60% target)
- Security best practices built-in
- Mermaid diagram generation with dark theme
- React-compatible visualizations
- Multi-cloud support (AWS, Azure, GCP)
- Workflow history and management UI
- Real-time diagram rendering
- CORS-enabled API

### ⏳ Planned (Future Versions)
- Actual deployment orchestration
- Live cloud pricing API integration (AWS/Azure/GCP)
- Infrastructure validation and testing
- Existing infrastructure migration
- Compliance scanning (SOC2, HIPAA, PCI)
- Multi-environment management (dev/staging/prod)
- Disaster recovery planning
- User authentication & authorization
- Multi-user support
- CI/CD pipeline integration

---

## Development Guidelines

### Adding New Agents
1. Create new agent file in `backend/brahma/agents/`
2. Implement agent class with GPT-4 integration
3. Add agent to orchestrator initialization
4. Update workflow steps as needed

### Adding New Tools
1. Create tool file in `backend/brahma/tools/`
2. Implement tool class with specific functionality
3. Integrate with orchestrator if needed
4. Add to relevant workflow steps

### Code Standards
- All agents use GPT-4 for intelligence
- Error handling in all methods
- Return structured dictionaries with `success` field
- Include timestamps in all outputs
- Use type hints for all functions
- Document all public methods

---

## Important Notes

### Current Limitations
- IaC output is Terraform only (CloudFormation/Pulumi removed for simplicity)
- Diagram format is Mermaid only (best for React integration)
- No actual deployment - only code generation
- Cost estimates are AI-based, not live API pricing
- No user authentication yet
- Single-user local development mode

### Production Considerations
Before production deployment, add:
1. User authentication and authorization
2. ~~Database for workflow persistence (PostgreSQL)~~ ✅ IMPLEMENTED
3. Cloud storage for generated files (S3/Azure Blob)
4. Rate limiting and API quotas
5. Monitoring and logging
6. ~~Frontend application~~ ✅ IMPLEMENTED
7. CI/CD pipeline
8. Infrastructure validation
9. SSL/TLS certificates
10. Production database (scale Neon or self-hosted PostgreSQL)

---

## API Endpoints (IMPLEMENTED ✅)

### Active FastAPI Endpoints
```
GET  /                                  - Platform info
GET  /health                            - Health check

POST /api/v1/workflows/intelligent      - Create workflow with AI planning
GET  /api/v1/workflows                  - List all workflows (from PostgreSQL)
GET  /api/v1/workflows/{workflow_id}    - Get specific workflow
```

### Frontend URLs
```
http://localhost:5173/                  - Dashboard home
http://localhost:5173/create            - Create new workflow
http://localhost:5173/workflows         - View all workflows
http://localhost:5173/workflows/{id}    - Workflow details with tabs
```

---

## Performance Targets

- Workflow execution: < 2 minutes end-to-end
- Code generation: 600-1000 lines of Terraform
- Diagram generation: < 30 seconds
- Cost analysis: < 15 seconds
- Service recommendations: < 20 seconds

---

## Innovation Highlights

**What Makes Brahma Unique:**

1. **Multi-Agent Specialization** - 4 specialized agents working together
2. **Location-Aware Intelligence** - Automatic provider/region selection
3. **Full Automation** - Natural language → Production-ready Terraform
4. **Cost-First Approach** - Optimization built into every step
5. **Production-Ready Output** - Security, monitoring, HA included
6. **React Integration** - Mermaid diagrams for easy frontend use

---

## Version History

**v1.5 (Current)** - January 2025 ✨
- React + TypeScript frontend with dark glassmorphism theme
- PostgreSQL database integration (Neon)
- AI-powered cloud pricing estimates
- Dark themed Mermaid diagrams
- Interactive service tooltips with requirements
- Workflow history UI
- CORS-enabled API
- Production-ready full-stack application

**v1.0** - October 2024
- All core agents operational
- Intelligent planner tool
- Diagram generator tool
- Terraform-only output
- Mermaid-only diagrams
- Production-ready for code generation

---

## Support & Documentation

- **Project Vision:** See `PROJECT_VISION.txt` for complete details
- **This File:** `CLAUDE.md` - Comprehensive technical documentation
- **Issues:** Track in GitHub issues (when repository is created)

---

**Last Updated:** January 2025
**Status:** Production-Ready (Full-Stack Application)
**Next Milestone:** User Authentication + Deployment Orchestration + Live Cloud API Integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sushilpandeyy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
