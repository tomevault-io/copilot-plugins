## knowledge-keeper-agent

> **Product Name:** KnowledgeKeeper

# KnowledgeKeeper — Claude Code Project Instructions
### Nukode | AI Automation Agency

---

## PROJECT OVERVIEW

**Product Name:** KnowledgeKeeper
**Builder:** Nukode (nukode.co.uk)
**Product Type:** Three-stage AI agent system for institutional knowledge capture during employee departures
**Current Phase:** MVP Build

KnowledgeKeeper is a multi-agent system that captures critical institutional knowledge when an employee leaves a business. It operates across three sequential stages:

- **Stage 1** — Business Interview (Manager/HR configures the agent)
- **Stage 2** — Employee Interview (departing employee is interviewed by the agent)
- **Stage 3** — Document Generation (synthesised handover pack is produced)

The Role Intelligence Profile produced in Stage 1 is passed as structured context into Stage 2 before the employee session begins. Stage 3 is triggered automatically on Stage 2 completion.

---

## CORE ARCHITECTURE PRINCIPLES

**1. Stages are isolated conversation sessions**
Stage 1 and Stage 2 must never share a live conversation thread. They are separate LangGraph graph instances, each with their own session ID. The Role Intelligence Profile is the only data passed between them — via a database lookup, never via a shared context window.

**2. The Role Intelligence Profile is the system's spine**
Every decision in Stage 2 — which blocks to run at full depth, which to run lightly, what tone to use, which risk categories to prioritise — is derived from the Role Intelligence Profile. Treat it as the single source of truth for a session. Its Pydantic schema must be strictly validated before Stage 2 activates.

**3. One question at a time — always**
Both Stage 1 and Stage 2 agents must never output more than one question per turn. This is a hard constraint enforced at the node level, not just in the prompt. Add an output validation step that checks for multiple question marks and re-routes if detected.

**4. Follow-up logic is a separate classification step**
Do not rely on the main LLM to decide whether a follow-up is needed. Use a lightweight classifier node (Claude Haiku) that receives the last question and answer and returns a structured decision: `{needs_followup: bool, reason: str, suggested_followup: str}`. The main graph uses this to route to a follow-up node or advance to the next question.

**5. Risk flags run as a parallel branch**
In Stage 2, risk flag detection runs as a parallel LangGraph branch on every answer. It does not block the main conversation flow. It appends to a `risk_flags: List[RiskFlag]` field in the state. These are surfaced in Section 2 of the final document.

**6. Stage 3 is a single synthesis call**
Do not stream Stage 3 section by section. Compile the full context (Role Intelligence Profile + Stage 1 transcript summary + Stage 2 full transcript + risk flags) and generate the complete document in one call. This gives the LLM full context for coherent cross-referencing between sections.

---

## PROMPT FILES

All prompts live in `prompts.py` within each stage directory. They are never hardcoded inline in graph nodes. Prompts are constructed dynamically using the Role Intelligence Profile at runtime.

The full prompt content for all three stages is documented in:
`knowledge-extraction-agent-prompt_1.md` (repo root)

Key prompt construction rules:
- Stage 2 system prompt is assembled at session start by injecting the serialised Role Intelligence Profile into the base prompt template
- Agent instruction flags from the profile are appended as a `## BEHAVIOURAL INSTRUCTIONS FOR THIS SESSION` block at the end of the Stage 2 system prompt
- Block question lists are injected as the relevant block node is entered — not all at once at session start

---

## DEVELOPMENT CONVENTIONS

**Naming**
- Node functions: `snake_case` with `_node` suffix e.g. `greeting_node`, `risk_flag_classifier_node`
- State fields: `snake_case`
- Block names: match exactly the nine knowledge categories defined in `constants.py`
- Graph files: one graph per file, graph object always named `graph`

**Error Handling**
- If the LLM returns a malformed response at any node, retry once with an explicit format instruction before raising
- If Stage 1 profile validation fails, return the specific validation errors to the session and ask the manager to clarify — do not silently fill defaults
- If the follow-up classifier call fails, default to `needs_followup: false` and advance — never block the main conversation for a classifier failure

**Testing**
- Every block in Stage 2 must have a unit test with a sample answer set covering: clear answer (no followup needed), vague answer (followup needed), refusal (move on), and partial answer (flag as gap)
- Stage 3 document generation tests use fixture Role Intelligence Profiles from `tests/fixtures/sample_role_profiles.json`
- Include at least one fixture for each role type: process-heavy, decision-heavy, relationship-heavy

