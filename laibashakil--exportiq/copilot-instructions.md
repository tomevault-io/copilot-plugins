## exportiq

> Generates 3-5 prioritized actions with deadlines

# ExportIQ — Pakistan Textile Export Compliance Agent
### AISeekho 2026 Google Antigravity Hackathon — Challenge 1

---

## Project Overview

An agentic AI system that ingests EU/UK compliance regulations, factory audit reports, and export data to automatically detect compliance gaps, calculate financial risk in PKR, generate prioritized action chains, and simulate execution — protecting Pakistani textile factories from losing billions in export orders.

**Hackathon:** AISeekho 2026 Google Antigravity Hackathon
**Challenge:** Challenge 1 — Autonomous Content-to-Action Agent
**Submission Deadline:** May 20, 2026
**Team Size:** Small team

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Mobile App | Expo (React Native) | Already familiar, Expo Go QR scan sufficient for demo |
| Backend | Python 3.11 + FastAPI | Fast to build, async-friendly for agent pipelines |
| Agent Orchestration | Google Antigravity + LangGraph | Antigravity is mandatory; LangGraph manages multi-agent state |
| LLM | Gemini 2.5 Pro (Vertex AI) | Required for Antigravity; best PDF/doc understanding |
| Database | Firebase Firestore | Real-time updates to mobile app, no server config |
| File Storage | Firebase Storage | Storing uploaded PDFs and generated documents |
| Hosting | Google Cloud Run | Free credits from hackathon, serverless, zero config |
| PDF Parsing | PyMuPDF (fitz) + Gemini | Extract text from audit reports and regulation PDFs |
| Auth | Firebase Auth (anonymous) | Quick demo auth, no signup friction |

---

## Project Structure

