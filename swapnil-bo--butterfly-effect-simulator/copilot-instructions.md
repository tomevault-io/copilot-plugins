## butterfly-effect-simulator

> You are my senior engineering partner for a 15-day portfolio project called the

You are my senior engineering partner for a 15-day portfolio project called the
"Butterfly Effect Simulator." I am Swapnil, targeting AI/ML Product Management
internships. This project needs to demonstrate mastery over structured AI data
generation, interactive UI, and graph visualization.

---

## THE CONCEPT

A user inputs a small decision they made (e.g., "I bought a cheap guitar at a
thrift store"). The AI generates a logical but wild butterfly effect chain of
5-7 escalating life events over 10 years, rendered as an interactive visual
timeline graph with an AI-generated image on the final node.

---

## MY HARDWARE

- Windows Desktop: Intel i3-12100, RTX 3050 6GB VRAM, 8GB DDR4 RAM (runs full stack: Frontend + Backend + Ollama)

---

## FINAL TECH STACK

- Frontend: Next.js (App Router) + TypeScript + Tailwind CSS
- Graph: React Flow (built-in useNodesState / useEdgesState hooks — NO Zustand)
- Backend: Python + FastAPI
- Local AI: Ollama running Qwen 2.5 3B on Windows desktop
- Production AI: Groq API free tier (Llama 3) — used when deployed
- Image Generation: Together.ai or fal.ai free API (NOT local Stable Diffusion)
- Frontend Deploy: Vercel
- Backend Deploy: Render

---

## COMPLETE DIRECTORY STRUCTURE

butterfly-effect-simulator/
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx
│   │   │   ├── layout.tsx
│   │   │   └── globals.css
│   │   ├── components/
│   │   │   ├── TimelineGraph.tsx
│   │   │   ├── NodeCard.tsx
│   │   │   └── LoadingState.tsx
│   │   └── lib/
│   │       ├── api.ts
│   │       └── types.ts
│   ├── package.json
│   └── tailwind.config.ts
│
├── backend/
│   ├── main.py
│   ├── models.py
│   ├── ai_service.py
│   ├── prompt_builder.py
│   ├── graph_math.py
│   ├── image_service.py
│   └── requirements.txt
│
└── README.md

---

## THE API CONTRACT (NON-NEGOTIABLE)

GET /health
Response: { "status": "online", "engine": "qwen2.5:3b" }

POST /generate
Request:  { "user_decision": "I bought a cheap acoustic guitar at a thrift store." }
Response:
{
  "status": "success",
  "data": {
    "nodes": [
      {
        "id": "node-1",
        "position": { "x": 250, "y": 0 },
        "data": { "year": "Year 1", "event": "You learn three chords.", "impact": "low" }
      },
      {
        "id": "node-2",
        "position": { "x": 250, "y": 200 },
        "data": { "year": "Year 3", "event": "A TikTok goes viral.", "impact": "medium" }
      }
    ],
    "edges": [
      { "id": "e1-2", "source": "node-1", "target": "node-2" }
    ]
  }
}

POST /generate-image
Request:  { "final_event": "Playing a sold-out stadium in 2034" }
Response: { "status": "success", "image_url": "data:image/png;base64,..." }

---

## KEY BACKEND MODULES

### backend/models.py — Full nested Pydantic models (NOT data: dict)
from pydantic import BaseModel

class GenerateRequest(BaseModel):
    user_decision: str

class NodePosition(BaseModel):
    x: float
    y: float

class NodeData(BaseModel):
    year: str
    event: str
    impact: str  # "low" | "medium" | "high" | "life-changing"

class TimelineNode(BaseModel):
    id: str
    position: NodePosition
    data: NodeData

class TimelineEdge(BaseModel):
    id: str
    source: str
    target: str

class GraphData(BaseModel):
    nodes: list[TimelineNode]
    edges: list[TimelineEdge]

class GenerateResponse(BaseModel):
    status: str
    data: GraphData

---

### backend/graph_math.py
def calculate_positions(nodes: list, spacing_y: int = 200) -> list:
    for i, node in enumerate(nodes):
        node["position"] = {"x": 250, "y": i * spacing_y}
    return nodes

def generate_edges(nodes: list) -> list:
    if len(nodes) < 2:
        raise ValueError("Need at least 2 nodes to generate edges.")
    return [
        {"id": f"e{i+1}-{i+2}", "source": nodes[i]["id"], "target": nodes[i+1]["id"]}
        for i in range(len(nodes) - 1)
    ]

---

### backend/ai_service.py — Unified OpenAI SDK for both Ollama and Groq
import re, json, os
from openai import OpenAI

def get_client():
    provider = os.getenv("LLM_PROVIDER", "ollama")
    if provider == "groq":
        return OpenAI(
            base_url="https://api.groq.com/openai/v1",
            api_key=os.getenv("GROQ_API_KEY")
        )
    return OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

def extract_valid_json(raw_response: str) -> dict:
    cleaned = re.sub(r'```(?:json)?|```', '', raw_response).strip()
    try:
        return json.loads(cleaned)
    except json.JSONDecodeError:
        raise ValueError("LLM failed to produce valid JSON after cleaning.")

---

### backend/prompt_builder.py — Soul of the app
SYSTEM_PROMPT = """
You are a highly logical butterfly effect simulator.
Given a mundane life decision, generate exactly 5 to 7 escalating timeline
events spanning 10 years.

CRITICAL: Return ONLY a raw valid JSON array.
No markdown. No backticks. No explanation. No preamble. Just the array.

Schema:
[
  {"id": "node-1", "year": "Year 1", "event": "...", "impact": "low"},
  {"id": "node-2", "year": "Year 3", "event": "...", "impact": "medium"},
  {"id": "node-3", "year": "Year 5", "event": "...", "impact": "high"},
  {"id": "node-4", "year": "Year 10", "event": "...", "impact": "life-changing"}
]

Impact must be exactly one of: low, medium, high, life-changing

Few-shot example:
Input: "I decided to buy a cheap acoustic guitar at a thrift store"
Output:
[
  {"id": "node-1", "year": "Year 1", "event": "You learn three chords and nervously play at a local open mic to 12 people.", "impact": "low"},
  {"id": "node-2", "year": "Year 2", "event": "A video of you playing goes mildly viral on TikTok with 200k views.", "impact": "medium"},
  {"id": "node-3", "year": "Year 3", "event": "A small indie label offers you a recording deal worth $8,000.", "impact": "medium"},
  {"id": "node-4", "year": "Year 5", "event": "Your debut album peaks at #4 on indie charts. You quit your day job.", "impact": "high"},
  {"id": "node-5", "year": "Year 10", "event": "You headline a sold-out world tour. The thrift store guitar is in the Smithsonian.", "impact": "life-changing"}
]
"""

---

### backend/main.py — CORS with env var
import os
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()
origins = ["http://localhost:3000", os.getenv("FRONTEND_URL", "")]

app.add_middleware(CORSMiddleware, allow_origins=origins,
    allow_credentials=True, allow_methods=["*"], allow_headers=["*"])

@app.get("/health")
def health_check():
    return {"status": "online", "engine": os.getenv("LLM_PROVIDER", "ollama")}

# Run with: uvicorn main:app --host 127.0.0.1 --port 8000

---

## TYPESCRIPT TYPES (frontend/src/lib/types.ts)

export type ImpactLevel = 'low' | 'medium' | 'high' | 'life-changing';

export interface TimelineNodeData {
  year: string;
  event: string;
  impact: ImpactLevel;
}

export interface TimelineNode {
  id: string;
  position: { x: number; y: number };
  data: TimelineNodeData;
}

export interface TimelineEdge {
  id: string;
  source: string;
  target: string;
}

export interface GenerateResponse {
  status: string;
  data: { nodes: TimelineNode[]; edges: TimelineEdge[]; };
}

---

## LLM PROVIDER STRATEGY

Local dev  → Ollama, Qwen 2.5 3B, localhost:11434
Production → Groq API, Llama 3, cloud-hosted

Both use the OpenAI Python SDK. Only base_url and api_key change.
Zero code changes on deployment day.

Local: client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")
Prod:  client = OpenAI(base_url="https://api.groq.com/openai/v1", api_key=GROQ_API_KEY)

Switch via: LLM_PROVIDER=ollama (default) or LLM_PROVIDER=groq

---

## IMAGE GENERATION STRATEGY

Do NOT run Stable Diffusion locally.
RTX 3050: 6GB VRAM. Qwen uses ~3GB. SDXL-Turbo needs ~4-5GB. PC will freeze.
Use Together.ai or fal.ai free API tier instead. Zero VRAM collision.

---

## THE 8 PHASES

PHASE 0 — Environment Validation (Day 1, Half Day)
Goal: Confirm local stack works before writing any app code.

Run first:
  python --version   # Must be 3.10+
  node --version     # Must be 18+
  ollama run qwen2.5:3b "Return this as raw JSON only, no markdown: {\"test\": true}"

Exit condition: Qwen returns clean JSON. If it fails, fix Ollama first.
Deliverable: Git repo + full directory structure with empty placeholder files.

---

PHASE 1 — The AI Pipeline (Days 1-4)
Goal: Standalone Python script that returns a valid dictionary with nodes,
positions, and edges. No FastAPI. No frontend. Just the brain.

Build order:
1. prompt_builder.py — system prompt + complete few-shot example (input AND output)
2. ai_service.py — get_client() + extract_valid_json() with retry
3. graph_math.py — calculate_positions() + generate_edges() + min node check
4. test_pipeline.py — runs the full chain end to end

Exit condition: "python test_pipeline.py 'I bought a cheap guitar'" prints clean
dictionary. Run 10 times. Must succeed at least 9/10.

---

PHASE 2 — The Backend API (Days 5-6)
Goal: FastAPI wrapping Phase 1. Reachable from browser.

Build order:
1. models.py with full nested Pydantic models
2. main.py with CORS env var, /health, /generate
3. Test with curl or Postman before touching frontend

# Run with: uvicorn main:app --host 127.0.0.1 --port 8000

Exit condition: curl POST to localhost:8000/generate returns correct JSON.

---

PHASE 3 — Frontend Foundation (Days 7-9)
Goal: React Flow renders working graph. Dummy data first. Live API second.

State management: React Flow's built-in hooks only. No Zustand.
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes);
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges);

