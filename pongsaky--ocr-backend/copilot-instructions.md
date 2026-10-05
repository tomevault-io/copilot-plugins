## rule

> - **Plan-First Approach:** Before any code change, config update, or documentation edit, AI must generate a detailed plan outlining:

# rule.mdc

## 0. Planning & Verification  
- **Plan-First Approach:** Before any code change, config update, or documentation edit, AI must generate a detailed plan outlining:  
  1. Objectives & scope  
  2. Step-by-step tasks  
  3. Dependencies & estimated effort  
- **User Verification:** After the plan, AI must pause and await explicit user approval or refinement instructions before proceeding with implementation.  
- **Plan Format:** Present plans as a numbered list, with clear headings for each task and subtask.

## 1. General Coding Rules  
- **PEP 8 Compliance:** All Python code must follow PEP 8 style guidelines (naming, indentation, line length, imports).  
- **Docstrings & Types:** Use Google-style docstrings for modules, classes, and functions. Annotate all public signatures with type hints.  

## 2. Environment & Configuration  
- **.env Management:** Never commit a `.env` file. To add or change env vars, update `.env.example` or create a new template file.  
- **Secrets Storage:** Use your chosen secrets management solution (e.g., AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault).

## 3. FastAPI-Specific Guidelines  
- **Project Layout:** Follow FastAPI’s recommended structure (e.g. `app/`, `routers/`, `models/`, `schemas/`).  
- **Routing & Versioning:** Prefix all routes with version (e.g. `/v1/lesson-plans`).  

## 4. Documentation  
- **API Spec:** Keep OpenAPI/Swagger definitions in sync with code. After any change to endpoints, regenerate the spec and update Markdown in `/docs`.  
- **Architecture Diagrams:** Maintain provider-specific architecture diagrams in `/architecture` (e.g., AWS, GCP or generic UML) using official or home-grown templates.  

## 5. Error Handling & Fallbacks  
- **Validation:** On missing or malformed inputs, return a 400 with JSON error schema.  
- **Graceful Recovery:** On ambiguous user requests, prompt for clarification rather than guessing.  
- **Default Fallback:** If an internal error occurs, return a 500 with a generic “Something went wrong” message, and log full details.  

## 6. Hallucination Mitigation & Source Grounding  
- **Citations:** All factual or behavioral rules must reference code comments, `/docs`, or official language/framework docs.  
- **“I’m Unsure”:** If the AI agent cannot verify a fact against the codebase or docs, it must respond with:  
  > “I don’t have enough information to answer that.”  

## 7. Observability, Logging & Telemetry  
- **Structured Logs:** Every AI-generated action (code change, doc update) must emit a JSON log entry with keys:  
  - `timestamp` (ISO 8601)  
  - `action` (e.g. `add-route`, `update-doc`)  
  - `context` (file path, function, etc.)  
  - `status` (`success`/`error`)  
- **Metrics:** Send latency, error-rate, and usage counters to your telemetry platform (e.g., AWS CloudWatch, Prometheus, Datadog, Splunk).

## 8. Automated Testing & Continuous Validation  
- **Test Prompts:** Maintain `tests/ci_prompts.md` with critical usage scenarios. Before merging, run AI agent against all prompts and enforce ≥ 90 % pass rate.  
- **Unit & Integration:** New code must include unit tests (≥ 80 % coverage) and integration tests for external services (e.g., object storage, database, auth flows).

## 9. Versioning & Rule Lifecycle  
- **Version Header:** At top of `rule.mdc`, include `version: MAJOR.MINOR.PATCH` and a short changelog section.  
- **Migration Notes:** For breaking changes, provide migration steps in `/docs/migrations.md`.  

## 10. Security, Privacy & Compliance  
- **PII Handling:** Mask or omit all PII in logs and examples.  
- **Threat Modeling:** Major auth or data-flow changes require a brief threat-model doc in `/security`.  
- **OWASP Best Practices:** Sanitize all inputs and outputs according to OWASP AI Security guidelines.  

## 11. Retrieval-Augmented Generation (RAG) Integration  
- **Doc Lookup:** Before answering API questions, AI must query the `/docs` folder (or your internal knowledge store) via vector search to retrieve relevant snippets.  
- **Code Embedding:** If an existing code snippet matches the request, embed it instead of generating new code.  

## 12. Continuous Learning & Feedback Loops  
- **Feedback Collection:** After each AI response, solicit a thumbs-up/down with optional comment; record feedback in `feedback/log.json`.  
- **Rule Refinement:** Monthly, review low-scoring prompts and update `rule.mdc` accordingly.  

## 13. Path for filesystem MCP by study existing backend project structure
- Using this path "/Users/pongsakon/Study/CEDT/Intern/SCB-Academy/smart_school/afast-smart-school-backend/" and current path is "/Users/pongsakon/Study/CEDT/Intern/nstda-intern/ocr-backend" Be careful about this!!!

---
> Source: [Pongsaky/ocr-backend](https://github.com/Pongsaky/ocr-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