```
ExportIQ/
├── CLAUDE.md                          # This file — AI coding assistant instructions
├── README.md                          # Hackathon submission README
│
├── backend/                           # FastAPI backend on Google Cloud Run
│   ├── main.py                        # FastAPI app entry point
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── .env.example
│   │
│   ├── agents/                        # All 6 LangGraph agents
│   │   ├── __init__.py
│   │   ├── orchestrator.py            # Master agent — coordinates all 6
│   │   ├── regulation_agent.py        # Agent 1: Parse EU/UK regulation PDFs
│   │   ├── factory_profile_agent.py   # Agent 2: Parse factory audit CSVs/PDFs
│   │   ├── gap_detection_agent.py     # Agent 3: Cross-reference & find gaps
│   │   ├── financial_impact_agent.py  # Agent 4: Calculate PKR risk exposure
│   │   ├── action_chain_agent.py      # Agent 5: Generate 3-5 prioritized actions
│   │   └── execution_agent.py         # Agent 6: Simulate action execution
│   │
│   ├── tools/                         # Agent tools (Antigravity Skills equivalent)
│   │   ├── pdf_parser.py              # PyMuPDF + Gemini PDF extraction
│   │   ├── csv_processor.py           # Factory data CSV ingestion
│   │   ├── contradiction_detector.py  # Cross-source conflict detection
│   │   ├── document_generator.py      # Generate buyer emails, checklists, reports
│   │   ├── compliance_scorer.py       # 0-100 compliance scoring logic
│   │   └── firestore_client.py        # Firebase read/write helpers
│   │
│   ├── models/                        # Pydantic data models
│   │   ├── factory.py
│   │   ├── regulation.py
│   │   ├── gap_report.py
│   │   └── action_chain.py
│   │
│   ├── api/                           # FastAPI route handlers
│   │   ├── upload.py                  # POST /upload — ingest PDFs/CSVs
│   │   ├── analyze.py                 # POST /analyze — trigger full agent pipeline
│   │   ├── actions.py                 # GET /actions/{factory_id} — get action chain
│   │   ├── simulate.py                # POST /simulate — run execution simulation
│   │   └── status.py                  # GET /status/{job_id} — poll agent progress
│   │
│   └── mock_data/                     # Mock inputs for demo
│       ├── factories/
│       │   ├── fwi_audit.pdf      # Mock factory audit report
│       │   ├── cfw_audit.pdf
│       │   └── factory_export_data.csv       # Pakistan-Textile-Mills-Council-style export volumes
│       └── regulations/
│           ├── eu_cbam_rules.pdf             # Real EU CBAM PDF (publicly available)
│           ├── uk_modern_slavery_act.pdf
│           └── eu_supply_chain_directive.pdf
│
├── mobile/                            # Expo React Native app
│   ├── app.json
│   ├── package.json
│   ├── App.js
│   │
│   ├── screens/
│   │   ├── SplashScreen.js             # First-launch splash
│   │   ├── HomeScreen.js               # Compliance score dashboard + deadlines badge
│   │   ├── ComplianceScreen.js         # Per-regulation breakdown (Status tab)
│   │   ├── ActionCenterScreen.js       # Agent-generated action items (Fix It tab)
│   │   ├── DocumentVaultScreen.js      # Upload PDFs, view generated docs (Documents tab)
│   │   ├── AgentTraceScreen.js         # Live Antigravity reasoning trace log (dev route)
│   │   ├── UploadScreen.js             # PDF upload + auto-trigger analysis
│   │   ├── AnalysisProgressScreen.js   # Animated progress while pipeline runs
│   │   ├── HowItWorksScreen.js         # Scoring + compliance explainer
│   │   ├── EditEmailScreen.js          # Compose/edit a generated buyer email
│   │   ├── SettingsScreen.js           # User preferences + cross-factory export
│   │   ├── DeadlinesScreen.js          # Upcoming gap deadlines across factories
│   │   └── BuyerCommsScreen.js         # (legacy file, not routed in App.js)
│   │
│   ├── components/                     # Score cards, action items, risk badge, agent
│   │                                   # status bar, contradiction alert, dashed empty
│   │                                   # score, info tooltip, interactive checklist,
│   │                                   # simulation reveal, etc.
│   │
│   ├── services/
│   │   ├── api.js                     # Calls to FastAPI backend
│   │   ├── firebase.js                # Firestore real-time listeners + writes
│   │   ├── notifications.js           # Expo push notifications
│   │   └── notificationsRead.js       # AsyncStorage read-set for deadline badges
│   │
│   └── constants/
│       ├── colors.js
│       └── config.js                  # Backend URL, Firebase config, DEMO_FACTORIES
│
├── antigravity/                       # Antigravity agent definitions
│   └── .agent/
│       ├── skills/                    # 8 skill.md files (one per Antigravity skill)
│       │   ├── regulation_parser/skill.md       # Parse EU/UK regulation PDFs
│       │   ├── factory_profile/skill.md         # Parse factory audit data
│       │   ├── gap_detector/skill.md            # Detect compliance gaps
│       │   ├── contradiction_detector/skill.md  # Find conflicting claims
│       │   ├── financial_impact/skill.md        # Calculate PKR risk
│       │   ├── action_chain_generator/skill.md  # Generate prioritized actions
│       │   ├── execution_simulator/skill.md     # Simulate action execution
│       │   └── document_drafter/skill.md        # Generate buyer emails/reports
│       └── workflows/
│           ├── full_compliance_analysis.md   # End-to-end analysis workflow
│           └── daily_regulatory_scan.md      # Daily new regulation check workflow
│
└── docs/
    ├── architecture.md                # System architecture for README
    ├── agent_trace_example.md         # Example agent reasoning trace
    └── demo_script.md                 # Step-by-step demo flow for judges
```

---

## Environment Variables

Create `backend/.env` based on `backend/.env.example`:

```env
# Google / Gemini
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_APPLICATION_CREDENTIALS=./service-account.json
GEMINI_MODEL=gemini-2.5-pro

# Firebase
FIREBASE_PROJECT_ID=your-firebase-project-id
FIREBASE_STORAGE_BUCKET=your-project.appspot.com

# App
ENVIRONMENT=development
MAX_PDF_SIZE_MB=20
AGENT_TIMEOUT_SECONDS=120
```

---

## Agent Pipeline (How the 6 Agents Work Together)

```
User uploads Factory PDF + Regulation PDF
              ↓
    [Orchestrator Agent]
    Spins up all agents, manages state via LangGraph
              ↓
    ┌─────────────────────────────────┐
    │  Agent 1: Regulation Ingestion  │  → Extracts rules, deadlines, limits
    │  Agent 2: Factory Profile       │  → Extracts current compliance status
    └─────────────────────────────────┘
              ↓ (both complete)
    [Agent 3: Gap Detection]
    Cross-references rules vs factory status
    Flags contradictions (factory claims X, data shows Y)
              ↓
    [Agent 4: Financial Impact]
    Calculates PKR value of at-risk orders
    Identifies buyer concentration risk
              ↓
    [Agent 5: Action Chain]
    Generates 3-5 prioritized actions with deadlines
              ↓
    [Agent 6: Execution Simulation]
    Simulates each action
    Shows before/after compliance score
    Generates output documents
              ↓
    Results saved to Firestore
    Mobile app updates in real time
```