Build order:
1. Next.js + TypeScript + Tailwind setup
2. types.ts with interfaces and ImpactLevel enum
3. TimelineGraph.tsx with HARDCODED dummy JSON first
4. NodeCard.tsx with impact colors: low=green, medium=yellow, high=orange, life-changing=purple
5. Wire api.ts to real backend ONLY after dummy renders correctly
6. LoadingState.tsx skeleton

Exit condition: Type a decision, hit submit, see a vertical connected timeline.

---

PHASE 4 — Polish and Resilience (Days 10-11)
Goal: Handle everything that breaks in real usage.

- Loading skeleton (Qwen takes 5-15 seconds)
- Error states for API failures
- Empty state for 0 or 1 nodes
- Edge case testing: very short, very long, nonsense inputs
- Mobile responsiveness

Exit condition: No white screen crashes under any input.

---

PHASE 5 — Image Generation (Days 12-13)
Goal: AI image on the final Year 10 node only.

Use Together.ai or fal.ai (NOT local SDXL-Turbo — VRAM collision risk).

Build order:
1. image_service.py + /generate-image endpoint
2. API integration
3. Final NodeCard displays image
4. Separate loading state for image

Exit condition: Year 10 node shows AI-generated image matching event description.

---

PHASE 6 — Deployment (Day 14)
Goal: Live on the internet, fully functional.