**Logging**
- Log session ID, stage, block, and question index on every LLM call
- Log all risk flags as they are detected with their source block
- Never log raw conversation content in production — log metadata only

---

## MVP SCOPE — WHAT IS IN AND OUT

**In scope for MVP:**
- Stage 1 chat interface (web)
- Stage 2 chat interface (web, separate session link)
- Stage 3 document output as Word (.docx) and PDF
- Risk flag detection and Risk Summary section
- Role Intelligence Profile generation and manager review step
- Follow-up classifier logic
- Email delivery of completed document to recipients

**Out of scope for MVP (Phase 2):**
- Notion, Confluence, SharePoint integrations
- WhatsApp or email-based interview interface
- Multi-language support
- Analytics dashboard
- Bulk/concurrent session management for enterprise
- Existing template ingestion and matching

---

## KNOWN DESIGN DECISIONS AND RATIONALE

**Why two separate session links rather than one conversation?**
Keeping Manager and Employee sessions fully separate prevents any awkwardness, ensures the employee can be candid, and gives the manager a review checkpoint between stages. It also makes the product feel more considered and enterprise-appropriate.

**Why a separate Haiku classifier for follow-up decisions?**
Relying on the main model to self-decide when to follow up creates inconsistency — sometimes it will, sometimes it won't, depending on subtle prompt drift across a long conversation. A dedicated lightweight classifier with a structured boolean output is more reliable and cheaper to run at scale.

**Why generate Stage 3 in a single call rather than section by section?**
Cross-referencing between sections (e.g. a risk flag in Section 2 referencing a relationship in Section 5) requires the full context to be present at generation time. Section-by-section generation loses this coherence.

**Why Postgres for session state between stages, rather than Redis?**
The gap between Stage 1 and Stage 2 could be hours or days (the manager completes Stage 1, then shares the link with the employee), so the state has to outlive the process — a redeploy in that window must not destroy an in-flight interview. Redis was the original plan, but LangGraph's `RedisSaver.setup()` issues `FT._LIST`, a RediSearch command: it requires Redis Stack, not the plain Redis addon a managed platform gives you, and it fails at startup with `ResponseError: unknown command 'ft._list'`. Postgres already had to be in the stack for durable documents and cross-worker locking, so everything now lives there: sessions and profiles (`kk_sessions`, `kk_profiles`, `kk_session_links`), LangGraph checkpoints (`PostgresSaver`), generated documents as bytes (`kk_documents`), and `pg_advisory_lock` to serialise a session's graph run across workers. TTL is enforced by an expiry sweep rather than by the store. When `DATABASE_URL` is unset the app falls back to in-memory equivalents — fine for tests and local runs, fatal at startup outside development. See `api/postgres_store.py`.

**Why stage-scoped signed tokens rather than unguessable session URLs?**
The Stage 2 link is *designed to be forwarded to the departing employee*, so a capability-URL model makes the link itself the credential — and it was the manager's session id. Anyone holding it could read and write the manager's Stage 1 interview, and could generate and download the handover pack including the Risk Summary written about them. Tokens are HMAC-signed with `API_SECRET_KEY`, carry `{session_id, scope, exp}`, and come in two scopes: `manager` (authorises its own session and the linked one, plus generate/download) and `employee` (that one session only, never the document). See `api/auth.py`.

**Why does the manager create the employee's session?**
Minting an employee token requires proving you are the manager. The employee's browser therefore no longer creates the Stage 2 session — it opens a link the manager generated, carrying a token scoped to that session alone.

---

## USEFUL COMMANDS

```bash
# Run Stage 1 in isolation (CLI for testing)
python -m agents.stage1_business_interview.graph --mode cli

# Run Stage 2 with a fixture profile (CLI for testing)
python -m agents.stage2_employee_interview.graph --mode cli --profile tests/fixtures/sample_role_profiles.json --profile-id relationship_heavy

# Generate a test document from fixture transcripts
python -m agents.stage3_document_generation.generator --stage1 tests/fixtures/stage1_transcript.json --stage2 tests/fixtures/stage2_transcript.json
```

---

*KnowledgeKeeper | Nukode | CLAUDE.md v1.0*

---
> Source: [iDevMode/knowledge_keeper_agent](https://github.com/iDevMode/knowledge_keeper_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
