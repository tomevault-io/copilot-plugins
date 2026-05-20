## nextjs-supabase-ai-webapp

> Follow these steps sequentially for **every** user request and follow behavioral rules above:


## Request Processing Steps
Follow these steps sequentially for **every** user request and follow behavioral rules above:

1.  **Tool Use Check:** If the user *explicitly* requests a specific tool, execute it using the `<use_tools>` workflow and respond. **Stop processing here.**
2.  **Initial Analysis:**
    * Read the user request carefully.
    * Consult `<documentation>` and instructions for understanding and planning the request
    * Identify the primary goal.
    * Determine the most relevant `<workflow>`. Default to `<research>` if unclear.
    * List affected architecture areas / subsystems. 
    * **Present Analysis:**
        ```
        Request type: [Inferred type]
        Primary Workflow: [<Chosen workflow>]
        Affected Areas: [List areas]
        Plan Overview: [Briefly mention key steps/tools, e.g., "Read docs, formulate prompt for edit_file, run tests"]
        ```
3.  **Task Planning (if complex):**
    * For multi-step workflow (`<develop>`, `<validate>`, `<design>`), break down into sub-tasks.
    * Use `edit_file` to add sub-tasks to `todo.md` (marked ❌). Include development, testing, documentation steps.
    * Inform the user the plan is in `todo.md`.
4.  **Workflow Execution:**
    * State: `Starting workflow: <workflow name>...`
    * Execute steps within the chosen `<workflow>`, focusing on providing **clear, context-rich prompts** to tools like `edit_file` and `run_terminal_cmd` (especially for `gemini` tools).
    * Adhere strictly to `<behavior_rules>`.
    * Review outputs and iterate with follow-up prompts if necessary.
    * State: `Completed Wokflow: <workflow name>.`
5.  **Final Recording (on task completion):**
    * Once the *entire* request is fulfilled:
        * Summarize significant changes.
        * Execute `<record>` workflow (update `ai_changelog.md`, `todo.md`).
        * Ensure `learnings.md` was updated during `<fix>` or `<invent>` workflows if applicable.

---
> Source: [LastBotInc/nextjs-supabase-ai-webapp](https://github.com/LastBotInc/nextjs-supabase-ai-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
