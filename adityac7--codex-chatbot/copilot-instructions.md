## codex-chatbot

> **Objective:** Build and deploy a scalable, production-ready RAG chatbot on AWS using proprietary data, supporting multiple LLMs, with VTION branding, developed component-by-component.

# Production RAG Chatbot on AWS: Development Plan (Concise)

**Objective:** Build and deploy a scalable, production-ready RAG chatbot on AWS using proprietary data, supporting multiple LLMs, with VTION branding, developed component-by-component.

**Core Principles:** Modular, User Control, Iterative Development (especially for prompts), Comprehensive (yet concise) Documentation, Detailed Logging, Best Practices, Security, VTION Branding (`#00afaf`, `#0c0e2d`, light theme).

---

## Phase 0: Project Setup & Foundational Configuration

**Goal:** Establish environment, AWS services, GitHub, LLM access, and project structure.

* [ ] **Task 0.1: Environment & Tooling**
    * [ ] Confirm local dev setup (IDE, Git).
    * [ ] Standardize: **Python 3.11**.
    * [ ] List/confirm library versions (FastAPI/Flask, Boto3, Frontend framework).
    * [ ] **AI Action:** Verify with user.

* [ ] **Task 0.2: AWS Setup/Verification**
    * [ ] Confirm AWS account & IAM permissions for: S3, Bedrock, Lambda, API Gateway, Athena/Redshift Spectrum, IAM, CloudWatch, (Optional: ECS, EKS, Amplify).
    * [ ] **AI Action:** Prompt user for AWS config/entitlement confirmation. Document services.

* [ ] **Task 0.3: GitHub & CI/CD Foundation**
    * [ ] Create private GitHub repo.
    * [ ] Define branching strategy (e.g., `main`, `develop`, `feature/*`).
    * [ ] Basic GitHub Actions: linting, placeholder for build/test/deploy.
    * [ ] **AI Action:** Guide repo creation & initial workflow.

* [ ] **Task 0.4: LLM Access & Key Management**
    * [ ] Confirm Bedrock access: **Claude 3 Sonnet** (default).
    * [ ] Confirm access/keys for optional LLMs (OpenAI, Gemini).
    * [ ] Secure API key management (AWS Secrets Manager or env vars). **No hardcoded keys.**
    * [ ] **AI Action:** Confirm LLM access; discuss key management.

* [ ] **Task 0.5: Define Core Technologies (Revisit)**
    * [ ] Backend: FastAPI (recommended) or Flask.
    * [ ] Frontend: React with Vite (recommended) or HTML/JS/Tailwind.
    * [ ] AWS SDK: Boto3 (latest).
    * [ ] **AI Action:** Discuss choices; document.

* [ ] **Task 0.6: Project Directory Structure**
    * [ ] Define structure (backend, frontend, data, scripts, docs, .github).
        ```
        /project-root
        ├── /backend (app, Dockerfile, requirements.txt)
        ├── /frontend (public, src, package.json, Dockerfile)
        ├── /data (sample data, S3 upload scripts, generated_context/)
        ├── /scripts (helpers, deployment)
        ├── /docs
        ├── .github/workflows
        ├── .gitignore
        └── README.md
        ```
    * [ ] **AI Action:** Propose; get approval.

---

## Phase 1: Data Understanding & Context Generation

**Goal:** Analyze data, build modular data loader, implement "first-time context & dictionary builder."

* [ ] **Task 1.1: Analyze Sample Data (S3)**
    * [ ] User provides sample data/schema.
    * [ ] Identify types, patterns, PII. Storage format in S3 (CSV, JSON, Parquet).
    * [ ] **AI Action:** Request sample/schema. Discuss S3 format.

* [ ] **Task 1.2: Design Modular S3 Data Access (Athena/Spectrum)**
    * [ ] Python module for S3 data interaction (via Athena/Spectrum).
    * [ ] Configurable for S3 paths & data formats.
    * [ ] **AI Action:** Propose design.

* [ ] **Task 1.3: Implement Initial Data Profiling**
    * [ ] Script: list columns, infer types, basic stats (via Athena/Spectrum).
    * [ ] **AI Action:** Code utility.

* [ ] **Task 1.4: Develop "First-Time Context & Dictionary Builder"**
    * [ ] **Concept:** One-time LLM process (Claude 3 Sonnet) on data subset (schema, sample rows) for:
        * ~500-word **domain context**: Overview of data, purpose, common query types, business rules.
        * **Data dictionary**: Column names, detailed descriptions, data types (Athena/Presto specific), table relationships, value distributions/enums where applicable, sample valid values.
    * [ ] **Implementation:** Script to fetch schema/sample, prompt LLM, store context/dictionary (e.g., JSON/YAML in S3 `/generated_context/` or DynamoDB). Ensure efficient loading/caching by backend.
    * [ ] **AI Action:** Design prompt, write script, discuss storage & efficient access.

---

