## sentinel

> A multi-agent AI system that detects satellite collision risks and autonomously coordinates avoidance maneuvers. The demo shows two satellites on a collision course, agents reasoning through the problem, and a final avoidance decision — all visible in a live UI.

# Autonomous Orbital Traffic Control — CLAUDE.md

## Project Goal
A multi-agent AI system that detects satellite collision risks and autonomously coordinates avoidance maneuvers. The demo shows two satellites on a collision course, agents reasoning through the problem, and a final avoidance decision — all visible in a live UI.

## Tech Stack
- **Backend**: Python 3.11+, FastAPI, LangGraph (agent pipeline), WebSockets
- **Frontend**: React + Vite, Three.js (orbit canvas)
- **Agent Model**: `claude-sonnet-4-6` via Anthropic SDK
- **Data**: JSON mock files only — no real orbital APIs

## Agent Responsibilities

| Agent | Role |
|---|---|
| `tracking_agent` | Ingests ConjunctionEvents; reasons about urgency and severity |
| `prediction_agent` | Contextualises risk — considers satellite priority, time window, debris implications |
| `optimization_agent` | Calls `simulate_maneuver()` for 3 delta-v options; reasons about trade-offs |
| `negotiation_agent` | Decides which satellite moves and which maneuver to execute; key differentiator |
| `governance_agent` | Validates the decision against hard safety rules; sets `validated: true/false` |

## Deterministic / Agent Boundary Rule

This is the most important architectural rule. **Never mix these layers.**

### DETERMINISTIC LAYER (tools/ — no LLM)
- `orbital_sim.py` computes everything with pure Python math
- `get_conjunction_events()` reads mock data → returns `ConjunctionEvent` objects
- `compute_collision_probability(sat_a, sat_b)` → returns `float 0–1`
- `simulate_maneuver(sat_id, delta_v)` → returns new predicted trajectory dict
- `get_kessler_cascade_events()` → returns cascading event list for stretch demo
- **Any function returning a float or dataclass MUST live in `tools/`, never in an agent**

### AGENT LAYER (agents/ — LLM reasoning only)
- Agents receive structured dataclasses as input
- Agents produce natural language reasoning + structured decisions as output
- **Agents never compute numbers — they interpret and decide**
- Agents never call each other directly — all communication goes through orchestrator state

## Key Conventions
- All inter-agent communication flows through the LangGraph state dict
- `orbital_sim.py` has zero imports from `agents/`
- Every agent appends a reasoning string to `state["agent_log"]`
- WebSocket streams partial updates as each agent completes — never wait for full pipeline
- The demo must work end-to-end with zero real API calls except Anthropic

## Negotiation Policy
- GPS satellites (priority 1) never maneuver unless no other option exists
- Decision is based on: priority, fuel remaining, controllability, operator policy

## Governance Rules (hard constraints)
- Miss distance after maneuver must be > 5 km
- Fuel cost must not exceed 30% of remaining fuel
- Uncontrollable debris cannot be assigned maneuvers

## Running the Project
```bash
# Backend
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000

# Frontend
cd frontend
npm install
npm run dev
```

Or use `docker-compose up` from the project root.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DinhPhucLe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
