## critqai

> > **CritiqAI** is a multi-agent peer-review simulator built with Google ADK.

# AGENTS.md — CritiqAI

> **CritiqAI** is a multi-agent peer-review simulator built with Google ADK.
> It assigns AI personas to challenge student essays using Socratic questioning —
> never revealing answers, only probing reasoning.
>
> **Core pitch:** *Using AI to teach students NOT to depend on AI.*

---

## Architecture Overview

```text
                         ┌─────────────────────────────────────────────────────┐
                         │                   CritiqAI System                   │
                         │                                                     │
    Student/Teacher ────►│  FastAPI Web UI  ◄──────►  DebateSessionManager     │
                         │  (web_app.py)              (session_manager.py)     │
                         └──────────────────────────────────┬──────────────────┘
                                                            │
                   ┌────────────────────────────────────────▼────────────────────────────────────┐
                   │                      Google ADK Agent Pipeline (6 agents)                  │
                   │                                                                             │
                   │   ┌────────────┐     ┌────────────────┐                                    │
                   │   │ Summarizer │────►│ Persona        │                                    │
                   │   │  Agent     │     │ Selector Agent │                                    │
                   │   └──────┬─────┘     └───────┬────────┘                                    │
                   │          │ essay_summary      │ personas[]                                 │
                   │          └────────────────────▼                                            │
                   │                        ┌─────────────┐   generate    ┌──────────────────┐  │
                   │                        │  Debate     │──────────────►│   Challenge      │  │
                   │                        │  Agent      │◄── student    │   Validator      │  │
                   │                        │  (3 rounds) │    response   │   Agent          │  │
                   │                        └─────────────┘               └────────┬─────────┘  │
                   │                                                   pass/retry  │ challenge  │
                   │                                                               ▼  to student│
                   │                                              ┌────────────────────────┐    │
                   │                                              │ transcript (validated) │    │
                   │                                              └───────────┬────────────┘    │
                   │                                                          │                 │
                   │                                                 ┌────────▼──────┐          │
                   │                                                 │  Analytics    │          │
                   │                                                 │  Agent        │          │
                   │                                                 └────────┬──────┘          │
                   │                                                          │ scores+analysis │
                   │                                                 ┌────────▼──────┐          │
                   │                                                 │  Report       │──► Gmail │
                   │                                                 │  Agent        │  (HITL)  │
                   │                                                 └───────────────┘          │
                   └─────────────────────────────────────────────────────────────────────────────┘

MCP Servers
───────────
  [HTTP/SSE]  argument-scorer (Cloud Run)  ──►  Analytics Agent  (hybrid: 0 tokens EN · ~300 tokens non-EN, LRU-cached)
  [SSE]       Google Drive MCP  ──►  read essay from Google Docs
  [SSE]       Google Sheets MCP ──►  append debate log rows
  [SSE]       Google Gmail MCP  ──►  create_draft only (compose scope)

  Local dev fallback: argument-scorer can run as stdio subprocess or be imported directly.

Key multi-agent interaction:
  Debate Agent generates a challenge → Validator Agent independently reviews it
  (deterministic: answer-leak check + single-question rule + length check) → passes or
  triggers regeneration (max 1 retry, zero LLM tokens). Two agents collaborating, not just a chain.
```

---

## Agents

### 1. Summarizer Agent

| Property | Value                                                                          |
| -------- | ------------------------------------------------------------------------------ |
| File     | `agents/orchestrator.py` → `run_summarizer()`                                  |
| Model    | `gemini-2.5-flash-lite`                                          |
| Input    | Full essay text (up to 2000 words, sanitized)                                  |
| Output   | `{main_claim, supporting_points[], evidence[], conclusion}`                    |
| Purpose  | Compress essay to ~200 tokens before downstream agents; reduces token cost 75% |
| Skills   | none                                                                           |

### 2. Persona Selector Agent

| Property | Value                                                                          |
| -------- | ------------------------------------------------------------------------------ |
| File     | `agents/orchestrator.py` → `run_persona_selector()`                            |
| Model    | `gemini-2.5-flash-lite`                                          |
| Input    | Summarized essay dict                                                          |
| Output   | `{selected_personas: ["Skeptic", "DevilsAdvocate"], reasoning: "..."}`         |
| Purpose  | Select 1–2 debate personas best matched to the essay's specific weaknesses     |
| Skills   | `skills/persona-selector/SKILL.md` — decision tree: weakness pattern → persona |

**Selection logic:**

- Weak evidence → `Skeptic`
- Ignores counterarguments → `DevilsAdvocate`
- Internal contradictions → `Nitpicker`
- Argument too narrow in scope → `Expander`

### 3. Debate Agent