---

## Core Data Models

### Factory Compliance Report
```python
{
  "factory_id": "fwi_fsd_001",
  "factory_name": "Faisal Weave Industries",
  "city": "Faisalabad",
  "compliance_score": 43,           # 0-100
  "risk_level": "CRITICAL",         # CRITICAL / WARNING / COMPLIANT
  "orders_at_risk_pkr": 340000000,
  "buyers_affected": ["NordStyle Group", "BritMart Retail"],
  "gaps": [
    {
      "regulation": "EU CBAM",
      "requirement": "Carbon declaration filing",
      "status": "MISSING",
      "severity": "CRITICAL",
      "deadline": "2026-01-01",
      "days_remaining": 231
    }
  ],
  "contradictions": [
    {
      "claim": "Factory claims ISO 14001 compliance",
      "evidence": "Water audit March 2025 shows non-conformance",
      "source_a": "factory_self_report.csv",
      "source_b": "water_audit_march25.pdf",
      "confidence": 0.91
    }
  ],
  "action_chain": [...],
  "simulation_result": {...}
}
```

### Action Item
```python
{
  "action_id": "act_001",
  "priority": 1,
  "title": "File CBAM Carbon Declaration",
  "description": "Submit carbon declaration for all EU-bound shipments",
  "effort": "HIGH",
  "deadline": "2025-12-01",
  "impact_pkr": 280000000,       # How much risk this action mitigates
  "status": "PENDING",           # PENDING / SIMULATED / EXECUTED
  "simulation_output": {
    "document_generated": "cbam_declaration_faisal_weave.pdf",
    "compliance_score_delta": +18,
    "risk_reduction_pkr": 200000000
  }
}
```

---

## Antigravity Skills — What to Define

Each file in `antigravity/.agent/skills/` tells Antigravity when and how to use that capability.

### Example: `regulation_parser/skill.md`
```markdown
# Skill: Regulation Parser

## When to use
Use this skill when you need to extract compliance requirements,
deadlines, numerical limits, or certification requirements from
EU or UK regulatory PDF documents.

## What this skill does
1. Extracts all compliance rules as structured JSON
2. Maps rules to factory attributes (chemicals, labour, carbon, audit certs)
3. Identifies deadlines and grace periods
4. Flags Pakistan-specific applicability (which rules apply to exporters)

## Input
- PDF file path or text content of regulation document

## Output
- Structured compliance rulebook JSON
- List of deadlines sorted by urgency
- Pakistan applicability flags per rule

## Tools used
- pdf_parser tool
- Gemini 2.5 Pro for rule extraction
- Firestore for caching parsed regulations
```

---

## API Endpoints

```
POST /upload
  Body: { file: PDF/CSV, type: "regulation"|"factory_audit"|"export_data", factory_id: str }
  Returns: { file_id, status: "uploaded" }

POST /analyze
  Body: { factory_id: str, regulation_ids: [str] }
  Returns: { job_id: str, status: "running" }

GET /status/{job_id}
  Returns: { status: "running"|"complete"|"failed", progress: 0-100, current_agent: str }

GET /report/{factory_id}
  Returns: Full compliance report with gaps, contradictions, action chain

POST /simulate/{factory_id}
  Body: { action_ids: [str] }  # Which actions to simulate
  Returns: { before_score, after_score, risk_reduction_pkr, documents_generated: [str] }

GET /actions/{factory_id}
  Returns: Just the prioritised action chain (action_chain[])

GET /documents/{factory_id}
  Returns: List of generated documents (buyer emails, checklists, CBAM forms)

POST /documents/{factory_id}/audit-ready
  Returns: Bundled audit-ready document set for a single factory

POST /failure-test/{job_id}
  Body: { agent: str, failure_type: "api_timeout"|"missing_data"|"contradiction" }
  Returns: Recovery agent trace — FOR DEMO FAILURE INJECTION

GET /export-summary?factory_ids=fwi_fsd_001,cfw_lhe_002
  Returns: Cross-factory CSV/markdown export — risk + gaps + actions in one file
```