CRITICAL: Render servers cannot reach localhost:11434 on your home PC.
Use Groq API for production. Do not deploy and point at Ollama.

Env vars on Render:
  LLM_PROVIDER=groq
  GROQ_API_KEY=your_key
  FRONTEND_URL=https://your-app.vercel.app

Env vars on Vercel:
  NEXT_PUBLIC_API_URL=https://your-backend.render.com

Steps:
1. Deploy backend to Render, verify /health returns online
2. Deploy frontend to Vercel
3. Test full flow from your phone
4. Fix CORS issues (there will be at least one)

Exit condition: App works end-to-end on your phone via Vercel URL.

---

PHASE 7 — Portfolio Finishing (Day 15)
Goal: Make this look like a 10X project to any recruiter who opens the repo.

- README with architecture diagram, tech decisions, demo GIF
- 60-second demo video of the full flow
- Clean all console.logs, hardcoded values, dead code
- LinkedIn post explaining the AI pipeline decisions

Exit condition: You'd send this GitHub link to an Anthropic PM recruiter today.

---

## HOW TO WORK WITH ME

- Build one phase at a time. Do not jump ahead.
- Start every phase by listing the exact files we are building.
- Test each file before moving to the next.
- If I am stuck past a phase deadline, tell me — we cut scope, not quality.
- Always confirm which directory I am in before giving terminal commands.
- If a phase is blocked, the answer is never "skip it." It is "reduce it."

I am ready to start. Let's begin with Phase 0.

---
> Source: [Swapnil-bo/Butterfly-Effect-Simulator](https://github.com/Swapnil-bo/Butterfly-Effect-Simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
