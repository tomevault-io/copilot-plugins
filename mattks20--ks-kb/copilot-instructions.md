## ks-kb

> A cost-effective, automated lead discovery pipeline that uses a "Hot-Swappable" configuration strategy. The goal is to change a single JSON spec file (ICP, Industry, Pain Points) to pivot the engine for different clients/industries without rewriting code.

# Project Lumina: Modular AI Lead Generation Engine

## 1. Project Vision
A cost-effective, automated lead discovery pipeline that uses a "Hot-Swappable" configuration strategy. The goal is to change a single JSON spec file (ICP, Industry, Pain Points) to pivot the engine for different clients/industries without rewriting code.

## 2. Tech Stack & Tools
- **Orchestrator:** Python (Async/Await)
- **AI Models:** Anthropic (Claude 3.5 Sonnet/Opus), OpenAI (GPT-4o), Google (Gemini 2.0)
- **Infrastructure:** Google Cloud Platform (Cloud Functions, Cloud Tasks)
- **Workspace:** Google Sheets (as the UI/Database), Google Workspace API
- **Dev Tools:** Claude Desktop (Architect), Claude Code CLI (Engineer), VS Code

## 3. Core Architecture
- `/config/`: Contains `icp_current.json` (The modular brain).
- `/core/`: Contains `scoring.py` (Multi-model consensus), `discovery.py` (Gemini search), and `workspace.py` (Sheets integration).
- `/plans/`: Documentation of implementation steps.

## 4. Current State & Intent
- **PRD Defined:** Modular lead gen with consensus scoring.
- **Goal:** Move from static lists to dynamic AI-scored discovery.
- **Next Task:** Initialize the filesystem, create the `.claude.md` rulebook, and build the Google Sheets authentication layer.

## 5. Coding Standards (for Claude Code)
- Use Service Account JSON for Google Auth.
- Always reference `config/icp_current.json` for targeting parameters.
- Implement a "Consensus Scoring" logic: Leads must be validated by both Claude and a secondary LLM before being pushed to Sheets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mattks20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