---

## Firebase Firestore Schema

```
/factories/{factory_id}
  - name, city, created_at
  - compliance_score (updated in real time)
  - risk_level
  - orders_at_risk_pkr

/factories/{factory_id}/reports/{report_id}
  - full compliance report object
  - gaps array
  - contradictions array
  - created_at

/factories/{factory_id}/actions/{action_id}
  - action item object
  - status (real-time updates as simulation runs)

/regulations/{regulation_id}
  - name, jurisdiction, parsed_rules
  - last_updated

/jobs/{job_id}
  - status, progress, current_agent
  - factory_id, started_at
  - agent_trace array (appended by each agent as it runs)
```

---

## 5-Day Build Plan

### Day 1 — Backend Foundation
- [ ] Set up FastAPI project, Dockerfile, Cloud Run config
- [ ] Set up Firebase project (Firestore + Storage + Auth)
- [ ] Implement `pdf_parser.py` tool using PyMuPDF + Gemini
- [ ] Implement `csv_processor.py` for factory export data
- [ ] Load mock regulation PDFs (CBAM publicly available at taxation.ec.europa.eu)
- [ ] Load mock factory audit data (create realistic JSON/PDF)
- [ ] Test Gemini 2.5 Pro PDF extraction end-to-end

### Day 2 — Multi-Agent Pipeline
- [ ] Implement Agent 1 (Regulation Ingestion) + Agent 2 (Factory Profile)
- [ ] Implement Agent 3 (Gap Detection) with contradiction logic
- [ ] Wire agents together in LangGraph with shared state
- [ ] POST /analyze endpoint triggering the pipeline
- [ ] GET /status endpoint with real-time progress
- [ ] Verify agent traces are being logged to Firestore

### Day 3 — Impact + Action + Simulation
- [ ] Implement Agent 4 (Financial Impact — PKR risk calculation)
- [ ] Implement Agent 5 (Action Chain — 3-5 prioritized actions)
- [ ] Implement Agent 6 (Execution Simulation — score delta, doc generation)
- [ ] `document_generator.py` — CBAM form, buyer email, checklist PDF
- [ ] POST /simulate endpoint
- [ ] Failure injection endpoint (POST /failure-test) for demo

### Day 4 — Mobile App
- [ ] Expo project setup, Firebase SDK, API service layer
- [ ] HomeScreen — compliance score card, risk badge, top actions
- [ ] ComplianceScreen — per-regulation breakdown list
- [ ] ActionCenterScreen — action cards with Simulate button
- [ ] AgentTraceScreen — live reasoning trace from Firestore
- [ ] Real-time Firestore listeners (score updates as agents run)
- [ ] Push notifications for deadline alerts

### Day 5 — Polish + Demo Engineering
- [ ] Antigravity Skills + Workflows files (the `.agent/` folder)
- [ ] Seed 3 mock factories with different risk levels (Critical/Warning/Compliant)
- [ ] Rehearse failure injection during demo
- [ ] Record 90-second backup video
- [ ] Write README.md (architecture, Antigravity usage, assumptions, limitations)
- [ ] Write `docs/demo_script.md` — exact judge-facing flow

---

## Demo Script (Judge-Facing Flow)

```
0:00 - Open mobile app, show HomeScreen with 3 factories loaded
0:20 - Select "Faisal Weave Industries" — score shows 43/100, PKR 340M at risk (RED)
0:40 - Show ComplianceScreen — 4 gaps highlighted, 1 contradiction card
1:00 - Tap contradiction: "Factory claims ISO 14001 — water audit disagrees"
1:20 - Switch to laptop, show Antigravity Manager view
       6 agents visible, each with their Artifacts
1:50 - Tap "Run Full Analysis" on mobile
       Agents fire in parallel — watch Manager view live
2:30 - Analysis complete. Score updated to 43. Action chain appears.
2:50 - Tap Action 1 "Simulate CBAM Filing"
       Score jumps: 43 → 61. Risk drops: 340M → 180M PKR
3:10 - Tap "Simulate All Actions"
       Score: 43 → 71. Risk: 340M → 60M PKR
3:30 - FAILURE INJECTION: hit POST /failure-test, kill CertVerify booking API
       Show Recovery Agent kicking in on Manager view
       Fallback: manual booking template generated instead
       Agents continue — pipeline does not break
4:00 - Show generated documents: CBAM form, buyer email, audit checklist
4:20 - Show AgentTraceScreen on mobile — full reasoning visible
4:40 - Close with: "15 million jobs depend on these exports.
       One missed deadline ends hundreds of them.
       ExportIQ prevents that."
```

