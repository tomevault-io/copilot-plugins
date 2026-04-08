## brain

> The basic concept will be a "Goal" that will be analyzed and reported on in a "run". Each "run" will have its own working directory in runs subdirectory.

# Multi-Agent Framework

The basic concept will be a "Goal" that will be analyzed and reported on in a "run". Each "run" will have its own working directory in runs subdirectory.

## Core Concepts

*   **Goal**: A statement or question that will need to be analyzed and reported on.
*   **WorkItem**: A task to be completed by an agent. It has a `description` and a `result`.
*   **Run**: Manages the state for a given `Goal`. It tracks all `WorkItem`s and other necessary state.
*   **Agent Persona**: A definition of an agent's capabilities, expertise, and personality, stored as a heading in the `agents.md` file.

## Run Directory & Artifacts

Each run will have a unique directory inside the `runs` directory. The directory name will be prefixed with a timestamp in the format `YYYYMMDD_hhmmss_<short_goal>`, where `<short_goal>` is a brief, 10-20 character version of the goal that best describes it.

Within this directory, the following files will be generated:

*   **`transcript.md`**: A detailed log of all agent interactions. This includes not just the results of each `WorkItem`, but also the detailed internal monologue, discussions, and step-by-step reasoning of the agents as they perform their tasks.
*   **`report.md`**: The final, user-facing report. This is a comprehensive, boardroom-quality document that leads with a strong executive summary and includes detailed sections on methodology, macroeconomic analysis, sector deep dives, company profiles, risk analysis, and the final strategic recommendation.

## Staged Workflow

1.  **Initiation**:
    *   The user provides a `GOAL:` prompt.
    *   All Agent Personas from `agents.md` are loaded.
    *   A `Run` is initiated, and its unique directory is created.

2.  **Pre-Read Materials (Optional)**:
    *   **Process**: The framework checks for files in the top-level `/info/` directory.
    *   If files exist, the framework lists them and asks the user to select which ones the agents should review for the current run.
    *   The content of the user-selected files is ingested and becomes the first entry in the `transcript.md`, providing a foundational context available to all agents throughout the run.
    *   **Output**: A "Pre-Read Materials" section at the top of the `transcript.md`.

3.  **Stage 1: Strategic Direction (C-Suite Agents)**
    *   **Agents**: `ChiefExecutiveOfficer`, `ChiefOperatingOfficer`, etc.
    *   **Process**: The C-Suite agents collaborate to analyze the `GOAL`, define the overall strategy, and establish success metrics. Their detailed discussion is logged.
    *   **Output**: A refined goal and a set of high-level strategic objectives.

3.  **Stage 2: Tactical Breakdown (Project Management Agents)**
    *   **Agents**: `SeniorProjectManager`, `JuniorProjectManager`.
    *   **Process**: The `SeniorProjectManager` breaks down the strategic objectives into major epics. The `JuniorProjectManager` then decomposes those epics into a granular, actionable list of `WorkItem`s with assigned agents.
    *   **Output**: A comprehensive list of `WorkItem`s.

4.  **Stage 3: Execution (Worker Agents)**
    *   **Agents**: Specialized workers (e.g., `Researcher`, `Analyst`).
    *   **Process**: Worker agents execute `WorkItem`s in sequence. All actions, including the agent's internal reasoning and the data gathered, are logged to `transcript.md`.
    *   **Output**: A list of completed `WorkItem`s with their detailed results.

5.  **Stage 4: Synthesis & Reporting (Report-Writer Agents)**
    *   **Agents**: `JuniorReportWriter`, `SeniorReportWriter`.
    *   **Process**: The writers synthesize the entire `transcript.md` to construct the initial draft of the detailed `report.md`.
    *   **Output**: A draft `report.md` file ready for review.

6.  **Stage 5: C-Suite Review & Feedback (Iterative Loop)**
    *   **Agents**: `ChiefExecutiveOfficer`, `ChiefOperatingOfficer`, etc.
    *   **Process**: The C-Suite agents review the generated `report.md` against the strategic objectives defined in Stage 1.
        *   **If the report is approved:** The run is considered complete.
        *   **If the report requires revision:** The C-Suite provides specific feedback, asking for additional information, deeper analysis, or clarification. This feedback serves as a new set of high-level objectives.
    *   **Output**: Either final approval of the `report.md` or a list of feedback points and new requirements.

7.  **Iteration Loop**
    *   If the C-Suite provides feedback, the process loops back to **Stage 2: Tactical Breakdown**.
    *   The Project Management agents will take the C-Suite's feedback and create new `WorkItem`s to address them.
    *   The workflow then proceeds through **Stage 3 (Execution)** and **Stage 4 (Synthesis & Reporting)** again, generating a revised `report.md`.
    *   This iterative process continues until the C-Suite gives their final approval in **Stage 5**.

## Execution Trigger

To initiate a run, the user will provide a prompt starting with the key phrase `GOAL:`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aliafshar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aliafshar)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