| Property | Value |
| --- | --- |
| File | `agents/debate.py` → `run_debate_round()` |
| Model | `gemini-2.5-flash-lite` |
| Input | persona, essay_summary, exchange_history (sliding window last 2), round_number, essay_language |
| Output | One pointed challenge string (never an answer or correction) |
| Purpose | Execute 1 of 3 debate rounds; escalates pressure each round |
| Skills | `skills/critical-thinking-rubric/SKILL.md` — Paul-Elder dimensions, escalation templates |

**Persona config loaded from `personas.json` at startup (hot-reloadable, no restart needed).**

| Persona | Focus | Tone |
| --- | --- | --- |
| `[SKEPTIC]` | Evidence reliability — source credibility, sample size, cherry-picking | Calm, methodical |
| `[DEVILS_ADVOCATE]` | Strongest opposing case — real counterexamples, forces engagement | Never "you're wrong", always "consider this case" |
| `[NITPICKER]` | Logical consistency — conclusion following premises, term consistency | Precise, surgical |
| `[EXPANDER]` | Scope & assumptions — hidden boundaries, unexamined edge cases | Expansive, probing |

**Round escalation:** Round 1 = open challenge; Round 2 = pressure on weakest response; Round 3 = corner the student.

### 4. Challenge Validator Agent

| Property | Value |
| --- | --- |
| File | `agents/validator.py` → `validate_challenge()`, `validate_and_maybe_retry()` |
| Model | **none — deterministic only, zero LLM tokens** |
| Input | challenge text, persona name, round_number, essay_language |
| Output | `{valid, issues[], severity: "pass"/"warn"/"fail", reasoning, fast_check_passed}` |
| Purpose | Quality-gate every debate challenge before it reaches the student. Pure Python checks — no API calls. |
| Security | Implements **Pillar 7 (Behavioral Monitoring)** — live answer-leak detection |

**Three deterministic checks (no LLM call — runs in microseconds):**

1. **Answer-leak pattern check (hard fail):** 18 keyword patterns (`_ANSWER_LEAK_PATTERNS`) — e.g. `"you should argue"`, `"the correct answer"`. If triggered → immediate `severity=fail`.
2. **Single-question rule (hard fail):** Counts `?` in the challenge. More than 2 → `severity=fail` (one main question + one rhetorical follow-up allowed).
3. **Length check (warn only):** More than 150 words → `severity=warn`, challenge still usable.

**Retry logic:** On `severity=fail`, regenerates challenge via Debate Agent (max 1 retry). If retry also fails, logs `ERROR` and passes through to avoid blocking the student.

### 5. Analytics Agent

| Property | Value                                                                                                                                     |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| File     | `agents/analytics.py` → `format_analytics()`                                                                                              |
| Model    | none — pure Python formatting                                                                                                             |
| Input    | MCP scores dict from `argument-scorer`                                                                                                    |
| Output   | `{session_id, student_name, scores, total_score, max_possible, percentage, key_strengths[], key_weaknesses[], improvement_suggestions[]}` |
| Purpose  | Format deterministic MCP scores into human-readable analytics                                                                             |
| Tools    | `argument-scorer` MCP — deterministic keyword scoring (EN, 0 tokens) or 1 compact LLM call (non-EN, ~300 tokens, LRU-cached)             |

### 6. Report Agent

| Property | Value                                                                                     |
| -------- | ----------------------------------------------------------------------------------------- |
| File     | `agents/report.py` → `build_report_email()`                                               |
| Model    | **none — pure Python template, zero LLM tokens**                                          |
| Input    | Analytics output, student first name                                                      |
| Output   | `(subject: str, body: str)` → passed to Gmail MCP `create_draft`                          |
| Purpose  | Generate email draft for teacher; **teacher must manually approve before sending (HITL)** |
| Tools    | Gmail MCP — `create_draft` only; `gmail.compose` scope; cannot read inbox or auto-send    |

---

## Tools & MCP Servers

### argument-scorer (custom FastMCP server)

| Property       | Value                                                                               |
| -------------- | ----------------------------------------------------------------------------------- |
| Transport      | **HTTP streamable-http** (Cloud Run) · stdio fallback for local dev                 |
| Location       | `mcp_servers/argument_scorer/server.py` · `Dockerfile`                              |
| Deployment     | Separate Cloud Run service; URL set via `SCORER_URL` env var                        |
| Tool           | `score_argument(text: str) → dict`                                                  |
| Implementation | **Hybrid:** keyword matching for English (0 LLM calls); 1 compact Gemini call for non-English vi/ja/zh (~300 tokens, LRU-cached, `temperature=0`) |

**Routing logic (`mcp_client.py`):**