---

## Antigravity-Specific Notes

- Open Antigravity Manager view on your laptop during the demo — this is mandatory for judges to see the multi-agent orchestration
- Each agent must produce at least one **Artifact** visible in Manager view (a plan document, a gap report, a generated email)
- Define all 6 Skills in `.agent/skills/` before the demo — judges will check this folder
- The agent trace logs in Firestore should mirror what's visible in Antigravity Manager view
- Antigravity requires **personal Gmail** — Workspace/org accounts are not supported in preview
- Have a **Gemini API key backup** via Google AI Studio in case Vertex AI quota runs out during demo

---

## Mock Data to Prepare

```
mock_data/factories/fwi_audit.pdf
  - SA8000 certificate: expired 4 months ago
  - ISO 14001: claimed compliant (CONTRADICTION with water audit)
  - Chemical discharge: 12 ppm reported (EU limit: 8 ppm)
  - CBAM declaration: missing
  - Working hours: manual paper logs (unverifiable)
  - Export value: PKR 340M to NordStyle Group and BritMart Retail

mock_data/factories/cfw_audit.pdf
  - All certificates valid
  - Score: 78/100 (WARNING level — some gaps but not critical)
  - Orders at risk: PKR 45M

mock_data/factories/rgl_audit.pdf  
  - Fully compliant
  - Score: 91/100 (COMPLIANT)
  - Good demo contrast case

mock_data/regulations/eu_cbam_rules.pdf
  - Download real PDF from: taxation.ec.europa.eu/carbon-border-adjustment-mechanism
  
mock_data/regulations/uk_modern_slavery_act.pdf
  - Download real PDF from: legislation.gov.uk/ukpga/2015/30
```

---

## Evaluation Criteria Mapping

| Criterion | Weight | How This Project Covers It |
|---|---|---|
| Google Antigravity Integration | 25% | 6 agents in Manager view, Skills folder defined, Workflows defined, Artifacts produced by every agent |
| Agentic Reasoning & Workflow | 20% | Multi-step pipeline, contradiction detection, constraint-based prioritization, failure recovery with fallback |
| Insight & Decision Quality | 20% | Specific PKR figures, named regulations, contradiction evidence cited with sources, non-trivial gaps |
| Action Simulation & Outcome | 15% | Score delta shown, PKR risk reduction calculated, documents generated, before/after state visible |
| Technical Implementation | 10% | FastAPI + LangGraph + Firebase + Expo, clean separation of agents/tools/models, failure handling |
| Innovation & UX | 10% | No existing solution in Pakistan, mobile-first compliance tool, financial stakes in PKR make it visceral |

---

## Key URLs & Resources

- EU CBAM Rules PDF: https://taxation.ec.europa.eu/carbon-border-adjustment-mechanism_en
- UK Modern Slavery Act: https://www.legislation.gov.uk/ukpga/2015/30/contents
- Google Antigravity Docs: https://developers.google.com/antigravity
- Gemini API (AI Studio): https://aistudio.google.com
- Vertex AI Setup: https://cloud.google.com/vertex-ai/docs/start/introduction-unified-platform
- LangGraph Docs: https://langchain-ai.github.io/langgraph/
- Firebase Setup: https://firebase.google.com/docs/web/setup
- Expo Docs: https://docs.expo.dev
- Cloud Run Deploy: https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-python-service
- Pakistan Textile Mills Council (Pakistan textile data): https://aptma.org.pk

---

## Commands to Get Started

```bash
# Clone and setup backend
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env      # Fill in your keys
uvicorn main:app --reload

# Setup mobile
cd mobile
npm install
npx expo start            # Scan QR with Expo Go app

# Deploy backend to Cloud Run
cd backend
gcloud builds submit --tag gcr.io/YOUR_PROJECT/ExportIQ
gcloud run deploy ExportIQ --image gcr.io/YOUR_PROJECT/ExportIQ --platform managed
```

---

*Built for AISeekho 2026 Google Antigravity Hackathon*
*Challenge 1: Autonomous Content-to-Action Agent*

---
> Source: [laibashakil/ExportIQ](https://github.com/laibashakil/ExportIQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