## Phase 2: Backend Skeleton - API & Core Logic

**Goal:** Setup backend app, API endpoints, LLM module, text-to-SQL, data querying.

* [ ] **Task 2.1: Design Core API Endpoints**
    * [ ] `/query` (POST), `/capabilities` (GET), `/config/llm` (GET/POST, Admin), `/admin/context/regenerate` (POST, Admin).
    * [ ] **AI Action:** Define OpenAPI/Swagger spec.

* [ ] **Task 2.2: Setup Basic Backend App (FastAPI/Flask)**
    * [ ] Init framework, "hello world" endpoint, `requirements.txt`, `Dockerfile`.
    * [ ] **AI Action:** Provide boilerplate.

* [ ] **Task 2.3: Implement LLM Interaction Module (`llm_clients.py`)**
    * [ ] Unified interface for LLMs.
    * [ ] **Sub-tasks:** Bedrock (Claude 3 Sonnet), optional OpenAI, optional Gemini. Secure key handling.
    * [ ] Easy model switching via config.
    * [ ] **AI Action:** Code module, start with Bedrock.

* [ ] **Task 2.4: Develop Text-to-SQL Component**
    * [ ] Takes user query, loads comprehensive context/dictionary (Task 1.4).
    * [ ] **Advanced Prompt Engineering:**
        * Use detailed context (schema, descriptions, relationships, sample values/queries, business rules).
        * Provide few-shot examples of good text-to-SQL pairs if necessary.
        * Clear instructions on SQL dialect (Athena/Presto), functions to use/avoid.
        * Security constraints (e.g., generate read-only queries, avoid specific clauses if risky).
    * [ ] Prompts LLM (Claude 3 Sonnet) for SQL.
    * [ ] **Robust SQL validation/sanitization:** e.g., allowlisting commands/patterns, parsing query structure, checking for DML or harmful operations, regex checks.
    * [ ] **AI Action:** Design prompt, implement component with validation.

* [ ] **Task 2.5: Implement AWS Data Querying (Athena/Redshift Spectrum)**
    * [ ] Executes SQL, retrieves results.
    * [ ] **Crucial:** Limit to 50 rows for LLM. Aggregate if >50 rows. No raw DB info to next LLM.
    * [ ] Ensure query execution IAM roles have least privilege access to data.
    * [ ] **AI Action:** Code module.

* [ ] **Task 2.6: Implement Result Processing & Formatting (LLM-based)**
    * [ ] Takes query data (max 50 rows), original query.
    * [ ] Prompts LLM (Claude 3 Sonnet) for natural language answer & graphical representation suggestion (e.g., `{type: 'bar', data: {...}}`).
    * [ ] **Strictly no raw table data in final response.**
    * [ ] **AI Action:** Design prompt, implement component.

* [ ] **Task 2.7: Implement Robust Logging Framework**
    * [ ] Python `logging` module, structured JSON logs (for CloudWatch).
    * [ ] Logs: timestamp, level, component, correlation ID, clear messages (mind PII).
    * [ ] **Strictly maintain detailed logging.**
    * [ ] **AI Action:** Setup config, integrate into components.

---

## Phase 3: Frontend Skeleton - User Interface

**Goal:** Develop basic UI with VTION branding.

* [ ] **Task 3.1: Choose Frontend Framework (Confirm 0.5)**
    * [ ] Confirm (React, Vue, or HTML/JS/Tailwind). Setup project.
    * [ ] **AI Action:** Provide boilerplate.

* [ ] **Task 3.2: Design Basic UI Layout**
    * [ ] Chat interface: input, conversation history, graphics area.
    * [ ] VTION branding: colors (`#00afaf`, `#0c0e2d`), light theme (e.g., `#f8f9fa`), logo. Responsive.
    * [ ] **AI Action:** Create HTML/CSS or React components.

* [ ] **Task 3.3: Implement "Capabilities Screen/Initial Message"**
    * [ ] On load/help: explain data, example questions, limitations. Static or dynamic.
    * [ ] **AI Action:** Implement.

* [ ] **Task 3.4: Implement Chat Input & Display Components**
    * [ ] User query input. Display user/bot messages (text & graphics placeholder).
    * [ ] **AI Action:** Develop UI components.

* [ ] **Task 3.5: Integrate Frontend with Backend API (`/query`)**
    * [ ] JS/TS: send query, receive response, display text, pass graphics data to stub.
    * [ ] Handle API states (loading, success, error).
    * [ ] **AI Action:** Implement API integration.

* [ ] **Task 3.6: Apply VTION Branding & Styling**
    * [ ] Ensure UI adheres to VTION colors/theme. Clean font.
    * [ ] **AI Action:** Refine CSS/styling.

---

## Phase 4: Connecting Pipeline & Initial End-to-End Testing

**Goal:** Integrate backend components, connect frontend for basic E2E pipeline.

