## timeless

> After every prompt from the human user, create a timestamped tracking file in the appropriate `.prompt_tracking` directory with the following structure:


## Prompt Tracking Requirements

After every prompt from the human user, create a timestamped tracking file in the appropriate `.prompt_tracking` directory with the following structure:

### Required File Content
Each tracking file must include:

1. **Timestamp**: ISO format timestamp (YYYY-MM-DDTHH-mm-ss)
2. **Prompt**: The exact user request/question
3. **Context**: Exactly 3 bullet points summarizing:
   - What specific task, document, or system component is being worked on
   - Primary files, directories, or architectural elements relevant to the request
   - Important limitations, requirements, or decision points that affect the work


### File Location Rules
- **Project-scoped prompts**: Place in project root `.prompt_tracking/` directory
- **Specific component prompts**: Place directly in the relevant component's `.prompt_tracking/` directory
- **Example**: If prompt concerns `shared-memory/` designs, use `shared-memory/.prompt_tracking/`



### File Naming Convention
Use format: `YYYY-MM-DDTHH-mm-ss-brief-description.md`

This ensures comprehensive tracking of all user interactions with proper context for future reference.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rappdw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
