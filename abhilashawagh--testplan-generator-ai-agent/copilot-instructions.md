## testplan-generator-ai-agent

> "jira_email": "string",

# Project Constitution

## Data Schemas
### Input (UI Form)
```json
{
  "jira_email": "string",
  "jira_api_key": "string",
  "jira_domain": "string",
  "jira_ticket_id": "string",
  "llm_config": {
    "execution_mode": "local | cloud",
    "ollama_host": "string (optional)",
    "model_name": "string"
  }
}
```

### Jira Ticket Data (Fetched)
```json
{
  "key": "string",
  "summary": "string",
  "description": "string",
  "acceptance_criteria": "string (optional)",
  "status": "string"
}
```

### Output Payload
```json
{
  "test_plan_content": "string (markdown based on testplan.pdf template)",
  "ticket_id": "string",
  "status": "success | error",
  "error_message": "string (optional)"
}
```

## Behavioral Rules
- Build deterministic, self-healing automation.
- Prioritize reliability over speed.
- Never guess at business logic.
- Use `.tmp/` for intermediate file operations.
- Separate concerns into the 3-layer A.N.T architecture.

## Architectural Invariants
- **Layer 1 (Architecture):** Technical SOPs in Markdown (`architecture/`). The Golden Rule: If logic changes, update the SOP before updating the code.
- **Layer 2 (Navigation):** Reasoning layer for routing data. Do not perform complex tasks directly; route to execution tools.
- **Layer 3 (Tools):** Deterministic Python scripts in `tools/`. Environment variables in `.env`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abhilashawagh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abhilashawagh)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
