## n8n-orchestration

> This document serves as the central knowledge base and governance protocol for all n8n workflows within this workspace. It establishes the architectural standards, operational rules, and self-healing strategies required to maintain robust and scalable automation systems.

# GEMINI Workflow Protocol

## Overview
This document serves as the central knowledge base and governance protocol for all n8n workflows within this workspace. It establishes the architectural standards, operational rules, and self-healing strategies required to maintain robust and scalable automation systems.

## DOE Architecture Framework

All workflows and automation components must adhere to the **DOE (Directives, Execution, Orchestration)** architecture. This separation of concerns ensures maintainability, clarity, and ease of debugging.

### 1. Directives (The "What" & "Why")
*   **Definition**: Configuration files, rulesets, and high-level logic that define *what* needs to be done.
*   **Components**:
    *   JSON/YAML configuration files.
    *   Global constants and environment variables.
    *   Policy definitions (e.g., "Max retries = 3", "Allowed domains").
*   **Location**: `/directives` directory.

### 2. Execution (The "How")
*   **Definition**: The atomic units of work. These are the scripts, functions, or sub-workflows that perform specific tasks without knowing the larger context.
*   **Components**:
    *   Individual n8n workflow nodes/sub-workflows.
    *   Python/JavaScript scripts called by n8n.
    *   API interaction modules.
*   **Location**: `/executions` directory.

### 3. Orchestration (The "When" & "Where")
*   **Definition**: The control layer that manages the flow of data and control between Directives and Executions. It handles scheduling, state management, and decision-making.
*   **Components**:
    *   Master workflows triggering sub-workflows.
    *   State machines.
    *   Trigger nodes (Cron, Webhook).
*   **Location**: `/orchestration` directory.

## Re-Annealing Strategy (Self-Correction & Learning)

To ensure system resilience, the following **Re-Annealing Strategy** must be implemented across all workflows:

### 1. Error Detection & Capture
*   All critical nodes must have error output paths defined.
*   Errors must be logged with context: `Timestamp`, `WorkflowID`, `NodeID`, `InputData`, `ErrorStack`.

### 2. Analysis & Assessment
*   Workflows should assess the severity of errors (Transient vs. Fatal).
*   *Transient* (e.g., Rate limit, Timeout): Trigger retry logic with exponential backoff.
*   *Fatal* (e.g., Invalid auth, Schema mismatch): Escalate to alerting and halt dependent branches.

### 3. Self-Correction (The Annealing Process)
*   **Dynamic Parameter Adjustment**: If a task fails due to resource constraints, the system should strictly attempt to retry with reduced concurrency or lower batch sizes.
*   **Feedback Loops**: Successful executions should reinforce current parameters; failures should trigger a search for alternative paths or fallback data sources.
*   **Persistent State**: Use external storage (e.g., Redis, Database, local JSON) to recall past failure modes and avoid repeating the same errors immediately.

## Implementation Guidelines
1.  **Directives**: Define your rules in `directives/` (e.g., `scraping_rules.json`).
2.  **Executions**: script your tasks in `executions/` (e.g., `parse_profile.js`).
3.  **Orchestration**: Build your main n8n flow in `orchestration/`, loading directives and calling executions.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naumanch969)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/naumanch969)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
