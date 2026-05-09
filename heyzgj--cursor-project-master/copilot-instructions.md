## 400-change

> The Change Controller. A high-priority interrupt handler for managing mid-project scope changes. It ensures the project adapts to new requirements gracefully and systematically.


# 400 · Change Controller: Interrupt, Document, Re-plan

## CONTEXT
This rule is an **interrupt-driven protocol**, triggered exclusively by a direct user command that indicates a change in project requirements (e.g., "Add a new feature," "Change the authentication method," "/replan").

---
### PROTOCOL: THE CHANGE MANAGEMENT LIFECYCLE

**STEP 1: ACKNOWLEDGE & SAFE-PAUSE**
- **Action:**
    1.  Immediately acknowledge the user's request: "Change request received. Pausing current work to process the update."
    2.  If a task is active in `/project/tasks/in_progress/`, execute `mv ./project/tasks/in_progress/T<ID>.md ./project/tasks/todo/T<ID>.md`. This safely returns the task to the `todo` queue for future re-evaluation.
- **Goal:** Ensure the system is in a stable, non-executing state before applying changes.

**STEP 2: DOCUMENT THE CHANGE (Source of Truth Update)**
- **Action:**
    1.  Your first implementation action **MUST** be to modify the source documents in `/docs/`.
    2.  A functional change **MUST** be reflected in `PRD.md`.
    3.  An architectural or technical change **MUST** be reflected in `TECH_SPEC.md`.
    4.  The rationale for this significant change **MUST** be logged as a new entry in `/docs/DECISION_LOG.md` (ADR format).
- **Goal:** Guarantee that documentation always reflects the latest requirements *before* any planning or coding happens.

**STEP 3: TRIGGER RE-PLANNING ENGINE**
- **Action:**
    1.  Inform the user: "Source documents have been updated. I will now invoke the Core Engine to reconcile the project plan."
    2.  Invoke the `100-engine.mdc`'s re-planning logic. The engine is responsible for:
        - Performing an "intelligent diff" between the updated `/docs` and the current `/project/plan.json`.
        - Generating new Epics and Tasks for new requirements.
- **Goal:** Delegate the complex task of re-planning to the specialized engine, keeping this rule lightweight.

**STEP 4: SYNCHRONIZE EXTERNAL MEMORY (Optional)**
- **Action:**
    ```
    {% if env.EXTERNAL_MEMORY == 'true' %}
    # Push the updated decision log to the external memory provider.
    <call memory.write file="/docs/DECISION_LOG.md" namespace="cursor-pm-main"/>
    {% endif %}
    ```
- **Goal:** Ensure that for teams using persistent cross-session memory (like OpenAI's Memory API), the most critical project changes are immediately synchronized.

**STEP 5: COMMUNICATE & RESUME**
- **Action:**
    1.  Report a concise summary of the plan changes to the user (e.g., "Re-plan complete. Plan diff: +3 new tasks, -1 obsolete task.").
    2.  Announce that you are resuming work based on the updated plan by activating the next high-priority task from the `todo` queue.
- **Goal:** Provide a clear confirmation to the user that their change has been fully integrated and that normal operation is resuming.

---
> Source: [heyzgj/cursor-project-master](https://github.com/heyzgj/cursor-project-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
