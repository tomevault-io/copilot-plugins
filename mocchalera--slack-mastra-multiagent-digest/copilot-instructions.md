## slack-mastra-multiagent-digest

> This document is designed to help AI agents understand and work with the `slack-mastra-multiagent-digest` repository.

# AGENTS.md

This document is designed to help AI agents understand and work with the `slack-mastra-multiagent-digest` repository.

## Project Overview

This project is a **Slack Daily Digest** tool powered by **Mastra** multi-agent system.
It aggregates one day's worth of Slack messages from specified channels, analyzes them using multiple specialized agents, and generates a structured daily digest post.

## Architecture

The system consists of the following main components:

1.  **Slack Log Fetcher** (`src/slack/*`):
    - Fetches messages using `@slack/web-api`.
    - Target channels are defined in `SLACK_CHANNEL_IDS`.

2.  **Mastra Agents** (`src/mastra/agents/*`):
    - **TopicAgent**: Extracts main topics and summaries.
    - **DecisionTodoAgent**: Identifies decisions and TODOs.
    - **MoodRiskAgent**: Analyzes team mood and potential risks.
    - **PersonaAgent**: "Editor-in-Chief" that formats the final digest based on a persona.

3.  **Mastra Workflow** (`src/mastra/workflows/dailyDigestWorkflow.ts`):
    - Orchestrates the agents.
    - **Step 1**: Runs analysis agents (Topic, Decision/TODO, Mood/Risk) in parallel.
    - **Step 2**: Passes structured results to PersonaAgent for final text generation.

4.  **Execution Script** (`src/scripts/runDailyDigest.ts`):
    - Entry point for the daily job.
    - Fetches logs -> Runs Workflow -> Posts to Slack (`SLACK_DIGEST_CHANNEL_ID`).

## Key Directories & Files

- `src/mastra/agents/`: Agent definitions and prompts.
- `src/mastra/workflows/`: Workflow definitions.
- `src/scripts/`: Execution scripts (e.g., `runDailyDigest.ts`).
- `src/slack/`: Slack API integration utilities.
- `docs/`: Detailed documentation (Architecture, Agents, Workflow).

## Agent Roles & Outputs

### 1. TopicAgent
- **Goal**: Summarize daily topics.
- **Output**: `topics` (List of topics with title, summary, related channels, importance).

### 2. DecisionTodoAgent
- **Goal**: Extract actionable items.
- **Output**: `decisions` (List of decisions), `todos` (List of tasks with assignee and deadline).

### 3. MoodRiskAgent
- **Goal**: Assess team health.
- **Output**: `mood` (Overall sentiment, notes), `risks` (Potential issues).

### 4. PersonaAgent
- **Goal**: Format the final report.
- **Input**: Outputs from the above 3 agents + `persona` string.
- **Output**: `digest` (Final markdown text for Slack).

## Setup & Execution

### Prerequisites
- Node.js & npm
- `.env` file with:
    - `OPENAI_API_KEY`
    - `SLACK_BOT_TOKEN`
    - `SLACK_CHANNEL_IDS` (comma-separated)
    - `SLACK_DIGEST_CHANNEL_ID`

### Commands
- **Install dependencies**: `npm install`
- **Run Daily Digest (Local)**: `npx ts-node src/scripts/runDailyDigest.ts`
- **Mastra Dev Server**: `npm run dev` (starts `mastra dev`)

## Development Guidelines

- **Modifying Prompts**: Edit the files in `src/mastra/agents/`.
- **Changing Workflow**: Edit `src/mastra/workflows/dailyDigestWorkflow.ts`.
- **New Agents**: Create a new agent in `src/mastra/agents/` and add it to the workflow.
- **Type Safety**: Ensure Zod schemas in agents match the expected outputs.

## Common Tasks for Agents

- **Debugging**: Check `src/scripts/runDailyDigest.ts` to see how the workflow is invoked.
- **Refactoring**: Keep agent logic separated. `PersonaAgent` should only handle formatting, not analysis.
- **Testing**: Use `npx ts-node src/scripts/runDailyDigest.ts` to test end-to-end flow.

---
> Source: [mocchalera/slack-mastra-multiagent-digest](https://github.com/mocchalera/slack-mastra-multiagent-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
