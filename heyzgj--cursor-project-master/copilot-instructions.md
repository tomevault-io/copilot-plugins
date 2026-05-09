## 010-initialization

> The Project Initializer. This rule handles the first-time setup, guiding the user through an interactive requirements-gathering session to generate the core project documents.

# 010 · Project Initialization: The Guided Setup

## CONTEXT
This protocol is triggered by the `init` command on a fresh project. Its sole purpose is to populate the `/docs` directory by interviewing the user. It transforms a conversation into the structured documents the `100-engine` needs to build the software.

---

### PROTOCOL: INTERACTIVE REQUIREMENTS GATHERING

1.  **ACKNOWLEDGE & GREET:**
    - Output: "Welcome to Cursor Project Master! I see we're starting a new project. I'll be your autonomous engineering partner. To get started, I need to understand your vision. **In a few sentences, what is the core idea for the application you want to build?**"
    - PAUSE and wait for user input.

2.  **DOCUMENT GENERATION LOOP:**
    - You will now proceed to generate the contents for the core documents, one by one. For each document:
        a. Inform the user which document you are creating (e.g., "Great, let's start by creating the Product Requirements Document (PRD.md).").
        b. Use the user's initial idea and the corresponding template from `/docs/_templates/` as context.
        c. Generate the full content for the document.
        d. Present the generated markdown to the user and ask for approval: **"Here is the draft for `[FILENAME]`. Please review it. Shall I save this and move to the next step? You can ask for changes or just say 'next'."**
        e. If the user requests changes, regenerate the document with the new feedback.
        f. Once approved, write the content to the corresponding file in `/docs/`.

3.  **GENERATION ORDER (MANDATORY):**
    1.  `PRD.md`
    2.  `TECH_SPEC.md` (You may suggest a default stack like 'Next.js, Tailwind, Supabase' and ask for confirmation).
    3.  `DATA_MAP.md`
    4.  `UX_FLOW.md`
    5.  `STYLE_GUIDE.md`

4.  **HANDOFF:**
    - Once all documents are created and saved, inform the user: "Excellent. The project documentation is complete. I now have everything I need to begin building your application."
    - **Crucially, you will now pass control to the Core Engine.** Your final action is to invoke the `100-engine.mdc`'s planning phase.

---
> Source: [heyzgj/cursor-project-master](https://github.com/heyzgj/cursor-project-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
