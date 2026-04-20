## exafish

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ExaFish** is a planning and research project exploring how to leverage [MiroFish](https://github.com/666ghj/MiroFish) — an open-source Swarm Intelligence Engine — for cloud computing use cases. This is not a buildable source code project; it is a design/brainstorming workspace.

The primary artifact is `mirofish-brainstorm.md`, which captures use cases, architecture concepts, and open questions.

## What is MiroFish?

MiroFish is a multi-agent simulation platform that builds high-fidelity digital worlds to predict outcomes. It operates in 5 steps: Graph Construction → Environment Setup → Simulation → Report Generation → Deep Interaction.

- GitHub: https://github.com/666ghj/MiroFish
- Docs: https://deepwiki.com/666ghj/MiroFish
- Live Demo: https://666ghj.github.io/mirofish-demo/console

## Target Use Cases

1. **Cloud Computing** — Simulate tenant behavior, cascading failures, autoscaling, cost/performance tradeoffs
2. **Anomaly Detection** — Build baseline simulations, detect divergence from expected agent behavior, generate synthetic anomaly datasets
3. **Predictive "What If" Analysis** — Infrastructure changes, capacity planning, policy simulation, security blast radius estimation

## Open Questions (Unresolved)

See `mirofish-brainstorm.md` for the full list. Key blockers:
- Target cloud platform (AWS / GCP / Azure / on-prem)
- Available seed data sources (logs, metrics, traces, CMDB)
- Operational vs. planning use (real-time anomaly detection vs. batch what-if)
- Self-hosted MiroFish vs. hosted API
- Integration target (standalone vs. existing observability stack)

## Progress Tracking

This project does not yet have a `PROGRESS.md`. If implementation work begins, create one and keep it current after every change per the global CLAUDE.md rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MSKazemi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
