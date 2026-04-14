## kairos-project-legal

> This document provides a high-level overview of the Kairos Project Legal project to help Gemini understand the codebase and assist more effectively.

# Gemini Project Context

This document provides a high-level overview of the Kairos Project Legal project to help Gemini understand the codebase and assist more effectively.

## Project Overview

This project is a legal AI system for analyzing South Korean legal precedents. It uses a multi-agent approach built with LangGraph to process legal queries. The system relies on a pre-existing, vectorized Supabase database of legal precedents. A series of AI agents analyze a user's query, find relevant precedents, and generate a detailed legal opinion including sentencing factors and a final conclusion. The entire process is traced and logged using Langfuse.

## Project Structure

- **`판례데이터/`**: Contains raw legal precedent data in JSON format, for reference purposes.
- **`legal_multiagent/`**: The core multi-agent application.
    - **`agents/`**: Contains individual Python scripts for each AI agent, responsible for specific tasks (e.g., fact extraction, issue generation, conclusion generation).
    - **`graph/`**: Contains the core logic for the LangGraph workflow.
        - **`state.py`**: Defines the `LegalCaseState` TypedDict, which represents the data structure passed between nodes in the graph.
        - **`nodes.py`**: Implements the functions that are executed at each node of the graph. Each function calls one or more AI agents to perform its task.
        - **`workflow.py`**: Assembles the nodes and edges to create the final, compiled LangGraph workflow.
    - **`main.py`**: The main entry point for the application. It initializes the workflow and invokes it with a sample query.
- **`Gemini.md`**: This file.
- **`.gitignore`**: Specifies files to be ignored by Git.
- **`requirements.txt`**: Lists all Python dependencies required for the project.

## Environment Setup

To run this project, you need to create a `.env` file in the root directory with the following environment variables:

```
# OpenAI API Key
OPENAI_API_KEY="sk-..."

# Supabase Project Details
SUPABASE_URL="https://<your-project-ref>.supabase.co"
SUPABASE_SERVICE_KEY="<your-supabase-service-role-key>"

# Langfuse Tracing (Optional)
LANGFUSE_PUBLIC_KEY="pk-lf-..."
LANGFUSE_SECRET_KEY="sk-lf-..."
LANGFUSE_HOST="https://cloud.langfuse.com"
```

## Dependencies

This project relies on Python dependencies listed in `requirements.txt`. You can install them using `pip`:

```bash
pip install -r requirements.txt
```

## How to Run the Project

1.  **Install Python Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

2.  **Run the Multi-agent System**:
    - Execute the main application. This will trigger the LangGraph workflow to process the sample legal case defined in `legal_multiagent/main.py`.
      ```bash
      python -m legal_multiagent.main
      ```

## Key Files and Their Roles

- **`legal_multiagent/main.py`**: The application's entry point. It loads the environment, creates the workflow by calling `create_workflow()`, defines a sample input, and runs the analysis.
- **`legal_multiagent/graph/workflow.py`**: The heart of the application. It imports the state and nodes, assembles the LangGraph workflow by defining all nodes and edges, sets up conditional branching, and compiles the final graph.
- **`legal_multiagent/graph/nodes.py`**: Contains the Python functions that serve as the logic for each node in the graph. These functions orchestrate calls to the various AI agents.
- **`legal_multiagent/graph/state.py`**: Defines the `LegalCaseState`, the central data structure that holds the state of the analysis as it progresses through the graph.
- **`legal_multiagent/agents/classify_legal_domains.py`**: Classifies the legal domain (e.g., civil, criminal, administrative) of a given case.
- **`legal_multiagent/agents/extract_basic_facts.py`**: Extracts the key facts from a legal document.
- **`legal_multiagent/agents/generate_legal_issue.py`**: Generates the legal issues or points of contention from the extracted facts.
- **`legal_multiagent/agents/recommend_relevant_laws.py`**: Recommends relevant legal articles and statutes based on the legal issues and facts.
- **`legal_multiagent/agents/search_similar_precedents_from_supabase.py`**: Searches the Supabase database for similar legal precedents based on facts, issues, and recommended laws, and summarizes them.
- **`legal_multiagent/agents/generate_conclusion_and_sentencing.py`**: Generates the final legal conclusion and, for criminal cases, relevant sentencing factors, based on all prior analysis.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dlrlgns0610)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dlrlgns0610)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