1. `SCORER_URL` is set → connect via HTTP streamable-http (Cloud Run)
2. `SCORER_URL` not set → spawn stdio subprocess (`mcp_servers/argument_scorer/server.py`)
3. Both fail → direct Python import of `rubric.py` (inline fallback, zero subprocess)

Scoring dimensions (Paul-Elder framework, each 0–5):

| Dimension                  | Measures                     | Key signals                                              |
| -------------------------- | ---------------------------- | -------------------------------------------------------- |
| `logical_coherence`        | Claims follow from premises  | "therefore", "because", "thus"; absence of non-sequiturs |
| `evidence_quality`         | Concreteness of evidence     | Specific data vs "everyone knows", "studies show"        |
| `counterargument_handling` | Engages with opposing views  | "however", "critics argue", "one might argue"            |
| `scope_awareness`          | Acknowledges argument limits | Hedging language, explicit scope, edge cases             |

Config: `scorer_config.json` (generated from UI Scorer Settings tab or default values).

### Google Drive MCP

- Transport: SSE
- Scope: `drive.readonly` only
- Restricted to: `essay-submissions/` folder
- Used by: `mcp_client.read_essay_from_drive()`

### Google Sheets MCP

- Transport: SSE / REST API fallback
- Scope: `spreadsheets` (append only in practice)
- Restricted to: `DEBATE_LOG_SHEET_ID` from env
- Used by: `mcp_client.append_debate_row()`

### Google Gmail MCP

- Transport: SSE
- Scope: `gmail.compose` — **cannot read inbox, cannot send**
- Allowed tools: `create_draft` only
- HITL: teacher opens Gmail, reviews draft, sends manually

---

## Agent Skills

### `critical-thinking-rubric`

- Location: `skills/critical-thinking-rubric/SKILL.md`
- Used by: Debate Agent
- Purpose: Teaches Paul-Elder framework dimensions; provides escalation pressure templates for rounds 1→2→3
- Trigger phrases: "evaluate argument", "score thinking", "assess reasoning", "critical challenge"

### `persona-selector`

- Location: `skills/persona-selector/SKILL.md`
- Used by: Persona Selector Agent
- Purpose: Decision tree mapping essay weakness patterns to persona names; includes sample opening lines per persona

### `answer-leak-patterns`

- Location: `skills/answer-leak-patterns/SKILL.md`
- Used by: Challenge Validator Agent
- Purpose: Documents the 18 heuristic patterns used to detect inadvertent answer-leaks in debate challenges, with rationale for each pattern

---

## Data Flow

```text
Input: student_name + essay_text (or Google Doc URL)
  │
  ▼
[sanitize_essay()]              ← strips control chars, enforces 2000-word limit (Pillar 4)
  │
  ▼
[detect_essay_language()]       ← regex-based: Japanese / Vietnamese / Chinese / English
  │
  ▼
[Summarizer Agent]              → essay_summary {main_claim, supporting_points, evidence, conclusion}
  │
  ▼
[Persona Selector Agent]        → personas ["Skeptic", "DevilsAdvocate"]  +  reasoning
  │                               (always 2 personas — scores all 4 dimensions, picks top 2)
  ▼
[Debate Agent] ──generate──► [Validator Agent] ──pass/retry──► challenge_1 → student
  │  student_response_1  ──────────────────────────────────────► [Sheets MCP: append row]
  ▼
[Debate Agent] ──generate──► [Validator Agent] ──pass/retry──► challenge_2 → student
  │  student_response_2  ──────────────────────────────────────► [Sheets MCP: append row]
  ▼
[Debate Agent] ──generate──► [Validator Agent] ──pass/retry──► challenge_3 → student
  │  student_response_3  ──────────────────────────────────────► [Sheets MCP: append row]
  │
  ▼
[argument-scorer MCP]           → scores {logical_coherence, evidence_quality,
  │                                        counterargument_handling, scope_awareness,
  │                                        total, percentage, language, scoring_method}
  │                                        ← 0 tokens (EN) or ~300 tokens 1-call (non-EN, cached)
  ▼
[Analytics Agent]               → {key_strengths, key_weaknesses, suggestions}  ← 0 LLM tokens
  │
  ▼
[Report Agent]                  → (subject, body)  ← 0 LLM tokens
  │
  ▼
[Gmail MCP: create_draft]       → draft_id  ──► Teacher reviews in Gmail → sends manually (HITL)
```

---

## Running the System

### Prerequisites

```bash
git clone <repo>
cd CritqAI
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
pip install -r requirements.txt
```

### Environment setup