* [ ] **Task 4.1: Backend: Integrate Context with Text-to-SQL**
    * [ ] Task 2.4 uses context/dictionary from Task 1.4. Test.
    * [ ] **AI Action:** Verify, debug.

* [ ] **Task 4.2: Backend: Integrate Text-to-SQL with Data Querying**
    * [ ] Task 2.4 SQL output to Task 2.5. Verify aggregation/50-row limit.
    * [ ] **AI Action:** Verify, debug.

* [ ] **Task 4.3: Backend: Integrate Query Results with LLM for Final Response**
    * [ ] Task 2.5 data to Task 2.6. Verify answer/graphic suggestion.
    * [ ] **AI Action:** Verify, debug.

* [ ] **Task 4.4: Full End-to-End Test (Manual)**
    * [ ] UI query -> Backend (Text-to-SQL -> Data Query -> Result Processing) -> UI response.
    * [ ] **Log extensively.**
    * [ ] **AI Action:** Guide testing, analyze logs.

* [ ] **Task 4.5: Implement Graphical Representation Stubs (Frontend)**
    * [ ] Placeholders for charts (log or display raw data).
    * [ ] **AI Action:** Implement stubs.

---

## Phase 5: Enhancements & Production Readiness

**Goal:** Add graphics, error handling, admin console, prepare for production (security, scalability).

* [ ] **Task 5.1: Develop Graphical Representation Component (Frontend)**
    * [ ] Charting library (Chart.js, D3.js, Recharts). Render chart from backend data. Responsive, VTION colors.
    * [ ] **AI Action:** Implement.

* [ ] **Task 5.2: Implement Robust Error Handling & User Feedback**
    * [ ] **Backend:** Global error handlers, meaningful errors/HTTP codes (no sensitive details to user).
    * [ ] **Frontend:** Graceful API error handling, user-friendly messages.
    * [ ] **AI Action:** Review, enhance.

* [ ] **Task 5.3: Build Basic Admin Console (Optional)**
    * [ ] Secured interface: view logs, trigger context regen, config LLM, view usage stats.
    * [ ] Secure appropriately.
    * [ ] **AI Action:** Design, implement.

* [ ] **Task 5.4: Security Hardening**
    * [ ] Input validation, SQLi prevention (validate LLM SQL per Task 2.4), API auth, least privilege IAM for all AWS services (including Bedrock model invocation).
    * [ ] Secure data handling, dependency scanning. Review Bedrock security best practices.
    * [ ] **AI Action:** Security review.

* [ ] **Task 5.5: Scalability & Performance**
    * [ ] **Backend:** Async (FastAPI), Lambda/API Gateway or ECS for scaling. Efficient loading/caching of context (Task 1.4).
    * [ ] **Data Querying:** Optimize Athena/Spectrum. Partition S3 data.
    * [ ] **LLM Calls:** Retries, timeouts.
    * [ ] **Frontend:** CDN for static assets.
    * [ ] **AI Action:** Review, suggest scaling strategies.

* [ ] **Task 5.6: Comprehensive CI/CD Pipeline (GitHub Actions)**
    * [ ] **Backend/Frontend:** Lint, test, build, deploy to staging/prod (Docker, ECR, Lambda/ECS, S3/Amplify).
    * [ ] **AI Action:** Implement CI/CD.

---

## Phase 6: Documentation & Logging Refinement

**Goal:** Finalize comprehensive documentation and perfect logging.

* [ ] **Task 6.1: Create User Manual**
    * [ ] Access, usage, examples, graphics, troubleshooting.
    * [ ] **AI Action:** Draft.

* [ ] **Task 6.2: Create Developer & Operations Documentation**
    * [ ] Architecture, setup, component details (including prompt strategies), data schema/context, deployment, troubleshooting, LLM config.
    * [ ] **Detailed for non-programmers.**
    * [ ] **AI Action:** Compile/refine throughout project.

* [ ] **Task 6.3: Finalize and Test Logging**
    * [ ] Review logs (CloudWatch). Ensure clarity, consistency, correlation IDs. No sensitive PII.
    * [ ] **AI Action:** Review logging.

---

## Recurring Tasks (For Each Component/Major Step)

* [ ] **R.1: Code:** Write. **Self-Reflect:** Correct, consistent, aligned? Prompts effective?
* [ ] **R.2: Unit Test:** Write, pass.
* [ ] **R.3: Push to GitHub:** Feature branch, PR.
* [ ] **R.4: CI Checks:** Pass.
* [ ] **R.5: Peer Review:** (Optional).
* [ ] **R.6: Deploy to Staging (AWS):** CI/CD or manual.
* [ ] **R.7: Test on Staging:** Integration, UAT. Verify, check logs, test various prompts.
* [ ] **R.8: User Approval:** On staging.
* [ ] **R.9: Merge:** To `develop`/`main`.
* [ ] **R.10: Deploy to Production (AWS).**
* [ ] **R.11: Update Documentation.**

---
This task list is a living document.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adityac7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adityac7)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
