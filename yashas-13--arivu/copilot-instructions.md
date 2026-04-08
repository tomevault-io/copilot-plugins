## arivu

> AGENTS.md – Contributor & Agent Guide for Arivu Supply Chain System

AGENTS.md – Contributor & Agent Guide for Arivu Supply Chain System

Overview

This repository implements a modular, FastAPI-based supply chain system for Arivu Foods, allowing distributors and retailers to manage inventory, orders, and authentication. Agents and contributors should focus on modular design, clean API boundaries, and security.

Key Modules

supply_chain_system/inventory/ – Inventory CRUD APIs

supply_chain_system/auth/ – Auth + JWT token issuance

supply_chain_system/database/ – SQLAlchemy setup and models

main.py – FastAPI app entry point, router registration

Development Environment

Activate virtual env: source arivu-venv/bin/activate

Install dependencies: pip install -r requirements.txt

Run server (dev): uvicorn supply_chain_system.main:app --reload

Run server (prod): gunicorn -w 4 -k uvicorn.workers.UvicornWorker supply_chain_system.main:app

Access docs: http://localhost:8000/docs

Testing & Validation

Add logic tests under tests/ directory (planned)

Verify routes using Swagger UI or curl

Run endpoint with token:

curl -H "Authorization: Bearer <token>" http://localhost:8000/inventory/

Lint checks (optional): flake8 supply_chain_system/

Roles & Access

Distributor (was 'admin') – Full access to inventory endpoints

Retailer – Read-only inventory access

Use Depends(get_current_user) for auth dependency. Check roles like:

if current_user.role != "distributor":
    raise HTTPException(403)

How Agents Should Work

Look for AGENTS.md in subfolders to scope tasks

Read routes.py in each module to discover public API

Inspect models.py for data relationships

Follow naming and Pydantic schema conventions from existing code

PR Instructions

Title format: [<module>] <change summary>

Include updated tests (if applicable)

Ensure all imports are scoped and DB sessions closed

Document all new endpoints in OpenAPI auto-docs (FastAPI will handle if annotated properly)

Prompting & Codex Tips

Refer to specific files (e.g., inventory/routes.py, auth/schemas.py)

Include exact models (like InventoryItem) in instructions

Provide steps to reproduce if describing a bug

Break down large tasks into multi-step subtasks if possible

Use this file as a config + context base for contributor agents

Migration Targets (WIP)

Migrate frontend/ to consume /inventory/ and /auth/

Introduce orders/ module with similar CRUD scaffolding

Add real-time notification system (via WebSocket or async tasks)

Summary

This project uses FastAPI + SQLAlchemy in a modular service architecture. Follow this AGENTS.md as shared context for agents and human contributors to maintain clarity, clean PRs, and secure scalable APIs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yashas-13)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yashas-13)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