```bash
cp .env.example .env
# Edit .env and fill in:
# GOOGLE_API_KEY=...            ← Gemini API key (Google AI Studio free tier)
# GOOGLE_OAUTH_CLIENT_ID=...
# GOOGLE_OAUTH_CLIENT_SECRET=...
# DEBATE_LOG_SHEET_ID=...
# TEACHER_EMAIL=...
```

### First-time OAuth (Drive / Sheets / Gmail)

```bash
python mcp_client.py
# Opens browser for Google OAuth — approve all requested scopes
# Saves token.json (gitignored)
```

### Start the web UI

```bash
python web_app.py
# Teacher dashboard → http://localhost:8000/teacher
# Student debate   → http://localhost:8000/student
# /  redirects to /teacher automatically
```

### Run evals

```bash
adk eval evals/debate_quality.evalset.json
adk eval evals/persona_trigger.evalset.json
```

### Run argument-scorer MCP directly

```bash
cd mcp_servers/argument_scorer
python server.py
# Runs on stdio — ADK connects automatically when web_app.py starts
```

---

## Security Architecture (7 Pillars)

| # | Pillar | Implementation |
| --- | --- | --- |
| 1 | **Input Validation** | `sanitize_essay()` strips control chars, enforces 2000-word hard limit before any LLM call |
| 2 | **Supply Chain** | All deps pinned in `requirements.txt` with exact versions; `requirements.lock` for full transitive freeze |
| 3 | **Secrets Management** | Zero hardcoded credentials; all via `.env` env vars; `token.json` + `.env` in `.gitignore` |
| 4 | **Least-Privilege Egress** | Drive: `drive.readonly`; Sheets: `spreadsheets` (append only in practice); Gmail: `gmail.compose` only — cannot read inbox or auto-send |
| 5 | **HITL Gate** | Gmail `create_draft` only — teacher opens Gmail, reviews, sends manually. System cannot send email autonomously. |
| 6 | **Zero Ambient Authority** | Scoped OAuth per service; no service account; no broad API keys; `GOOGLE_API_KEY` only for Gemini inference |
| 7 | **Behavioral Monitoring** | `Challenge Validator Agent` (`agents/validator.py`) quality-gates every LLM output before it reaches the student: 18-pattern answer-leak check + single-question rule + length check — all deterministic Python, zero LLM tokens; retries on hard fail; all severity levels logged |

---

## File Structure

```text
CritiqAI/
├── AGENTS.md                        ← this file
├── SPEC.md                          ← agent acceptance criteria & test spec
├── CONTEXT.md                       ← extended developer context
├── COMPETITION_OVERVIEW.md          ← competition rubric context
├── RUNNING.md                       ← detailed setup guide
├── QUICKSTART.md                    ← quick start guide
├── LICENSE                          ← CC-BY 4.0
├── requirements.txt                 ← pinned dependencies
├── requirements.lock                ← exact frozen versions
├── web_app.py                       ← FastAPI app + embedded UI (single file)
├── session_manager.py               ← DebateSessionManager (orchestration)
├── mcp_client.py                    ← MCP client + OAuth helpers
├── agents/
│   ├── orchestrator.py              ← Summarizer + PersonaSelector ADK agents
│   ├── debate.py                    ← Debate ADK agent (4 personas, personas.json driven)
│   ├── validator.py                 ← Challenge Validator (deterministic, 0 LLM tokens)
│   ├── model_config.py              ← Model fallback chain + ADK runner
│   ├── analytics.py                 ← Analytics formatting
│   └── report.py                    ← Report + Gmail draft
├── mcp_servers/
│   └── argument_scorer/
│       ├── server.py                ← FastMCP server
│       └── rubric.py                ← Deterministic scoring (Paul-Elder)
├── skills/
│   ├── critical-thinking-rubric/SKILL.md
│   └── persona-selector/SKILL.md
├── evals/
│   ├── debate_quality.evalset.json
│   └── persona_trigger.evalset.json
├── scorer_config.json               ← scorer parameters (gitignored, UI-generated)
├── critiqai_demo.ipynb              ← Interactive demo notebook (local / Colab)
└── critiqai_kaggle.ipynb            ← Kaggle submission notebook (reproducible demo)
```

---

## Evaluation Summary

| Evalset           | What it tests                                            | Pass threshold                        |
| ----------------- | -------------------------------------------------------- | ------------------------------------- |
| `debate_quality`  | Debate Agent challenges: relevant + withholds answer     | LLM-as-judge ≥ 3/5 on both dimensions |
| `persona_trigger` | PersonaSelector picks correct persona for 20 test essays | Exact match ≥ 85%                     |

---

*License: CC-BY 4.0 — see LICENSE file.*

---
> Source: [francisnguyenanh/CritqAI](https://github.com/francisnguyenanh/CritqAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
