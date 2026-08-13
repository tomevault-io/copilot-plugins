## csv-gamechanger

> Generates User Requirements Specifications by querying Pinecone for relevant GAMP 5 guidance.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CSV-GameChanger is a GAMP 5 and CSA (Computer Software Assurance) compliant CSV (Computer System Validation) Engine.

## Branding Convention (IMPORTANT — follow exactly)

This project uses **dual branding**. Getting this wrong causes rework.

| Concept | Name | Where to use |
|---------|------|-------------|
| **Product / tool brand** | **EVOLV** (all caps) | "Powered by" lines, tool-name contexts, sidebar logo text, legal/signature text |
| **Functional label** | **Validation Factory** | Page headers / hero sections, navigation menu, about text, process descriptions, requirement-generation logic |
| **Combined form** | **EVOLV: The Validation Factory** or **EVOLV \| The Validation Factory** | Browser tab title, PDF headers, CLI banners |
| **Company** | **WingstarTech Inc.** | Footer attribution |

**Rules:**
1. The main page header (hero) for Tab 6 must say **"Validation Factory"**, NOT "EVOLV".
2. The sidebar nav label for Tab 6 must say **"6. Validation Factory"**.
3. Footer stays: `"Powered by EVOLV | A WingstarTech Inc. Product"`.
4. PDF headers use: `"EVOLV | The Validation Factory"`.
5. Sidebar logo: "EVOLV" with subtitle "THE VALIDATION FACTORY".
6. **Never** replace "Validation Factory" with just "EVOLV" in user-facing text — "Validation Factory" is the core business identity.
7. Old names ("Trustme AI", "CSV Engine") are retired. Internal class names like `CSVEngineError` are kept to avoid breaking imports.

## Build and Development Commands

```bash
# Install dependencies
pip install fastapi uvicorn pydantic pinecone openai langchain-community langchain-text-splitters fpdf2

# Run the API server
uvicorn API.main:app --reload

# Run from project root
cd C:\Users\sreej\OneDrive\Desktop\CSV-GameChanger
uvicorn API.main:app --reload --host 0.0.0.0 --port 8000

# Generate URS document (interactive mode)
python scripts/draft_urs.py

# Generate URS from file
python scripts/draft_urs.py -f requirements.txt -n "Project Name"
```

## Architecture

```
CSV-GameChanger/
├── Agents/
│   ├── __init__.py
│   ├── risk_strategist.py       # GAMP 5 risk assessment logic
│   ├── requirement_architect.py # URS generation from natural language
│   ├── verification_agent.py    # URS verification against GAMP 5 text
│   └── integrity_manager.py     # Central audit trail + logic archives
├── API/
│   ├── __init__.py
│   └── main.py                  # FastAPI app with ServiceNow webhook
├── scripts/
│   ├── setup_pinecone_index.py  # Creates Pinecone index
│   ├── ingest_docs.py           # Ingests GAMP 5 PDFs to Pinecone
│   └── draft_urs.py             # Generate URS documents from requirements
├── utils/
│   ├── __init__.py
│   └── pdf_generator.py         # URS PDF export with signature page
├── output/
│   ├── urs/                     # Generated URS Markdown/PDF files
│   └── logic_archives/          # Hidden JSON logic-archive files (generated)
├── audit_trail.log              # 21 CFR Part 11 compliant audit log (generated)
└── CLAUDE.md
```

## Current Implementation State

### API/main.py

**Endpoint:** `POST /webhook/sn-change`

Receives ServiceNow Change Requests and triggers automated risk assessment.

**Request Model (`ServiceNowChangeRequest`):**
```python
{
    "cr_id": str,              # Change Request ID
    "description": str,         # Change description
    "system_criticality": str,  # "high", "medium", "low", "critical", "minor"
    "change_type": str          # "emergency", "normal", "standard", "routine"
}
```

**Response Model (`ChangeRequestResponse`):**
```python
{
    "status": "assessed",
    "cr_id": str,
    "message": str,
    "timestamp": str,
    "risk_assessment": {
        "severity": str,           # HIGH, MEDIUM, LOW
        "occurrence": str,         # FREQUENT, OCCASIONAL, RARE
        "detectability": str,      # HIGH, MEDIUM, LOW
        "rpn": int,                # Risk Priority Number (1-27)
        "risk_level": str,         # "High", "Medium", "Low"
        "testing_strategy": str,   # CSA recommendation
        "patient_safety_override": bool
    }
}
```

**Audit Events Logged:**
1. `CHANGE_REQUEST_RECEIVED` - When CR arrives
2. `RISK_ASSESSMENT_COMPLETED` - After risk calculation
3. `CHANGE_REQUEST_FAILED` - On any error

**Exception Classes:**
- `CSVEngineError` - Base exception
- `ValidationError` (CSV-001) - Input validation failed
- `AuditLogError` (CSV-002) - Audit logging failed
- `ProcessingError` (CSV-003) - Processing failed

### Agents/risk_strategist.py

**Risk Strategist Agent** - Implements GAMP 5 risk-based approach.

**Enums:**
- `RiskLevel`: LOW, MEDIUM, HIGH
- `Severity`: LOW (1), MEDIUM (2), HIGH (3)
- `Occurrence`: RARE (1), OCCASIONAL (2), FREQUENT (3)
- `Detectability`: HIGH (1), MEDIUM (2), LOW (3)
- `TestingStrategy`: UNSCRIPTED, HYBRID, RIGOROUS_SCRIPTED

**Core Functions:**

| Function | Input | Output | Purpose |
|----------|-------|--------|---------|
| `calculate_risk_score()` | Severity, Occurrence, Detectability | (RPN, RiskLevel) | Calculates Risk Priority Number |
| `get_csa_testing_strategy()` | RiskLevel | TestingStrategy | Returns CSA testing recommendation |
| `assess_change_request()` | system_criticality, change_type | dict | Full assessment from ServiceNow fields |
| `map_criticality_to_severity()` | str | Severity | Maps ServiceNow criticality to GAMP 5 |
| `map_change_type_to_occurrence()` | str | Occurrence | Maps change type to occurrence |

**GAMP 5 Risk Logic:**
1. **Patient Safety Override:** If Severity = HIGH → Risk = HIGH (regardless of other factors)
2. **RPN Calculation:** Severity × Occurrence × Detectability (scale 1-27)
3. **Risk Thresholds:**
   - RPN ≤ 4 → LOW risk
   - RPN 5-12 → MEDIUM risk
   - RPN > 12 → HIGH risk

**CSA Testing Strategy:**
- LOW risk → Unscripted Testing
- MEDIUM risk → Hybrid Testing (Scripted + Unscripted)
- HIGH risk → Rigorous Scripted Testing

**ServiceNow Field Mappings:**

| system_criticality | → Severity |
|--------------------|------------|
| high, critical | HIGH |
| medium, moderate | MEDIUM |
| low, minor | LOW |

| change_type | → Occurrence |
|-------------|--------------|
| emergency, expedited | FREQUENT |
| normal | OCCASIONAL |
| standard, routine | RARE |

### Agents/requirement_architect.py

**Requirement Architect Agent** - Generates URS documents from natural language using GAMP 5 context.

**Class: `RequirementArchitect`**

Generates User Requirements Specifications by querying Pinecone for relevant GAMP 5 guidance.

**Enums:**
- `Criticality`: HIGH, MEDIUM, LOW
- `RiskAssessmentCategory`: GXP_DIRECT, GXP_INDIRECT, GXP_NONE
- `ImplementationMethod`: OUT_OF_THE_BOX, CONFIGURED, CUSTOM
- `URFRRiskLevel`: HIGH, MEDIUM, LOW
- `URFRTestStrategy`: OQ_UAT, INFORMAL, SUPPLIER_PROVIDED

**Data Classes:**
- `URSDocument`: Structured URS with urs_id, requirement_statement, criticality, regulatory_rationale

**Exception Classes:**
- `RegulatoryContextNotFoundError` (CSV-004) - No matching GAMP 5/CSA context found

**Data Classes:**
- `SearchResult`: chunk_id, text, source_document, page_number, similarity_score, reg_version
- `SearchResponse`: query, results, total_results

**Core Methods:**

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `search()` | query: str, top_k: int, min_score: float | SearchResponse | Queries Pinecone for GAMP 5/CSA chunks |
| `generate_urs()` | requirement: str, min_score: float | dict | Generates structured URS from natural language |
| `transform_urs_to_ur_fr()` | urs: dict, role, category, risk_assessment, implementation_method, additional_context | dict | Transforms URS into UR/FR document (deterministic) |

**`additional_context` Parameter (Optional):**

Passed from the Validation Factory UI to enrich the UR/FR output with project-specific context. All keys are optional; the parameter itself defaults to `None`.

| Key | Source (UI) | Incorporated Into |
|-----|-------------|-------------------|
| `system_description` | System Description text area | `assumptions_and_dependencies` |
| `workshop_notes` | Workshop Notes text area | `assumptions_and_dependencies` |
| `roles_and_permissions` | User Roles & Permissions text area | `compliance_notes` |
| `lucidchart_url` | Lucidchart / Diagram Link text input | `implementation_notes` |
| `lucidchart_content` | Diagram file uploader (decoded UTF-8) | `implementation_notes` |
| `lucidchart_filename` | Diagram file uploader (binary fallback) | `implementation_notes` |

When any context is provided, the raw dict is also stored in the output under the `additional_context` key for downstream consumers (e.g. PDF generator, test scripts).

**UR/FR Risk Matrix (RiskAssessmentCategory x ImplementationMethod → Risk Level):**

| IM \ RA | GxP Direct | GxP Indirect | GxP None |
|---------|-----------|-------------|---------|
| Custom | HIGH | MEDIUM | LOW |
| Configured | HIGH | HIGH | LOW |
| Out of the Box | MEDIUM | LOW | LOW |

**UR/FR Test Strategy Matrix (Risk Level x ImplementationMethod → Test Strategy):**

| Risk \ IM | Out of the Box | Configured | Custom |
|-----------|---------------|-----------|--------|
| HIGH | OQ and/or UAT | OQ and/or UAT | OQ and/or UAT |
| MEDIUM | Supplier Provided | Informal | Informal |
| LOW | Supplier Provided | Informal | Informal |

**URS Output Format:**
```python
{
    "URS_ID": "URS-7.1",
    "Requirement_Statement": "The system shall track warehouse temp.",
    "Criticality": "Medium",
    "Regulatory_Rationale": "Per GAMP5_Guide.pdf [GAMP5_Rev2] (p.42): ...",
    "Reg_Versions_Cited": ["GAMP5_Rev2"]
}
```

**Search Output Format:**
```python
{
    "query": "temperature monitoring",
    "total_results": 3,
    "results": [
        {
            "chunk_id": "abc123",
            "text": "Temperature monitoring is critical...",
            "source_document": "GAMP5_Guide.pdf",
            "page_number": 42,
            "similarity_score": 0.87,
            "reg_version": "GAMP5_Guide"
        }
    ]
}
```

**Criticality Classification Logic:**

| Criticality | Indicators |
|-------------|------------|
| HIGH | patient, safety, critical, gxp, compliance, validation, sterile, batch, release, adverse, pharmacovigilance, clinical, regulatory, fda, ema |
| MEDIUM | quality, audit, traceability, calibration, deviation, capa, change control, training, document, sop, warehouse, inventory, temperature |
| LOW | Administrative functions, non-GxP systems, convenience features |

**Dependencies:**
- Pinecone (csv-knowledge-base index)
- OpenAI (text-embedding-3-small)

**Usage Example:**
```python
from Agents.requirement_architect import (
    RequirementArchitect,
    RegulatoryContextNotFoundError
)

architect = RequirementArchitect()

# Search for relevant GAMP 5/CSA content
results = architect.search("temperature monitoring requirements")
for r in results.results:
    print(f"{r.source_document} (p.{r.page_number}): {r.text[:100]}...")

# Generate URS (requires at least one matching chunk)
try:
    urs = architect.generate_urs("I want to track warehouse temp")
    print(urs)
except RegulatoryContextNotFoundError as e:
    print(f"Error: {e}")

# Transform URS to UR/FR (deterministic, no LLM calls)
ur_fr = architect.transform_urs_to_ur_fr(
    urs=urs,
    role="Lab Technician",
    category="General",
    risk_assessment="GxP Indirect",
    implementation_method="Configured",
    additional_context={
        "system_description": "LabCore LIMS v4.2 — cloud-hosted",
        "workshop_notes": "Chain-of-custody is safety-critical",
        "roles_and_permissions": "Technician: receipt/transfer\nSupervisor: disposal",
        "lucidchart_url": "https://lucid.app/example",
    },
)
print(ur_fr["user_requirement"]["risk_level"])   # "High"
print(ur_fr["user_requirement"]["test_strategy"]) # "OQ and/or UAT"
```

**UR/FR Output Format:**
```python
{
    "urs_id": "URS-7.1",
    "requirement_summary": "The system shall track warehouse temperature.",
    "category": "General",
    "user_requirement": {
        "ur_id": "UR-1",
        "statement": "As a User, there will be track warehouse temperature so that the requirement is fulfilled.",
        "risk_assessment": "GxP Indirect",
        "implementation_method": "Configured",
        "risk_level": "High",
        "test_strategy": "OQ and/or UAT",
        "risk_note": "Final Risk Profiling will be decided with stakeholders..."
    },
    "functional_requirements": [
        {
            "fr_id": "FR-1",
            "parent_ur_id": "UR-1",
            "statement": "The system shall track warehouse temperature",
            "acceptance_criteria": ["Given/When/Then..."]
        }
    ],
    "assumptions_and_dependencies": ["...", "System Description: ...", "Workshop Notes: ..."],
    "compliance_notes": ["Cross-reference SOP-436231...", "...", "User Roles & Permissions: ..."],
    "implementation_notes": ["...", "Diagram Reference: https://...", "Diagram content provided inline."],
    "reg_versions_cited": ["GAMP5_Rev2"],
    "additional_context": {
        "system_description": "...",
        "workshop_notes": "...",
        "roles_and_permissions": "...",
        "lucidchart_url": "...",
        "lucidchart_content": "..."
    }
}
```

### Agents/verification_agent.py

**Verification Agent** - Reviews URS output from the RequirementArchitect against GAMP 5 regulatory text in Pinecone. Rejects non-compliant drafts and logs Compliance Exceptions.

**Class: `VerificationAgent`**

Runs three independent checks on each URS document:

1. **Criticality Alignment** - Detects under-classification by scanning GAMP 5 chunks for high-risk indicators when criticality is Low or Medium.
2. **Rationale Relevance** - Verifies the best Pinecone match score meets the relevance threshold (0.45).
3. **Contradiction Scan** - Matches known contradiction phrase pairs (e.g. "skip validation" vs. GAMP 5 "validation is required") across validation, testing, audit trail, and change control domains.

**Enums:**
- `Verdict`: APPROVED, REJECTED
- `CheckStatus`: PASS, FAIL

**Data Classes:**
- `VerificationFinding`: check_name, status, detail, gamp5_reference
- `VerificationResult`: urs_id, verdict, findings

**Exception Classes:**
- `VerificationError` (CSV-010) - Base verification error
- `InvalidURSError` (CSV-011) - URS missing required fields

**Core Methods:**

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `verify_urs()` | urs: dict, min_score: float | VerificationResult | Verify a single URS against GAMP 5 text |
| `verify_batch()` | urs_list: List[dict], min_score: float | List[VerificationResult] | Verify multiple URS documents |

**Configuration Constants:**

| Constant | Value | Purpose |
|----------|-------|---------|
| `VERIFICATION_TOP_K` | 5 | Max Pinecone results per query |
| `VERIFICATION_MIN_SCORE` | 0.35 | Minimum similarity score for retrieved chunks |
| `RATIONALE_RELEVANCE_THRESHOLD` | 0.45 | Minimum score for rationale to be considered relevant |

**High-Risk Indicators:**
patient, safety, critical, gxp, sterile, batch release, adverse event, pharmacovigilance, clinical, life-sustaining, life-supporting, validated, 21 cfr part 11

**Contradiction Pairs:**

| Requirement Phrase | Opposing GAMP 5 Keyword |
|--------------------|------------------------|
| skip validation, no validation required | validation, shall be validated |
| skip testing, no testing required | testing, shall be tested, test plan |
| no audit trail, disable audit | audit trail, traceability, 21 cfr part 11 |
| no change control, bypass change control | change control, change management |

**Audit Events Logged:**
1. `URS_VERIFIED` - URS passed all three checks (Compliance Impact: Regulatory Compliance)
2. `COMPLIANCE_EXCEPTION` - URS rejected due to check failure (Compliance Impact: Compliance Exception)
3. `URS_BATCH_VERIFIED` - Batch verification completed (Compliance Impact: Regulatory Compliance)

**Verification Result Output Format:**
```python
{
    "URS_ID": "URS-7.1",
    "Verdict": "Approved",  # or "Rejected"
    "Findings": [
        {
            "check_name": "Criticality Alignment",
            "status": "Pass",
            "detail": "Criticality Medium is consistent with ...",
            "gamp5_reference": "Per GAMP5_Guide.pdf [GAMP5_Guide] (p.42): ..."
        },
        {
            "check_name": "Rationale Relevance",
            "status": "Pass",
            "detail": "Best GAMP 5 match score is 0.87, above ...",
            "gamp5_reference": "Per GAMP5_Guide.pdf [GAMP5_Guide] (p.42): ..."
        },
        {
            "check_name": "Contradiction Scan",
            "status": "Pass",
            "detail": "No contradictions detected ...",
            "gamp5_reference": "Per GAMP5_Guide.pdf [GAMP5_Guide] (p.42): ..."
        }
    ]
}
```

**Dependencies:**
- Pinecone (csv-knowledge-base index)
- OpenAI (text-embedding-3-small)

**Usage Example:**
```python
from Agents.verification_agent import (
    VerificationAgent,
    InvalidURSError
)

agent = VerificationAgent()

# Verify a single URS from RequirementArchitect
urs = {
    "URS_ID": "URS-7.1",
    "Requirement_Statement": "The system shall track warehouse temperature.",
    "Criticality": "Medium",
    "Regulatory_Rationale": "Per GAMP 5 Guide (p.42): ..."
}

result = agent.verify_urs(urs)
print(result.verdict)       # "Approved" or "Rejected"
print(result.is_rejected)   # True/False

for finding in result.findings:
    print(f"{finding.check_name}: {finding.status} - {finding.detail}")

# Batch verification
results = agent.verify_batch([urs1, urs2, urs3])
rejected = [r for r in results if r.is_rejected]
```

### Agents/integrity_manager.py

**Integrity Manager Module** - Provides a central, append-only CSV audit trail and optional logic archives for AI reasoning transparency.

**Constants:**

| Constant | Value | Purpose |
|----------|-------|---------|
| `AUDIT_TRAIL_PATH` | `output/audit_trail.csv` | Central CSV audit trail |
| `LOGIC_ARCHIVE_DIR` | `output/logic_archives/` | Directory for JSON logic-archive files |
| `_ARCHIVE_SCHEMA_VERSION` | `"1.0.0"` | Schema version embedded in each archive |

**Core Functions:**

| Function | Input | Output | Purpose |
|----------|-------|--------|---------|
| `log_audit_event()` | agent_name, action, user_id, decision_logic, compliance_impact, audit_path, thought_process | str (SHA-256 hash) | Append audit record to CSV; optionally write logic archive |
| `_compute_reasoning_hash()` | timestamp, user_id, agent_name, action, decision_logic, compliance_impact | str (SHA-256 hex) | Tamper-evident hash over audit row fields |
| `_validate_thought_process()` | thought_process: Dict | None | Validates dict has `inputs`, `steps` (list), `outputs` keys |
| `_write_logic_archive()` | timestamp, agent_name, action, user_id, compliance_impact, decision_logic, audit_trail_hash, thought_process | Path | Write hidden JSON archive cross-referenced to CSV row |
| `_ensure_csv_header()` | path: Path | None | Write CSV header if file is new or empty |

**Logic Archive Feature:**

When `thought_process` is passed to `log_audit_event()`, a hidden dot-prefixed JSON file is written to `output/logic_archives/` containing the full AI reasoning chain (inputs, intermediate steps, outputs). The archive is cross-referenced to the CSV audit trail row via the SHA-256 reasoning hash and includes its own tamper-evident integrity hash.

**`thought_process` Required Shape:**
```python
{
    "inputs": { ... },   # Dict - agent inputs
    "steps": [ ... ],    # List - intermediate reasoning steps
    "outputs": { ... },  # Dict - agent outputs
}
```

**Logic Archive JSON Schema:**
```json
{
    "$schema_version": "1.0.0",
    "archive_type": "logic_archive",
    "audit_trail_hash": "<SHA-256 from CSV row>",
    "timestamp": "<ISO-8601>",
    "agent_name": "...",
    "action": "...",
    "user_id": "...",
    "compliance_impact": "...",
    "decision_logic_summary": "...",
    "inputs": { },
    "steps": [ ],
    "outputs": { },
    "integrity": {
        "archive_hash": "<SHA-256 of JSON content>",
        "algorithm": "sha256"
    }
}
```

**Archive Filename Convention:** `.{ACTION}_{YYYYMMDDTHHMMSSZ}_{hash[:8]}.json`

**Backward Compatibility:** The `thought_process` parameter defaults to `None`. All existing callers of `log_audit_event()` are unaffected.

**Thread Safety:** Archive writes occur inside the same `_write_lock` as CSV writes, ensuring the CSV row and JSON file are atomically paired.

**Audit Events Logged:**
All agent actions across the system are logged through this module. See `_IMPACT_MAP` for the full action-to-compliance-impact mapping.

**Usage Example:**
```python
from Agents.integrity_manager import log_audit_event

# Without logic archive (existing behavior)
reasoning_hash = log_audit_event(
    agent_name="RiskStrategist",
    action="RISK_ASSESSMENT_COMPLETED",
    decision_logic="RPN=6, Medium risk",
)

# With logic archive
reasoning_hash = log_audit_event(
    agent_name="RequirementArchitect",
    action="URS_GENERATED",
    decision_logic="Generated URS-7.1 from warehouse temp requirement",
    thought_process={
        "inputs": {"requirement": "Track warehouse temperature"},
        "steps": [
            "Queried Pinecone for GAMP 5 context",
            "Classified criticality as Medium",
            "Built regulatory rationale from page 42",
        ],
        "outputs": {"urs_id": "URS-7.1", "criticality": "Medium"},
    },
)
```

### scripts/draft_urs.py

**URS Drafting Script** - Generates complete URS documents from project descriptions.

**Features:**
- Interactive mode for entering requirements
- File input mode for batch processing
- Command-line arguments for automation
- Outputs Markdown files to `output/urs/`

**Core Functions:**

| Function | Input | Output | Purpose |
|----------|-------|--------|---------|
| `draft_urs()` | project_name, project_description | dict | Main entry point for URS generation |
| `parse_requirements()` | project_description | List[str] | Parses bullets/numbered lists into requirements |
| `generate_urs_table()` | requirements, project_name | str | Generates Markdown table format |
| `save_urs_document()` | content, project_name | Path | Saves to output/urs/ directory |

**Usage:**
```bash
# Interactive mode
python scripts/draft_urs.py

# From file
python scripts/draft_urs.py -f requirements.txt -n "My Project"

# Direct input
python scripts/draft_urs.py -n "Warehouse System" -r "Track temperature" "Monitor humidity"

# With custom similarity threshold
python scripts/draft_urs.py -n "Project" -r "requirement" --min-score 0.4
```

**Output Format:**
Generates a Markdown file with:
- Header with project name and timestamp
- Requirements table (URS ID, Statement, Criticality, Rationale)
- Detailed requirements section with full regulatory rationale
- List of failed requirements (if any)

**Example Output File:** `output/urs/URS_Warehouse_System_20240115_143022.md`

### utils/pdf_generator.py

**PDF Generators** - Converts approved URS dictionaries and combined Validation Reports into professional PDFs with a Manifestation of Signature page for 21 CFR Part 11 compliance.

**Core Functions:**

| Function | Input | Output | Purpose |
|----------|-------|--------|---------|
| `generate_urs_pdf()` | urs: dict, signer_name: str, meaning: str | bytes | Generate a two-page PDF from an approved URS |
| `generate_validation_report_pdf()` | ur_fr: dict, test_script: dict, signer_name: str, meaning: str | bytes | Generate combined Validation Report PDF |

**Parameters:**

| Parameter | Type | Default | Purpose |
|-----------|------|---------|---------|
| `urs` | Dict[str, Any] | required | URS dict with URS_ID, Requirement_Statement, Criticality, Regulatory_Rationale, Reg_Versions_Cited |
| `signer_name` | str | required | Full name of the approver |
| `meaning` | str | "Approval of Requirements" | Meaning of the electronic signature |

**PDF Structure:**

- **Page 1 — URS Document:**
  - Branded header (EVOLV | The Validation Factory)
  - URS ID and generation timestamp
  - Requirement Statement
  - Criticality (color-coded: High=red, Medium=amber, Low=green)
  - Regulatory Rationale (split by citation)
  - Regulatory Versions Cited
  - Page footer with page number and date

- **Page 2 — Manifestation of Signature:**
  - Title: "Manifestation of Signature"
  - Signature table with Document, Signer Name, Timestamp (UTC), Meaning
  - Signature and date lines
  - 21 CFR Part 11 compliance note

**Dependencies:**
- fpdf2

**Usage Example:**
```python
from utils.pdf_generator import generate_urs_pdf

urs = {
    "URS_ID": "URS-7.1",
    "Requirement_Statement": "The system shall track warehouse temperature.",
    "Criticality": "Medium",
    "Regulatory_Rationale": "Per GAMP5_Guide.pdf [GAMP5_Rev2] (p.42): ...",
    "Reg_Versions_Cited": ["GAMP5_Rev2"],
}

pdf_bytes = generate_urs_pdf(
    urs=urs,
    signer_name="Jane Smith",
    meaning="Approval of Requirements",
)

# Write to file
with open("URS-7.1.pdf", "wb") as f:
    f.write(pdf_bytes)

# Or use with Streamlit
st.download_button("Download PDF", data=pdf_bytes, file_name="URS-7.1.pdf", mime="application/pdf")
```

**Validation Report PDF Structure:**

- **Page 1 — Cover (Portrait):** Title, doc ID, key-value summary (category, risk, strategy, script ID), requirement summary, assumptions, compliance notes, reg versions
- **Page 2 — UR/FR Table (Landscape):** UR summary metadata, FR table (FR ID, Parent UR, Statement, Acceptance Criteria)
- **Page 3 — Test Script Table (Landscape):** Script metadata, steps table (Type, #, Title, Instruction, Expected Result, Case, Ref)
- **Page 4 — Regulatory Justification (Portrait):** Full justification text (if present)
- **Page 5 — Manifestation of Signature (Portrait):** 21 CFR Part 11 signature page

**Validation Report Usage Example:**
```python
from utils.pdf_generator import generate_validation_report_pdf

# ur_fr from RequirementArchitect, test_script from DeltaAgent
pdf_bytes = generate_validation_report_pdf(
    ur_fr=ur_fr,
    test_script=test_script,
    signer_name="Jane Smith",
    meaning="Approval of Validation Report",
)

st.download_button("Download Validation Report", data=pdf_bytes, file_name="report.pdf", mime="application/pdf")
```

### Agents/delta_agent.py

**Delta Agent** - Facade for CSA testing strategy determination, test script generation, and deterministic CSA test script generation from UR/FR documents.

**Enums:**
- `CSATestType`: `INFORMAL`, `FORMAL_OQ`, `FORMAL_UAT`
- `StepType`: `SETUP`, `EXECUTION`
- `TestCaseType`: `POSITIVE`, `NEGATIVE`, `EDGE_CASE`

**Data Classes:**
- `CSATestStep`: step_type, step_number, step_title, step_instruction, expected_result, test_case_type, requirement_reference
  - `to_dict()` → serializes to dict matching Excel column structure

**Exception Classes:**
- `DeltaAgentError` (CSV-012) - Delta agent processing failed

**Core Methods:**

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `determine_testing_strategy()` | risk_level: RiskLevel | TestingStrategy | CSA strategy from risk level |
| `generate_test_script()` | urs: dict | dict | Generate test script from URS (LLM-based) |
| `generate_test_batch()` | urs_list: List[dict] | List[dict] | Batch test script generation |
| `generate_csa_test_from_ur_fr()` | ur_fr: dict, test_type: str | dict | Generate CSA test from UR/FR (deterministic) |
| `generate_csa_test_batch()` | ur_fr_list: List[dict], test_type: str | List[dict] | Batch CSA test generation from UR/FR |

**Static Helper Methods:**

| Method | Purpose |
|--------|---------|
| `_build_setup_steps()` | Common setup steps (login, navigate, prepare data) |
| `_build_positive_steps()` | Positive execution steps per FR |
| `_build_negative_steps()` | Negative execution steps per FR |
| `_build_edge_case_steps()` | Edge-case execution steps per FR |
| `_build_uat_steps()` | UAT business-process steps (end-to-end) |
| `_build_charter_steps()` | Unscripted exploratory charter steps |
| `_build_quality_checklist()` | Self-check quality dict |

**Routing Logic (``generate_csa_test_from_ur_fr()``):**

| Risk Level | Test Type | Steps Generated |
|------------|-----------|-----------------|
| High | Informal | Setup + positive + negative + edge per FR |
| High | Formal OQ | Setup + positive per FR |
| High | Formal UAT | Setup + UAT business-process per FR |
| Medium/Low | Any | Charter (exploratory) steps |

**Scripted Output (High risk):**
```python
{
    "script_id": "TS-URS-7.1",
    "urs_id": "URS-7.1",
    "ur_id": "UR-1",
    "test_type": "Informal",
    "risk_level": "High",
    "test_strategy": "OQ and/or UAT",
    "generated_at": "2024-...",
    "steps": [
        {
            "step_type": "Setup",
            "step_number": 1,
            "step_title": "Login as System Owner",
            "step_instruction": "Log into the application...",
            "expected_result": "",
            "test_case_type": "",
            "requirement_reference": ""
        },
        {
            "step_type": "Execution",
            "step_number": 1,
            "step_title": "Verify FR-1 - Positive",
            "step_instruction": "...",
            "expected_result": "System records temperature...",
            "test_case_type": "Positive",
            "requirement_reference": "UR-1 / FR-1"
        }
    ],
    "quality_checklist": {
        "steps_clear_and_sequential": true,
        "expected_results_observable": true,
        "execution_steps_have_references": true,
        "test_types_assigned": true,
        "no_redundant_steps": true
    }
}
```

**Charter Output (Medium/Low risk):**
```python
{
    "script_id": "TC-URS-7.1",
    "urs_id": "URS-7.1",
    "ur_id": "UR-1",
    "test_type": "Informal",
    "risk_level": "Medium",
    "test_strategy": "Informal",
    "generated_at": "...",
    "steps": [
        {
            "step_type": "Setup",
            "step_number": 1,
            "step_title": "Establish test environment",
            "step_instruction": "Confirm system access...",
            "expected_result": "",
            "test_case_type": "",
            "requirement_reference": ""
        },
        {
            "step_type": "Execution",
            "step_number": 1,
            "step_title": "Exploratory: Verify FR-1 core functionality",
            "step_instruction": "Using tester expertise, exercise...",
            "expected_result": "Feature operates as intended...",
            "test_case_type": "Positive",
            "requirement_reference": "UR-1 / FR-1"
        }
    ],
    "quality_checklist": { ... }
}
```

**Audit Events Logged:**
1. `CSA_TEST_SCRIPT_GENERATED` - High risk scripted test generated (Compliance Impact: Validation Evidence)
2. `CSA_TEST_CHARTER_GENERATED` - Medium/Low risk charter generated (Compliance Impact: Validation Evidence)
3. `CSA_TEST_BATCH_GENERATED` - Batch CSA test generation completed (Compliance Impact: Validation Evidence)

**Dependencies:**
- None (fully deterministic, no LLM or Pinecone calls)

**Usage Example:**
```python
from Agents.delta_agent import DeltaAgent

agent = DeltaAgent()

# Assume ur_fr from RequirementArchitect.transform_urs_to_ur_fr()
ur_fr = {
    "urs_id": "URS-7.1",
    "requirement_summary": "The system shall track warehouse temperature.",
    "category": "General",
    "user_requirement": {
        "ur_id": "UR-1",
        "statement": "As a User, there will be track warehouse temperature...",
        "risk_assessment": "GxP Indirect",
        "implementation_method": "Configured",
        "risk_level": "High",
        "test_strategy": "OQ and/or UAT",
    },
    "functional_requirements": [
        {
            "fr_id": "FR-1",
            "parent_ur_id": "UR-1",
            "statement": "The system shall track warehouse temperature",
            "acceptance_criteria": ["Given/When/Then..."],
        }
    ],
}

# Generate informal scripted test (High risk)
script = agent.generate_csa_test_from_ur_fr(ur_fr, "Informal")
print(script["script_id"])  # "TS-URS-7.1"
print(len(script["steps"]))  # Setup + positive + negative + edge

# Generate OQ test (High risk, positive only)
oq = agent.generate_csa_test_from_ur_fr(ur_fr, "Formal OQ")

# Generate UAT test (High risk, business-process)
uat = agent.generate_csa_test_from_ur_fr(ur_fr, "Formal UAT")

# Batch generation
results = agent.generate_csa_test_batch([ur_fr], "Informal")
```

## URS Traceability Index

| URS ID | Requirement | Implemented In |
|--------|-------------|----------------|
| URS-1.1 | Accept change requests from ServiceNow | `API/main.py:receive_servicenow_change()` |
| URS-1.2 | Acknowledge receipt of change requests | `API/main.py:ChangeRequestResponse` |
| URS-2.1 | Maintain 21 CFR Part 11 compliant audit trail | `API/main.py:log_audit_event()` |
| URS-3.1 | Classify risk as Low, Medium, or High | `Agents/risk_strategist.py:RiskLevel` |
| URS-3.2 | Assess severity based on patient impact | `Agents/risk_strategist.py:Severity` |
| URS-3.3 | Assess occurrence likelihood | `Agents/risk_strategist.py:Occurrence` |
| URS-3.4 | Assess detectability of failures | `Agents/risk_strategist.py:Detectability` |
| URS-4.1 | Recommend testing strategy per CSA | `Agents/risk_strategist.py:TestingStrategy` |
| URS-4.2 | Calculate risk using GAMP 5 methodology | `Agents/risk_strategist.py:calculate_risk_score()` |
| URS-4.3 | Classify RPN into risk categories | `Agents/risk_strategist.py:_determine_risk_level()` |
| URS-4.4 | Recommend CSA testing strategy | `Agents/risk_strategist.py:get_csa_testing_strategy()` |
| URS-4.5 | Map external criticality to severity | `Agents/risk_strategist.py:map_criticality_to_severity()` |
| URS-4.6 | Map change type to occurrence | `Agents/risk_strategist.py:map_change_type_to_occurrence()` |
| URS-4.7 | Assess risk for all change requests | `Agents/risk_strategist.py:assess_change_request()` |
| URS-6.1 | Generate URS from natural language input | `Agents/requirement_architect.py:generate_urs()` |
| URS-6.2 | Classify requirements as High/Med/Low | `Agents/requirement_architect.py:Criticality` |
| URS-6.3 | Output structured URS documents | `Agents/requirement_architect.py:URSDocument` |
| URS-6.4 | Provide JSON output format | `Agents/requirement_architect.py:URSDocument.to_dict()` |
| URS-6.5 | Connect to Pinecone knowledge base | `Agents/requirement_architect.py:RequirementArchitect.__init__()` |
| URS-6.6 | Validate environment before processing | `Agents/requirement_architect.py:_validate_dependencies()` |
| URS-6.7 | Embed user input for similarity search | `Agents/requirement_architect.py:_get_embedding()` |
| URS-6.8 | Retrieve relevant GAMP 5 sections | `Agents/requirement_architect.py:_query_pinecone()` |
| URS-6.9 | Assess requirement criticality | `Agents/requirement_architect.py:_determine_criticality()` |
| URS-6.10 | Assign unique URS identifiers | `Agents/requirement_architect.py:_generate_urs_id()` |
| URS-6.11 | Provide regulatory justification | `Agents/requirement_architect.py:_build_regulatory_rationale()` |
| URS-6.12 | Format requirements per standards | `Agents/requirement_architect.py:_format_requirement_statement()` |
| URS-6.13 | Require regulatory context for URS | `Agents/requirement_architect.py:RegulatoryContextNotFoundError` |
| URS-6.14 | Return structured search results | `Agents/requirement_architect.py:SearchResult` |
| URS-6.15 | Search knowledge base for context | `Agents/requirement_architect.py:search()` |
| URS-7.1 | Generate URS documents from project input | `scripts/draft_urs.py:draft_urs()` |
| URS-7.2 | Parse requirements from description | `scripts/draft_urs.py:parse_requirements()` |
| URS-7.3 | Output URS as Markdown table | `scripts/draft_urs.py:generate_urs_table()` |
| URS-7.4 | Save URS to output/urs directory | `scripts/draft_urs.py:save_urs_document()` |
| URS-7.5 | Accept interactive user input | `scripts/draft_urs.py:interactive_input()` |
| URS-12.1 | Verify generated URS against GAMP 5 text | `Agents/verification_agent.py:VerificationAgent.verify_urs()` |
| URS-12.2 | Reject URS drafts that contradict regulatory guidance | `Agents/verification_agent.py:VerificationAgent.verify_urs()` |
| URS-12.3 | Report verification errors | `Agents/verification_agent.py:VerificationError` |
| URS-12.4 | Produce structured verification findings | `Agents/verification_agent.py:VerificationFinding` |
| URS-12.5 | Connect to Pinecone for verification queries | `Agents/verification_agent.py:VerificationAgent.__init__()` |
| URS-12.6 | Validate environment before verification | `Agents/verification_agent.py:VerificationAgent._validate_dependencies()` |
| URS-12.7 | Embed text for verification queries | `Agents/verification_agent.py:VerificationAgent._get_embedding()` |
| URS-12.8 | Retrieve GAMP 5 text for verification | `Agents/verification_agent.py:VerificationAgent._query_pinecone()` |
| URS-12.9 | Validate URS input before verification | `Agents/verification_agent.py:VerificationAgent._validate_urs()` |
| URS-12.10 | Detect criticality misclassification | `Agents/verification_agent.py:VerificationAgent._check_criticality_alignment()` |
| URS-12.11 | Verify rationale relevance | `Agents/verification_agent.py:VerificationAgent._check_rationale_relevance()` |
| URS-12.12 | Detect contradictions between URS and GAMP 5 | `Agents/verification_agent.py:VerificationAgent._check_contradictions()` |
| URS-12.13 | Support batch verification | `Agents/verification_agent.py:VerificationAgent.verify_batch()` |
| URS-13.1 | Archive AI reasoning alongside audit records | `Agents/integrity_manager.py:log_audit_event()` |
| URS-13.2 | Validate thought-process payload shape | `Agents/integrity_manager.py:_validate_thought_process()` |
| URS-13.3 | Write tamper-evident logic-archive JSON | `Agents/integrity_manager.py:_write_logic_archive()` |
| URS-13.4 | Cross-reference archive to CSV audit row | `Agents/integrity_manager.py:_write_logic_archive()` |
| URS-13.5 | Compute integrity hash for archive file | `Agents/integrity_manager.py:_write_logic_archive()` |
| URS-15.1 | Generate URS as professional PDF | `utils/pdf_generator.py:generate_urs_pdf()` |
| URS-15.2 | Append Manifestation of Signature page | `utils/pdf_generator.py:generate_urs_pdf()` |
| URS-15.3 | Include signer name, timestamp, and meaning | `utils/pdf_generator.py:generate_urs_pdf()` |
| URS-15.4 | Provide PDF download from Streamlit UI | `frontend/app.py` (Page 2) |
| URS-14.1 | Derive reg version from PDF filename at ingestion | `scripts/ingest_docs.py:_derive_reg_version()` |
| URS-14.2 | Store reg_version in Pinecone chunk metadata | `scripts/ingest_docs.py:DocumentChunk.to_metadata()` |
| URS-14.3 | Propagate reg_version through search results | `Agents/requirement_architect.py:SearchResult` |
| URS-14.4 | Include reg version in verification citations | `Agents/verification_agent.py:_format_gamp5_ref()` |
| URS-14.5 | Include reg version in URS rationale citations | `Agents/requirement_architect.py:_build_regulatory_rationale()` |
| URS-14.6 | Collect Reg_Versions_Cited per URS | `Agents/requirement_architect.py:generate_urs()` |
| URS-14.7 | Include regulatory version footer in URS document | `scripts/draft_urs.py:generate_urs_table()` |
| URS-14.8 | Detect new regulatory versions at ingestion | `scripts/ingest_docs.py:ingest_documents()` |
| URS-14.9 | Detect new regulatory versions at query time | `Agents/requirement_architect.py:search()` |
| URS-14.10 | Detect new regulatory versions during verification | `Agents/verification_agent.py:verify_urs()` |
| URS-14.11 | Include reg version in gap analysis citations | `Agents/ingestor_agent.py:analyze_gaps()` |
| URS-16.1 | Classify UR risk assessment and implementation method | `Agents/requirement_architect.py:RiskAssessmentCategory, ImplementationMethod` |
| URS-16.2 | Determine UR/FR risk level from matrix | `Agents/requirement_architect.py:_determine_ur_fr_risk_level()` |
| URS-16.3 | Determine UR/FR test strategy from matrix | `Agents/requirement_architect.py:_determine_ur_fr_test_strategy()` |
| URS-16.4 | Decompose URS into functional requirements | `Agents/requirement_architect.py:_split_requirement_to_frs()` |
| URS-16.5 | Generate acceptance criteria for FRs | `Agents/requirement_architect.py:_generate_acceptance_criteria()` |
| URS-16.6 | Transform URS to UR/FR document | `Agents/requirement_architect.py:transform_urs_to_ur_fr()` |
| URS-16.7 | Log UR/FR transformation to audit trail | `Agents/requirement_architect.py:transform_urs_to_ur_fr()` |
| URS-17.1 | Generate CSA test scripts from UR/FR documents | `Agents/delta_agent.py:generate_csa_test_from_ur_fr()` |
| URS-17.2 | Support informal, OQ, and UAT test types | `Agents/delta_agent.py:CSATestType` |
| URS-17.3 | Separate setup from execution steps | `Agents/delta_agent.py:_build_setup_steps()` |
| URS-17.4 | Classify steps as positive, negative, or edge case | `Agents/delta_agent.py:TestCaseType` |
| URS-17.5 | Produce tabular test steps with standard columns | `Agents/delta_agent.py:CSATestStep` |
| URS-17.6 | Generate UAT business-process test steps | `Agents/delta_agent.py:_build_uat_steps()` |
| URS-17.7 | Generate unscripted test charters for medium/low risk | `Agents/delta_agent.py:_build_charter_steps()` |
| URS-17.8 | Support batch CSA test generation | `Agents/delta_agent.py:generate_csa_test_batch()` |
| URS-18.1 | Generate combined Validation Report PDF | `utils/pdf_generator.py:generate_validation_report_pdf()` |
| URS-18.2 | Include cover page with summary metadata | `utils/pdf_generator.py:generate_validation_report_pdf()` |
| URS-18.3 | Include landscape UR/FR and test script tables | `utils/pdf_generator.py:_table_page()` |
| URS-18.4 | Include regulatory justification page | `utils/pdf_generator.py:generate_validation_report_pdf()` |
| URS-18.5 | Include Manifestation of Signature page | `utils/pdf_generator.py:generate_validation_report_pdf()` |
| URS-18.6 | Provide Validation Report download from Streamlit UI | `frontend/app.py` (Page 6) |
| URS-19.1 | Detect ambiguous language in human-written requirements | `utils/demo_comparison.py:detect_ambiguities()` |
| URS-19.2 | Detect missing regulatory controls | `utils/demo_comparison.py:detect_regulatory_gaps()` |
| URS-19.3 | Rewrite requirements to audit-ready format | `utils/demo_comparison.py:rewrite_requirement()` |
| URS-19.4 | Evaluate and score requirement quality | `utils/demo_comparison.py:evaluate_requirements()` |
| URS-19.5 | Inject AI output into .docx templates | `utils/demo_comparison.py:inject_into_docx()` |
| URS-19.6 | Display side-by-side comparison with cost analysis | `frontend/app.py` (Page 8) |
| URS-19.7 | Provide CSV, PDF, and Word Factory exports | `frontend/app.py` (Page 8) |
| URS-20.1 | Generate intelligence from requirements context | `Agents/intelligence_engine.py:IntelligenceEngine.generate_intelligence()` |
| URS-20.2 | Validate LLM dependencies for Intelligence Engine | `Agents/intelligence_engine.py:IntelligenceEngine.__init__()` |
| URS-20.3 | Produce structured intelligence package | `Agents/intelligence_engine.py:IntelligenceResult` |
| URS-20.4 | Generate Mermaid.js workflow diagram | `Agents/intelligence_engine.py:IntelligenceEngine._generate_mermaid_diagram()` |
| URS-20.5 | Categorise requirements and generate acceptance criteria | `Agents/intelligence_engine.py:IntelligenceEngine._analyse_requirements()` |
| URS-20.6 | Find security gaps in workflow vs. security matrix | `Agents/intelligence_engine.py:IntelligenceEngine._find_security_gaps()` |
| URS-20.7 | Display split-screen Intelligence Dashboard in UI | `frontend/app.py` (Page 2) |
| URS-20.8 | Provide editable Risk Rank toggle in Smart Table | `frontend/app.py` (Page 2, `p2_smart_table` data_editor) |
| URS-21.1 | Accept multi-section GxP requirement input | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine.refine_to_smart()` |
| URS-21.2 | Detect FDA/EMA 2026 AI Guidance triggers | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._detect_fda_ema_flags()` |
| URS-21.3 | Rewrite vague requirements to SMART format | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._deterministic_refine()` |
| URS-21.4 | Generate Negative Test Scenarios for high-risk requirements | `Agents/smart_requirements_engine.py:_NEGATIVE_TEST_TEMPLATES` |
| URS-21.5 | Support LLM and deterministic refinement modes | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._llm_refine()` |
| URS-21.6 | Produce structured SMART requirement output | `Agents/smart_requirements_engine.py:SMARTRequirement` |
| URS-21.7 | Return aggregate summary statistics | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._compute_stats()` |
| URS-21.8 | Raise typed errors for engine failures | `Agents/smart_requirements_engine.py:SMARTEngineError` |
| URS-21.9 | Classify requirement risk level by keyword | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._keyword_risk()` |
| URS-21.10 | Generate template acceptance criteria | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._build_template_ac()` |
| URS-21.11 | Apply vague-word substitution to requirement text | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._apply_vague_substitutions()` |
| URS-21.12 | Ensure 'The system shall' prefix on all SMART requirements | `Agents/smart_requirements_engine.py:SMARTRequirementsEngine._ensure_shall_prefix()` |
| URS-21.13 | Display SMART Requirements Engine UI on Page 12 | `frontend/app.py` (Page 12) |
| URS-21.14 | Provide section tabs with SMART guidance help text | `frontend/app.py` (Page 12, `SMART_HELP_TEXT`) |
| URS-21.15 | Export SMART requirements to Generate Reqs page | `frontend/app.py` (Page 12, `sre_export_to_p2`) |
| URS-22.1 | Provide regulatory citations per test archetype | `Agents/regulatory_citations.py:CITATION_MAP` |
| URS-22.2 | Map archetype → citation list | `Agents/regulatory_citations.py:citations_for()` |
| URS-22.3 | Map risk level → bundle-level citations | `Agents/regulatory_citations.py:citations_for_risk_level()` |
| URS-22.4 | Generate risk-adaptive test bundles from risk-ranked requirements | `Agents/test_authoring_engine.py:TestAuthoringEngine.generate_bundle()` |
| URS-22.5 | Compute test depth from risk level (FULL/STANDARD/MEDIUM/CHARTER) | `Agents/test_authoring_engine.py:TestDepth` |
| URS-22.6 | Build setup + execution + UAT + charter step blocks | `Agents/test_authoring_engine.py:_build_setup_block()`, `_build_execution_block()`, `_build_uat_block()`, `_build_charter()` |
| URS-22.7 | Optionally enrich steps with LLM (hybrid mode, silent fallback) | `Agents/test_authoring_engine.py:_enrich_with_llm()` |
| URS-22.8 | Support batch test bundle generation | `Agents/test_authoring_engine.py:generate_batch()` |
| URS-22.9 | Persist test bundles atomically as JSON | `Agents/test_authoring_engine.py:_persist()` |
| URS-22.10 | Expose POST /test-authoring/generate endpoint | `API/routers/test_authoring.py:generate_bundle()` |
| URS-22.11 | Expose POST /test-authoring/generate-batch endpoint | `API/routers/test_authoring.py:generate_batch()` |
| URS-22.12 | Expose GET /test-authoring/bundles + GET /test-authoring/bundle/{id} | `API/routers/test_authoring.py:list_bundles()`, `get_bundle()` |
| URS-22.13 | Render risk-adaptive Test Authoring UI as Design tab #2 | `react-platform/src/apps/design/TestAuthoring.jsx` |
| URS-22.14 | Persist test bundles in Zustand and promote to Verify | `react-platform/src/store/useAppStore.js:promoteBundleToScript()` |
| URS-22.15 | Surface authored bundles in Verify (pull + From Bundle pill) | `react-platform/src/apps/Verify.jsx` (header + NoScriptsState) |
| URS-23.1 | Author test bundles manually without AI generation | `react-platform/src/store/useAppStore.js:createManualBundle()` |
| URS-23.2 | Add / update / remove individual steps in manual bundles | `react-platform/src/store/useAppStore.js:addBundleStep(), updateBundleStep(), removeBundleStep()` |
| URS-23.3 | Recompute bundle quality checklist on every step edit | `react-platform/src/store/useAppStore.js:_recomputeQuality()` |
| URS-23.4 | Render manual-mode authoring pane (per-row inline editor) | `react-platform/src/apps/design/TestAuthoring.jsx` (ManualStepRow + Author Manually toggle) |
| URS-23.5 | Insert tester-authored adhoc steps mid-execution with hierarchical numbering | `react-platform/src/store/useAppStore.js:insertAdhocStep()` |
| URS-23.6 | Tag adhoc steps with `source:'tester-adhoc'` for audit distinguishability | `react-platform/src/store/useAppStore.js:insertAdhocStep()` |
| URS-23.7 | Surface ⚡ Adhoc badge with hover audit details (who/when/why) on adhoc steps | `react-platform/src/apps/Verify.jsx:StepRow` |
| URS-23.8 | Provide inline Insert Adhoc Step form between execution rows | `react-platform/src/apps/Verify.jsx:InsertStepForm` |
| URS-23.9 | Compute UR test-bundle coverage and identify uncovered URs | `react-platform/src/apps/design/CoverageMonitor.jsx:computeCoverage()` |
| URS-23.10 | Surface coverage gap banner with deep-link "Generate now" CTA | `react-platform/src/apps/design/CoverageMonitor.jsx:CoverageMonitor` |
| URS-23.11 | Hard-block Design phase completion when any GxP Direct UR is uncovered | `react-platform/src/apps/Design.jsx:DesignSpecTab` (Save gate) |
| URS-23.12 | Surface coverage gate as Release readiness checklist entry | `react-platform/src/apps/Release.jsx` (checks list IIFE) |
| URS-23.13 | Provide pre-lock QA Review screen filtering failed / blocked / adhoc steps | `react-platform/src/apps/verify/QAReviewPanel.jsx:QAReviewPanel` |
| URS-23.14 | Auto-suggest QA review checklist values from observed run state | `react-platform/src/apps/verify/QAReviewPanel.jsx:autoSuggest()` |
| URS-23.15 | Record reviewer attestation (4-point checklist + comments + signature timestamp) | `react-platform/src/store/useAppStore.js:setQaReview(), setQaReviewCheck(), markQaReviewSigned()` |
| URS-26.1 | Generate Validation Plan (VP) PDF — Phase-1 deliverable | `utils/pdf_generator.py:generate_validation_plan_pdf()` |
| URS-26.2 | Generate Design Specification (DS) PDF with traceability matrix | `utils/pdf_generator.py:generate_design_specification_pdf()` |
| URS-26.3 | Generate Validation Summary Report (VSR) PDF — Phase-6 closing artefact | `utils/pdf_generator.py:generate_validation_summary_report_pdf()` |
| URS-26.4 | Expose POST /exports/{validation-plan,design-specification,validation-summary-report} endpoints | `API/routers/exports.py:export_validation_plan(), export_design_specification(), export_validation_summary_report()` |
| URS-26.5 | Surface VP / DS / VSR Download buttons in React phase pages | `react-platform/src/apps/Plan.jsx`, `Design.jsx`, `Release.jsx` (handleExportVP / handleExportDS / handleExportVSR) |
| URS-27.1 | Expose audit rows via JSON API for the React Audit Trail Viewer (sortable / filterable, server-side severity + phase enrichment) | `API/routers/audit.py:get_all_audit()` |
| URS-27.2 | Expose per-row logic-archive JSON for AI reasoning drill-down by reasoning-hash prefix | `API/routers/audit.py:get_logic_archive()` |
| URS-27.3 | Expose lifecycle timeline endpoint with Mermaid journey source + phase counts | `API/routers/audit.py:get_audit_timeline()` |
| URS-27.4 | Generate signed audit-trail Inspection Export PDF from a filtered slice (cover + landscape table + Manifestation of Signature) | `utils/pdf_generator.py:generate_audit_export_pdf()`, `API/routers/audit.py:export_audit_slice_pdf()` |
| URS-27.5 | Render React Audit Trail Inspection Viewer with table / drill-down drawer / lifecycle timeline / PDF export modal | `react-platform/src/apps/AuditTrail.jsx` |
| URS-27.6 | Register Audit Trail as a top-level Tools-group app in the EVOLV sidebar | `react-platform/src/data/apps.js` (APPS + NAV_GROUPS Tools), `react-platform/src/App.jsx` (RENDERERS) |
| URS-27.7 | Audit-export endpoint emits 3-event audit triplet (RECEIVED / COMPLETED / FAILED) per EVOLV API rules | `Agents/integrity_manager.py:_IMPACT_MAP` (AUDIT_EXPORT_*), `API/routers/audit.py:export_audit_slice_pdf()` |
| URS-28.1 | Compute Living Traceability Matrix as a pure read-model over the React store (one row per UR, joining Risk → Bundle → Runs → Defects → Approvals → Status) | `react-platform/src/apps/TraceabilityMatrix.jsx:computeTraceability()` |
| URS-28.2 | Render the Living Traceability Matrix UI with five filter chips (All / GxP Direct / Has Gaps / Failed Tests / Released), search, sortable 10-column table, and side drawer with deep-link CTAs to upstream phases | `react-platform/src/apps/TraceabilityMatrix.jsx:TraceabilityMatrix`, `TraceDrawer` |
| URS-28.3 | Export the active filtered matrix slice as CSV client-side (proper quoting, filename slugified to project name) | `react-platform/src/apps/TraceabilityMatrix.jsx:handleExportCsv` |
| URS-28.4 | Produce a signed Traceability Matrix Inspection Export PDF (cover summary + landscape table + Manifestation of Signature) from the filtered slice | `utils/pdf_generator.py:generate_traceability_matrix_pdf()`, `API/routers/traceability.py:export_traceability_matrix_pdf()` |
| URS-28.5 | Traceability export endpoint emits the standard 3-event audit triplet (RECEIVED / COMPLETED / FAILED) per EVOLV API rules | `Agents/integrity_manager.py:_IMPACT_MAP` (TRACEABILITY_EXPORT_*), `API/routers/traceability.py:export_traceability_matrix_pdf()` |
| URS-28.6 | Register Traceability Matrix as a top-level Intelligence-group app in the EVOLV sidebar | `react-platform/src/data/apps.js` (APPS + NAV_GROUPS Intelligence), `react-platform/src/App.jsx` (RENDERERS) |
| URS-29.1 | Single warm-light theme as the product surface — Claude.ai-inspired palette (warm off-white #FAFAF7 base, pure white cards, warm near-black text #2A2825, warm borders #EAE7E1) replacing the previous dark default and dark/light dual-theme switcher | `react-platform/src/index.css` `:root` (warm tokens), `react-platform/src/store/useAppStore.js` (`theme: 'light'`, `toggleTheme` locked to no-op) |
| URS-29.2 | Bump base body typography from 14px → 15px to match AI-tool-familiar reading comfort (Claude / ChatGPT scale) for long audit sessions | `react-platform/src/index.css` (`body { font-size: 15px; line-height: 1.5; }`) |
| URS-29.3 | Remove the dark/light theme toggle from the top header (single-theme product surface) | `react-platform/src/shell/TopHeader.jsx` (toggle button block deleted, `theme` / `toggleTheme` removed from destructure) |
| URS-29.4 | Restyle the Cmd+K search overlay from hardcoded dark glass to warm-light glass so it reads as the focal element on the cream backdrop | `react-platform/src/shell/TopHeader.jsx` (overlay container + kbd / divider classes converted to `bg-bg-hover`/`border-border-base`) |
| URS-29.5 | Drop the "Validation Factory" sub-brand from working-page hero text and inline copy (sub-brand retained in Sidebar logo subtitle, footer attribution, and PDF / CLI banners per CLAUDE.md branding rules) | `react-platform/src/apps/Home.jsx` (hero subtitle), `Verify.jsx` (NoScriptsState empty-state copy), `Design.jsx` (no-requirements empty state), `Risk.jsx` + `Requirements.jsx` (data-bridge sync messages + re-sync button title) |
| URS-30.1 | Lifecycle-first sidebar — Intelligence + Tools nav groups collapse to a single header chevron by default so the first paint shows only the 8 V-model phases ("show only lifecycle first / non-busy" per April demo feedback). User's group-expansion choices persist across reload via Zustand. | `react-platform/src/store/useAppStore.js` (`navGroupsCollapsed` slice + `toggleNavGroup` action + `partialize`), `react-platform/src/shell/Sidebar.jsx` (group toggle delegates to store) |
| URS-30.2 | Hidden-activity dot on collapsed group headers — when a group is collapsed AND any item inside is the active tab or carries a non-`info` status badge (warning / error / success), an amber dot pulses next to the group label so the calm first-paint never silently buries activity. | `react-platform/src/shell/Sidebar.jsx` (`hiddenActivity` derivation in NAV_GROUPS map) |
| URS-30.3 | TopHeader diet — removed the EVOLV AI vanity badge, the three compliance pills (21 CFR 11 / GAMP 5 / FDA AI), and the standalone font-size button. Right-of-divider control count drops from 8 to 5 (search · project pill · audit · avatar). | `react-platform/src/shell/TopHeader.jsx` (compliance badges block + EVOLV AI badge + font-size button deleted; layout comment updated) |
| URS-30.4 | Font-size segmented control relocated into the avatar profile dropdown (preference bucket alongside name / role / org). 3-segment A / A+ / A++ pill mirrors the old standalone button's semantics; clicking a segment advances `cycleFontSize` until the requested size is reached. | `react-platform/src/shell/TopHeader.jsx` (segmented control inside profile dropdown, calls `useAppStore.getState().fontSize` for direct-set semantics over the cycle action) |
| URS-31.1 | Copilot-style chat-input hero on Home — single front-door textarea with first-name greeting, focus-state lime→blue gradient ring, autosizing rows (max 5), Enter-to-submit / Shift+Enter newline (Claude/ChatGPT convention). Replaces the old "EVOLV Platform" title + "EVOLV AI Active" badge so the AI is signalled by the interaction surface itself. | `react-platform/src/apps/Home.jsx` (`HeroPrompt` component) |
| URS-31.2 | Deterministic natural-language intent router — keyword map covering all 16 apps (8 lifecycle phases + 8 cross-cutting tools); first-match-wins ordering puts more-specific keywords (audit, traceability) before broader ones (verify, plan); falls back to Requirements when no keyword matches. No LLM call, so demos never fail unpredictably. | `react-platform/src/apps/Home.jsx` (`ROUTE_KEYWORDS` map + `routeIntent()` helper) |
| URS-31.3 | Four calibrated suggestion chips below the prompt — "Start a new validation" → Plan, "Continue test execution" → Verify, "Review portfolio status" → Portfolio, "Search the audit trail" → Audit Trail. Hint pill on each chip names the destination app for predictability. | `react-platform/src/apps/Home.jsx` (`SUGGESTIONS` const + chip render block in HeroPrompt) |
| URS-31.4 | Recent-apps tracking for the Cmd+K palette — FIFO list (max 5, deduped) populated as the user opens tabs. Home is excluded (it's pinned and would never feel "recently visited"). Persisted across browser refreshes via Zustand `partialize` so the second-time visitor's palette already feels personal. | `react-platform/src/store/useAppStore.js` (`recentApps` slice + `openTab` mutation + `partialize` block) |
| URS-31.5 | Command palette actions inside Cmd+K — global `run()`-backed actions exposed as searchable rows alongside apps: "Open Demo project", "Toggle audit drawer", "Reset reading size", "Go to Home", and one "Switch project: <name>" per registered project (excluding the active one). Built fresh each render so closures see latest store state. | `react-platform/src/shell/TopHeader.jsx` (`commands` useMemo + `pick()` dispatcher) |
| URS-31.6 | Sectioned Cmd+K overlay — empty-query state renders Recent / Commands / Lifecycle / Intelligence / Tools as named sections (curated picker); typed-query state collapses to a single flat Fuse-ranked "Results" section over apps + commands together so "demo"/"audit" jump straight to the right action. Keyboard ↑/↓ threads through all rows in render order via an absolute cursor index. | `react-platform/src/shell/TopHeader.jsx` (`sections` + `flatItems` useMemo + sectioned render block in Cmd+K overlay) |
| URS-32.1 | V-model hero SVG on Home with animated path draw-in on first paint — 8 phase nodes plotted as a V-shape, blue→lime→blue gradient stroke animates from 0 to full length over 1.2s ease-out via `strokeDasharray` / `strokeDashoffset` pattern; nodes fade in sequentially (i × 0.12s + 0.4s delay) so the user sees the lifecycle "draw itself" on landing. | `react-platform/src/apps/Home.jsx` (`VModelHero` component, `V_NODES` geometry, `V_PATH` polyline, `pathRef.getTotalLength()` + double-rAF trigger) |
| URS-32.2 | Live phase-progress strip — each of the 8 V-model nodes is colour-coded against `phaseCompletion`: lime + tick mark for done, blue with pulsing halo for the next-incomplete (active) phase, muted grey for not-started, locked styling for Retire. Node click opens that phase via `onPhaseClick` → `openTab(phaseId)` so the hero doubles as a permanent navigation surface. | `react-platform/src/apps/Home.jsx` (`VModelHero` node-render block — done/active/locked branches + label positioning above top nodes / below apex) |
| URS-32.3 | Natural-language "what's next" routing — `NEXT_PATTERNS` regex (`what's next`, `where am i`, `next phase/step`, `continue`, `pick up`, `resume`, `what should i do`) pre-checked inside `HeroPrompt.handleSubmit` before the keyword router runs; matched queries route to `nextPhase` (first incomplete lifecycle phase) so the chat front-door becomes context-aware about the user's progress. | `react-platform/src/apps/Home.jsx` (`NEXT_PATTERNS` regex + `HeroPrompt` `nextPhase` prop + `handleSubmit` pre-check) |
| URS-33.1 | Shared V-model geometry module — single source of truth (`PHASE_ORDER`, `V_NODES` for hero, `V_NODES_SPINE` for compact strip, both built from same x positions and linearly scaled y values). Hero and spine consume the same module so their nodes can never drift, and any future V-shape surface inherits the same brand visual for free. | `react-platform/src/shell/vmodelGeometry.js` (`V_NODES`, `V_PATH`, `V_NODES_SPINE`, `V_PATH_SPINE`, `PHASE_SHORT`) |
| URS-33.2 | LifecycleStrip upgraded from flat horizontal node row to compact V-shape SVG that mirrors VModelHero — same gradient stroke, same colour conventions, same click-to-jump behaviour, but compressed to a 96px expanded height (32px vertical V depth instead of 105px). The spine is no longer just a progress bar with circles; it's a literal scaled-down V-model. | `react-platform/src/shell/LifecycleStrip.jsx` (V-shape SVG render block, replaces old `PhaseNode` + `Connector` flat-row layout) |
| URS-33.3 | Active-phase emphasis on spine — current open phase tab shown in blue with a pulsing halo (overrides done-state lime); completed phases keep their lime + tick mark; locked phases (Retire pre-release) show as muted no-op. The spine answers "where am I?" instantly the moment a phase page renders, even mid-scroll. | `react-platform/src/shell/LifecycleStrip.jsx` (node render block — `isActive` branch in fill / stroke / labelClr ternaries; `activeTabId === n.id` precedence over `done`) |
| URS-34.1 | Single-textarea Brief intake on Requirements — paste a one-paragraph brief, get UR/FR drafts back. Submit on Enter (Shift+Enter for newline) matches Claude/ChatGPT chat-input convention. Optional "Use sample brief" link pre-fills a representative LIMS paragraph for demo presenters. Backend reuse: posts to the existing `/requirements/generate` endpoint with the brief in `system_description` plus sane defaults (`role:'User'`, `risk_assessment:'GxP Indirect'`, `implementation_method:'Configured'`); no new server code. | `react-platform/src/apps/requirements/BriefIntake.jsx` (`BriefIntake` component, `SAMPLE_BRIEF` const) |
| URS-34.2 | Stepwise narration during in-flight requirement generation — three-stage cycle ("Querying GAMP 5 corpus…" → "Extracting candidate requirements…" → "Classifying criticality + risk…") at 900ms intervals replaces a frozen spinner. Pulsing blue dot accompanies the active step. Resets to step 0 on each new submission so the cycle always begins where the user expects. Same UX trick Claude / Cursor use during long requests — the AI "thinks out loud." | `react-platform/src/apps/requirements/BriefIntake.jsx` (`PROGRESS_STEPS` const + `useEffect` interval cycling `stepIdx`) |
| URS-34.3 | Brief mode is the new default for the Requirements phase, prepended to the `MODES` array ahead of Workshop-Driven and Manual Authoring. Workshop-Driven retains the 9-field intake form for power users who want to control role / risk / implementation method and upload diagrams; Manual Authoring stays the escape hatch for hand-written rows. All three modes share the same `handleWorkshopGenerated` callback and busy/status setters so the row editor below behaves identically regardless of how the rows arrived. | `react-platform/src/apps/Requirements.jsx` (`MODES` array — `'brief'` first, `useState('brief')` default, `BriefIntake` render branch above `WorkshopIntake`) |
| URS-35.1 | Project Templates strip on the Plan phase — five deterministic one-click templates (LIMS, eQMS, ERP/MES, CTMS, Custom/Bespoke) covering ~90% of pharma CSV portfolios. Each template seeds projectName, GAMP category, system description, project scope, and the regulatory frameworks universally expected for that system class. Zero backend code: applies via the existing `setPlanData` setter, one call per field. | `react-platform/src/apps/Plan.jsx` (`PROJECT_TEMPLATES` const — five entries with `id / icon / name / blurb / plan` shape) |
| URS-35.2 | PlanTemplates strip auto-collapses once the user either applies a template or types into projectName themselves — collapsed view shows a single-line pill with "Use a template" / "Switch template" link so the strip never squats on the canvas after first use. Open view renders cards in a responsive auto-fill grid (200px min) with active-template highlighting and "✓ Applied" badge. Skip ✕ in the header lets the user dismiss without picking. | `react-platform/src/apps/Plan.jsx` (`PlanTemplates` component — `collapsed` state seeded from `planData.projectName`, `useEffect` watcher on `[planData.projectName, applied]`) |
| URS-35.3 | Plan.jsx renders `PlanTemplates` as the first section in the form body (above Project Details), wired through `appliedTemplate` / `setAppliedTemplate` state on the parent so the strip remembers which template was applied across re-opens. The strip uses brand-blue accent border + soft shadow inset to match BriefIntake's visual weight on Requirements — both are "AI-assisted shortcut" surfaces in the platform's visual vocabulary. | `react-platform/src/apps/Plan.jsx` (`Plan` component — `appliedTemplate` state + `<PlanTemplates>` render block above `<Section title="Project Details">`) |
| URS-35.5.1 | Plan → Requirements project-context wire (BriefIntake): the single-textarea front door now reads `planData.projectName` and `planData.systemDescription` from the store and forwards `project_name` in the POST body to `/requirements/generate`, plus concatenates the Plan-phase system description to the brief so the LLM gets full context. A brand-blue "for: {projectName}" pill in the header makes the wire-up visible; an amber "(no project — set name in Plan)" fallback nudges the user back to Plan if they skipped it. | `react-platform/src/apps/requirements/BriefIntake.jsx` (`useAppStore(s => s.planData)` import, `project_name` + concatenated `system_description` in POST body, project-context pill in header strip) |
| URS-35.5.2 | Plan → Requirements project-context wire (WorkshopIntake): the 9-field intake form's `projectName` and `systemDescription` fields now initialise from `planData` via lazy `useState` initialisers, so power users who arrive from Plan see their project pre-filled instead of an empty form. Local state is preserved on subsequent edits so the user can override without the Plan values stomping back over their work. | `react-platform/src/apps/Requirements.jsx` (`WorkshopIntake` — `planData = useAppStore(s => s.planData)`, `useState(() => planData?.projectName ?? '')`, `useState(() => planData?.systemDescription ?? '')`) |
| URS-35.5.3 | ChangeControlTab → active-project wire: the Monitor-phase change-control surface now synthesises an "active project" system entry from the live store (`planData` + `riskData` + `requirements` + `phaseCompletion` + `releaseData`) and pins it at the top of the system dropdown in its own `<optgroup label="★ Your active project">`. The CR form auto-selects the active project on mount, a brand-blue banner under the header announces "Active project pre-selected", and the dropdown visually separates real-project targets from demo-portfolio targets. The full Change Impact Assessment (which URs/bundles a CR actually affects, plus CCR generation and targeted revalidation) is Sprint 36 work — this wire is the prerequisite that makes the CR aware of the project at all. | `react-platform/src/apps/monitor/ChangeControlTab.jsx` (`buildActiveProjectSystem()` helper, `useMemo` over store slices, `useEffect` keeping dropdown in sync with renames in Plan, optgroup-split dropdown render, header banner) |
| URS-35.5.4 | Intelligence nav group flipped to default-expanded (Sprint 30 revision): the Sprint 30 navigation diet collapsed Intelligence + Tools on first paint so only the 8 lifecycle phases were visible. In pre-launch validation walkthroughs, that hid the Living Traceability Matrix (Sprint 28 — the flagship audit-readiness artefact) plus Portfolio, Audit Trail, and the other read-side dashboards behind a chevron pharma QA leads kept missing. Intelligence is now default-expanded (7 apps visible from first paint). Tools stays collapsed (Dev Portal, Config, Academy, Docs — admin/secondary surfaces). The user's own collapse/expand choices still persist via Zustand `partialize`. Net effect: 8 → 15 apps visible on first paint, with the audit-and-traceability story now visible without a click. | `react-platform/src/store/useAppStore.js` (`navGroupsCollapsed` initial — `Intelligence: false`, comment block expanded with Sprint 35.5 rationale) |
| URS-35.5.5 | Orphan-component cleanup: `react-platform/src/apps/ValidationFactory.jsx` (33 LOC, Streamlit-iframe wrapper) was unreferenced anywhere in the React shell after the platform pivoted to React-native pages in earlier sprints. Deleted in the same pre-launch hygiene pass that flipped the Intelligence nav-group default. Module count dropped 482 → 481. No functional change — the Streamlit iframe (`http://localhost:8501`) is no longer surfaced in EVOLV's UI; pharma customers run the React shell as the primary surface per `.claude/rules/react-platform.md`. | (deleted) `react-platform/src/apps/ValidationFactory.jsx` |
| URS-37.1 | Explicit Permission Envelope per specialist function — machine-readable metadata declaring `allowed_actions`, `forbidden_actions`, `data_classifications_allowed`/`forbidden`, `requires_human_signoff_on`, `outputs_audited_via`, `rollback_eligible`, and `llm_usage` constraints. Six agents registered in v1.0.0: RequirementArchitect, VerificationAgent, RiskStrategist, DeltaAgent, ChangeImpactAgent (1.0.0-rc for Sprint 36), IntegrityManager. Borrowed from Salim Ismail ExO 3.0 *Agent Passport* pattern (Web3 smart-contract metadata) and aligned with Nuno Valério *Trust Architecture* bounded-autonomy thesis. Self-validates at import time — malformed passport raises and crashes the API server loudly rather than serving bad data. | `Agents/agent_passports.py` (`AGENT_PASSPORTS` dict, `list_agent_passports()`, `get_agent_passport()`, `validate_passport_shape()`) |
| URS-37.2 | Passport metadata surfaceable via JSON API for customer / auditor inspection. Two read endpoints with the standard 3-event audit triplet per the EVOLV API rules. Pharma QA director reads the full registry in ~5 minutes; FDA inspector asks for individual passport by name. | `API/routers/agents.py` (`GET /agents/passports`, `GET /agents/passports/{agent_name}`) |
| URS-37.4 | Agent Passports surfaced in Dev Portal as a dedicated tab. Each agent renders as a collapsible card with LLM-backed vs Deterministic pill, Rollback Eligible vs Append-only pill, expandable detail showing purpose + allowed/forbidden actions + data classifications + human-signoff outputs + audit-event triplet + LLM usage envelope. | `react-platform/src/apps/DevPortal.jsx` (`AgentPassportsPanel`, `PassportList`, new `passports` tab) |
| URS-38.1 | Standing eval set for every specialist function with deterministic pass/fail scoring — Salim Ismail ExO 3.0 "Trusted Evals" pillar #1 applied to EVOLV. v1.0.0 ships 10 golden test entries for RequirementArchitect, each declaring `must_contain_keywords`, `must_cite_frameworks`, `expected_criticality`, `acceptance_criteria_min`. Deterministic checks catch 80% of regressions for 0% LLM cost; LLM-as-judge for semantic similarity lands in Sprint 44. | `Agents/agent_evals.py` (`REQUIREMENT_ARCHITECT_GOLDEN_SET`, `EvalCheck`, `EvalResult`, `EvalRun` dataclasses) |
| URS-38.2 | Run evals on demand from the command line or from a Dev Portal action. `run_evals()` accepts an injectable `architect_factory` for unit testing without spinning up Pinecone + OpenAI. Failure modes captured per-eval (not raised) so a single broken eval doesn't take down the whole run. CLI entrypoint via `python -m Agents.agent_evals` with `--json` and `--out` flags. | `Agents/agent_evals.py` (`run_evals()`, `_cli()`) |
| URS-38.3 | Human-readable eval run summary. Formats EvalRun as a CLI-friendly summary block suitable for Logic Archive narratives and Dev Portal display. | `Agents/agent_evals.py` (`summarise_eval_run()`) |
| URS-39.1 | Architecture map doc — EVOLV explicitly mapped to Salim Ismail ExO 3.0 framework (MTP / DRIVE / SHAPE pillars, 6-layer Intelligence Stack, 4 Govern/Assure pillars, 4 Moats) and to Nuno Valério Trust Architecture vocabulary. Honest inventory of what EVOLV has shipped vs. what remains as named gaps with sprint targets (Sprint 40 Recursive Learning, Sprint 41 MTP-as-runtime-constraint, Sprint 42 Sense Layer, Sprint 43 Granular Rollback, Sprint 44 continuous Trusted Evals). Slide-ready for pharma QA-leader demos. | `docs/architecture.md` |
| URS-36.1 | Generate AI-drafted Change Impact Assessment from a Change Request + active project snapshot — Sprint 36's flagship capability. Given CR text + UR/FR list + risk_data + test_bundles + signed approvals, the ChangeImpactAgent identifies affected URs (Jaccard token overlap above threshold), inherits FRs from affected parents, names test bundles requiring revalidation, and lists approvals requiring re-attestation. Returns a structured `ChangeImpactAssessment` dataclass with `cia_id`, `summary`, affected_urs/frs/bundles, invalidated_approvals, risk_delta_summary, recommendation (`revalidate` / `no_revalidation_needed`), and a full `reasoning_chain` for explainability. Deterministic in Sprint 36 (predictable for demos); LLM/embedding match against the regulatory corpus lands in Sprint 40. The agent never modifies existing records, never triggers revalidation directly, never signs the CCR — bounded autonomy applied to change management. | `Agents/change_impact_agent.py` (`ChangeImpactAgent.assess()`, `ChangeImpactAssessment` dataclass, `_build_cia()`, `_overlap_score()`) |
| URS-36.2 | Every CIA generation writes a Logic Archive with inputs, reasoning steps, and outputs hash-linked to the audit trail. The CIA_GENERATED audit row includes the full `thought_process` payload that an inspector can replay to re-derive the assessment from inputs. Three-event triplet: `CIA_RECEIVED` / `CIA_GENERATED` / `CIA_FAILED` per the EVOLV API rules. | `Agents/change_impact_agent.py` (`ChangeImpactAgent.assess()` — thought_process arg to `log_audit_event`) |
| URS-36.3 | The agent never modifies existing URs or risk classifications. The CIA is a proposal only. Bounded autonomy: AI proposes, human signs the CCR, only then does the revalidation sub-run spawn. This is the trust-architecture move Nuno Valério and Salim Ismail both describe — the AI does not have the last word. | `Agents/change_impact_agent.py` (`AGENT_NAME` is read-only; passport `forbidden_actions` enumerates the gates) |
| URS-36.6 | Record signed Change Control Record for a generated CIA. Three-decision controlled vocabulary (`approve_revalidation`, `approve_no_revalidation`, `reject`) — no default decision per the Sprint 36 design call (force conscious selection). `CCR_RECEIVED` / `CCR_APPROVED` / `CCR_FAILED` audit triplet. The signed CCR is the human-signature gate that authorises revalidation; without it nothing propagates. 21 CFR Part 11 §11.50 compliant — signer name + role + meaning + UTC timestamp captured. | `Agents/change_impact_agent.py` (`sign_ccr()` module-level function) |
| URS-36.7 | Expose CIA generation via JSON API. `POST /change-control/cia` accepts a CR + project snapshot Pydantic body (with example payload in the OpenAPI schema), routes to `ChangeImpactAgent.assess()`, returns the full CIA dict. HTTPException 400 for invalid project snapshot, 500 for unexpected errors. | `API/routers/change_control.py` (`CIARequest` Pydantic model, `generate_cia()` endpoint) |
| URS-36.8 | Expose CCR signing via JSON API. `POST /change-control/ccr` accepts cia_id + cr_id + signer details + decision, calls `Agents.change_impact_agent.sign_ccr()`, returns the signed CCR record. | `API/routers/change_control.py` (`CCRRequest` Pydantic model, `sign_ccr_endpoint()` endpoint) |
| URS-36.9 | Zustand `changeRecords` slice keyed by cr_id with three transition actions: `addChangeRecord(crId, payload)` for optimistic UI add, `attachCIA(crId, cia)` when the agent returns, `signCCR(crId, ccr)` after human signature. Status field walks `received` → `cia_generated` → `ccr_signed`. Persisted via partialize so a refreshed browser keeps the in-flight CIA state. | `react-platform/src/store/useAppStore.js` (`changeRecords`, `addChangeRecord`, `attachCIA`, `signCCR`, `clearChangeRecords`) |
| URS-36.10 | CIA viewer inline in Monitor → Change Control. Renders below the existing risk-assessment response only when the active project is selected and `Generate Change Impact Assessment (AI)` has been clicked. Shows summary, affected URs with risk before/after, downstream FRs, test bundles needing revalidation, prior approvals requiring re-attestation, and an expandable reasoning chain. Bottom action bar: gradient `✍ Sign Change Control Record` button opens the CCR modal; after signing, replaced by a green `✓ CCR Signed` card. | `react-platform/src/apps/monitor/ChangeControlTab.jsx` (`CIAViewer`, `CCRSignModal` components, `handleGenerateCIA`, `handleSignCCR`) |
| URS-37.1 | Per-UR Validated State Confidence score derived deterministically from observed signals — Sprint 37's flagship capability. Given a project snapshot (requirements + risk_data + test_bundles + test_runs + defects + change_records), the ValidatedStateEngine scores every UR 0-100 based on bundle staleness (days since last locked run × 0.10, max -25), open defect pressure (count × 5, max -25), CIA change-history density (recent CIA hits × 5, max -20), no-bundle penalty (-30 flat), no-risk penalty (-15 flat), and bonuses for recent successful re-verification (+10) and all FRs covered (+5). Tiers: green ≥ 80, yellow 50-79, red < 50. Returns full `ValidatedStateReport` dataclass with per-UR `URStateAssessment`, signal breakdown, suggested action, aggregate score + headline. Deterministic in v1.0.0 — LLM-augmented summaries land Sprint 38. The "EVOLV helps you STAY validated" answer to Nuno Valério's continuous-demonstration thesis and Salim Ismail's Recursive Learning DRIVE characteristic. | `Agents/validated_state_engine.py` (`ValidatedStateEngine.assess()`, `_build_report()`, `_score_one_ur()`, `URStateAssessment`/`ValidatedStateReport` dataclasses, `_suggested_action()`, `_headline()`) |
| URS-37.2 | Every assessment writes a Logic Archive with inputs, per-UR scoring steps, and aggregate outputs hash-linked to the audit trail. `STATE_ASSESSMENT_RECEIVED` / `STATE_ASSESSMENT_COMPLETED` / `STATE_ASSESSMENT_FAILED` triplet per the EVOLV API rules. Inspector can re-derive any score from the snapshot inputs alone — full audit-defensibility per `compliance_impact='Validation Continuity + AI Reasoning'`. | `Agents/validated_state_engine.py` (`assess()` thought_process payload + `_IMPACT_MAP` entries) |
| URS-37.3 | Engine never modifies existing records; never triggers revalidation; never signs approvals. Score is a proposal only — the human QA team decides whether to act on a yellow/red tier suggested action. Bounded autonomy applied to the validation-continuity loop, declared in the agent passport's `forbidden_actions` block. | `Agents/agent_passports.py` ("ValidatedStateEngine" passport v1.0.0 — forbidden_actions enumerates the gates) |
| URS-37.7 | Expose Validated State assessment via JSON API. `POST /validated-state/assess` accepts a project snapshot Pydantic body (with example payload in OpenAPI schema), routes to `ValidatedStateEngine.assess()`, returns the full report dict including per-UR scores, signal breakdowns, suggested actions, and aggregate headline. HTTPException 400 for invalid snapshot, 500 for unexpected errors. | `API/routers/validated_state.py` (`StateAssessRequest` Pydantic model, `assess_validated_state()` endpoint) |
| URS-37.8 | Zustand `validatedState` slice with `report`/`byUrId`/`loading`/`error`/`lastFetched`. Three setters: `setValidatedStateLoading`, `setValidatedStateReport` (denormalises assessments into byUrId map for fast lookup), `setValidatedStateError`. Persisted via partialize so the latest assessment survives browser reloads. | `react-platform/src/store/useAppStore.js` (`validatedState` initial + 3 setters + `clearValidatedState`) |
| URS-37.9 | Living Traceability Matrix gains the "State" column rendering a `StatePill` (green/yellow/red dot + numeric score) per UR. Header banner appears after an assessment: shows aggregate score + headline + tier counts + assessment time. Gradient `🧠 Assess Validated State` button triggers the POST to `/validated-state/assess`. Drawer drill-down adds a new "Validated State" section with the score card, suggested action, and full signal-by-signal breakdown showing `−N for stale bundle` / `−N for open defect` etc — the audit-defensible reasoning trail an inspector reads. | `react-platform/src/apps/TraceabilityMatrix.jsx` (`StatePill` + `STATE_TONE`, `handleAssessVse` callback, aggregate banner, `<th>State</th>` + `<td><StatePill/></td>` column, drawer Validated State section) |
| URS-40.1 | Bounded Autonomy Profile (BAP) engine — Sprint 40's flagship capability. The diagnostic stack that wraps EVOLV's Sprint 39 Trustworthiness Report. Where the TWR organises evidence around external frameworks (NIST / FDA GMLP / ISO 22989), the BAP runs a Context of Use through three diagnostic layers (Impact Class → Failure Envelope → Control Sustainability) and outputs a proportional assurance tier (BAP-0 Productivity → BAP-1 Advisory → BAP-2 Controlled Drafting → BAP-3 Decision-Support → BAP-4 Bounded Action, or BAP-X Out-of-Envelope exclusion). Includes 7-question Assurance Argument with Q7 Fragility Markers — named assumptions + watch signals + owner roles that, if shifted, would invalidate the safety case. Bounded autonomy: engine never modifies records, never signs approvals, never overrides exclusion verdicts. Deterministic in v1.0.0. | `Agents/bounded_autonomy_profile.py` (`BoundedAutonomyProfileEngine.assess()`, `_build_profile()`, `_impact_class()`, `_failure_envelope()`, `_control_sustainability()`, `_assign_tier()`, `_assurance_argument()`, `ImpactClass` / `FailureEnvelope` / `ControlSustainability` / `FragilityMarker` / `AssuranceArgument` / `BoundedAutonomyProfile` dataclasses) |
| URS-40.2 | Context of Use as the unit of BAP assessment. Same shape as Sprint 39's TWR COU so a single React call invokes both engines. Five required fields (customer_name, statement, gxp_classification, risk_level, decision_authority) plus optional context (target_system, integrates_with, poc_or_production). | `Agents/bounded_autonomy_profile.py` (`_validate_cou()` — raises `InvalidProfileInputError` CSV-042 with field-by-field error messages) |
| URS-40.3 | Five hard exclusion rules force BAP-X regardless of controls offered: EX-1-SIGN (AI executes electronic signature → 21 CFR Part 11 §11.50 violation), EX-2-RELEASE (AI releases batch/lot → QP responsibility under GMP), EX-3-CAPA (AI closes CAPA/deviation autonomously → §820.100 violation), EX-4-CLINICAL (AI makes clinical decision affecting patient → FDA SaMD territory), EX-5-VALIDATED-WRITE (AI writes validated record without human signature gate → §11.10(e) violation). The exclusion category is an EXCLUSION not a higher tier — temptation in pharma is to control upward; some risks don't yield to that. | `Agents/bounded_autonomy_profile.py` (`EXCLUSION_RULES` constant + `_check_exclusion_rules()` method) |
| URS-40.4 | Four-bucket Scenario Coverage classification per agent — Verified Safe / Verified Unsafe (Blocked) / Unmapped / Insufficient Evidence. Success metric shifts from "validation passed" to "moved scenarios from Unmapped or Insufficient Evidence into Verified Safe or Verified Unsafe (Blocked)". Coverage score 0-100 computed deterministically from the four bucket counts. | `Agents/bounded_autonomy_profile.py` (`SCENARIO_BUCKETS` constant, `ScenarioBucketEntry` dataclass, `_failure_envelope()` builds Approved Operating Envelope from Agent Passport allowed/forbidden actions union) |
| URS-40.5 | 7-question Assurance Argument structure with Q7 Fragility Markers as the differentiator move. Q1 Approved Purpose, Q2 Explicitly Out of Scope, Q3 Hazard Mechanisms, Q4 Controls per Hazard, Q5 Evidence per Control, Q6 Residual Risk Owners, Q7 Fragility Markers (assumption + if_broken_then + watch_signal + owner_role). A safety argument that doesn't name its own fragility is mostly a sales document. | `Agents/bounded_autonomy_profile.py` (`AssuranceArgument` + `FragilityMarker` dataclasses, `_assurance_argument()` method) |
| URS-40.6 | Per-tier Required Controls catalogue — closes the QA-director question "what do I need by Monday for BAP-2". Tier-graduated control lists (BAP-0 = 2 controls, BAP-2 = 9 controls, BAP-4 = everything BAP-3 plus 4 stricter ones, BAP-X = REFUSE + re-scope path). | `Agents/bounded_autonomy_profile.py` (`REQUIRED_CONTROLS_BY_TIER` constant) |
| URS-40.7 | Every BAP assessment writes a Logic Archive with inputs, per-layer reasoning steps, and tier output hash-linked to the audit trail. `BAP_ASSESSMENT_RECEIVED` / `BAP_ASSESSMENT_COMPLETED` / `BAP_ASSESSMENT_FAILED` triplet per the EVOLV API rules with `compliance_impact='AI Trustworthiness + Bounded Autonomy'`. Inspector can re-derive any tier verdict from the COU inputs alone. | `Agents/bounded_autonomy_profile.py` (`assess()` thought_process payload + `Agents/integrity_manager.py` `_IMPACT_MAP` entries for BAP_*) |
| URS-40.8 | Expose BAP tier ladder + per-tier required controls + scenario buckets via JSON API. Read-only. Pharma evaluators read this first to understand the proportional control system EVOLV uses to scale assurance against deployment risk. | `API/routers/bap.py` (`GET /bap/tiers` endpoint returning `BAP_TIERS` + `REQUIRED_CONTROLS_BY_TIER` + `SCENARIO_BUCKETS`) |
| URS-40.9 | Expose the five hard exclusion rules via JSON API. Reviewing this list is the fastest way for a pharma QA director to understand EVOLV's honesty contract: we will refuse deployments in certain shapes regardless of revenue at stake. | `API/routers/bap.py` (`GET /bap/exclusion-rules` endpoint serialising `EXCLUSION_RULES` to JSON-safe dicts) |
| URS-40.10 | Expose full BAP assessment + lightweight pre-flight exclusion check via JSON API. `POST /bap/assess` runs the full three-layer stack; `POST /bap/check-exclusion` is the sales-grade 30-second screen returning just `would_be_excluded` + `rules_fired` + `verdict`. Pre-flight check used during sales discovery calls + partner customer-screening + LinkedIn newsletter reply qualification. | `API/routers/bap.py` (`POST /bap/assess` + `POST /bap/check-exclusion` endpoints + `BAPAssessRequest` / `ExclusionCheckRequest` Pydantic models) |
| URS-40.11 | Generate signed PDF Bounded Autonomy Profile report — EVOLV-branded ~11-page format with prominent color-coded tier badge on cover (BAP-0 grey → BAP-2 lime → BAP-3 amber → BAP-X red), full three-layer diagnostic documentation, 7-question Assurance Argument with Fragility Markers as a dedicated page, and 5-signer Manifestation of Signature page (Business Owner · QA · Service Owner · System SME · AI Model SME) matching the pharma SOP RACI pattern. The TWR + BAP PDF pair is the new evidence pair pharma customers receive from EVOLV. | `utils/pdf_generator.py` (`generate_bounded_autonomy_profile_pdf()`, `_bap_tier_badge()`, `_bap_section()`, `_bap_q_block()`, `_bap_q_block_list()`, `_bap_5_signer_page()` helpers, `_BAP_TIER_COLOR` palette) + `API/routers/bap.py` (`POST /bap/pdf` endpoint, `BAPAssessPDFRequest` + `_BAPSignerMeta` Pydantic models) |
| URS-44.1 | Standing eval sets for every deterministic specialist function (RiskStrategist 12 · DeltaAgent 7 · ChangeImpactAgent 6 · ValidatedStateEngine 5 · BAPExclusionScreen 95 incl. generated variants) — 125 deterministic evals, zero LLM tokens, no API server needed | `Agents/eval_suite.py` (`AGENT_RUNNERS`, per-agent `run_*_evals()`, golden sets) |
| URS-44.2 | Optional LLM-as-judge scoring layer — small-Claude judge on text outputs when ANTHROPIC_API_KEY is set; silent skip otherwise so deterministic checks always stand alone | `Agents/eval_suite.py` (`_llm_judge()`, `apply_judge_to_run()`, `--judge` flag) |
| URS-44.3 | Single-command cross-agent suite run with aggregate scoreboard (`python -m Agents.eval_suite`, flags `--json` / `--out` / `--verbose` / `--agent` / `--include-llm`) | `Agents/eval_suite.py` (`run_suite()`, `summarise_suite()`, `_cli()`) |
| URS-44.4 | BAP exclusion-rule hardening caught by the eval suite: subject widened to (ai/llm/model), clause-bounded matching so rules cannot straddle sentence boundaries or leak into decision_authority, British verb forms + "signs off" + "recommends treatment" added, EX-5 human-gate suppressor word-bounds "with" so "without review" fires. Website screener JS kept in sync. | `Agents/bounded_autonomy_profile.py` (`_SUBJ`, `EXCLUSION_RULES`), `website/index.html` (JS `EXCLUSION_RULES`) |
| URS-45.1 | Audit-trail hash chaining (closes security finding SEC-9): every row's Reasoning_Hash incorporates the previous row's hash (SHA-256 over prev_hash + fields), computed atomically under the write lock with a per-file last-hash cache. Edits, deletions, and reorders of any chained row break every hash after it. Legacy pre-upgrade rows keep verifying against the original per-row formula. | `Agents/integrity_manager.py` (`_compute_chained_hash()`, `_get_prev_hash()`, `_read_last_row_hash()`, `CHAIN_GENESIS_HASH`, `log_audit_event()` chaining) |
| URS-45.2 | Full-chain verification an inspector can run on demand — walks the CSV, classifies each row CHAINED / LEGACY / TAMPERED, flags legacy-after-chained downgrade patterns, reports chain head hash for external anchoring (tail-truncation detection). 6 IntegrityManager evals in the Trusted Evals suite (edit / middle-delete / reorder detection). | `Agents/integrity_manager.py` (`verify_audit_chain()`, `ChainVerificationReport`), `scripts/verify_audit_chain.py` (CLI, exit 0/1), `Agents/eval_suite.py` (`run_integrity_manager_evals()`) |
| URS-45.3 | Expose chain verification via JSON API with the standard 3-event audit triplet (AUDIT_CHAIN_VERIFY_RECEIVED / COMPLETED / FAILED) — the verification rows are themselves chained, so verifying the trail extends the trail. | `API/routers/audit.py` (`GET /audit/verify-chain`), `Agents/integrity_manager.py` (`_IMPACT_MAP` AUDIT_CHAIN_VERIFY_*) |
| URS-46.1 | Expose the Trusted Evals suite via JSON API — `GET /evals/agents` (registered agents + counts) and `POST /evals/run` (full suite or single agent, synchronous, deterministic) with the EVAL_SUITE_RUN_* audit triplet. Eval runs invoke real agents, so every Dev Portal run appends chained audit evidence. | `API/routers/evals.py` (`list_eval_agents()`, `run_eval_suite()`), `API/main.py` (router registration), `Agents/integrity_manager.py` (`_IMPACT_MAP` EVAL_SUITE_RUN_*) |
| URS-46.2 | Trusted Evals tab in the Dev Portal — run the 131-check suite from the UI with live scoreboard (per-agent pass rates, expandable failing-eval drill-down) plus one-click audit-chain verification showing intact/broken status, chained-vs-legacy row counts, and the chain head hash for external anchoring. | `react-platform/src/apps/DevPortal.jsx` (`TrustedEvalsPanel`, `evals` tab) |
| URS-43.1 | Platform security hardening (Sprint 43): optional app-wide API-key gate, filename sanitisation, CORS allow-list — renumbered from URS-SEC-1 so the compliance gate's numeric tag matcher recognises it | `API/security.py` (all public functions) |
| URS-47.1 | Deployment-context regulatory configuration | `Agents/compliance_context.py` |
| URS-47.2 | Tenant nomenclature mapping | `Agents/metadata_mapper.py` |
| URS-47.3 | Sentinel change impact analysis | `Agents/sentinel/impact_engine.py`, `Agents/sentinel_impact_agent.py` |
| URS-47.4 | Sentinel impact assessment report generation | `Agents/sentinel/justification_engine.py` |
| URS-47.5 | Test Pilot execution and reporting | `Agents/test_pilot.py` |
| URS-47.6 | Audit logging decorator | `utils/audit_decorator.py` |
| URS-47.7 | Bulk job progress tracking | `API/job_store.py` |
| URS-47.8 | Scoped API key lookup | `API/key_store.py` |
| URS-47.9 | Project persistence store | `API/project_store.py` |
| URS-47.10 | Plan data lifecycle (reset endpoint) | `API/routers/plan.py:clear_plan()` |
| URS-47.11 | Webhook delivery records | `API/webhook_registry.py` |
| URS-47.12 | AST-based compliance gate: URS tags checked in the full docstring (not a 10-line window); boilerplate names, properties, and overload stubs exempt; retired-brand scan word-bounded; CI pip-audit runs in requirements mode | `scripts/compliance_check.sh`, `scripts/validate_urs_tag.py`, `.github/workflows/compliance-check.yml` |
| URS-48.1 | Machine-readable component/model version registry with customer-facing changelog and change-notification commitment — answers the vendor-AI governance dealbreaker "how will we be notified when the model is updated?" | `Agents/version_registry.py` (`COMPONENT_REGISTRY`, `VERSION_CHANGELOG`, `get_registry()`), `API/routers/versions.py` (`GET /versions/registry`) |
| URS-48.2 | Upstream foundation-model drift detection — runtime model-ID observations compared against registry declarations; mismatch logs UPSTREAM_MODEL_CHANGED to the chained audit trail (EVOLV governs its own AI suppliers by the standard customers govern EVOLV) | `Agents/version_registry.py` (`record_model_observation()`, `OBSERVATIONS_PATH`), `Agents/eval_suite.py` (`_llm_judge` hook) |
| URS-49.1 | Customer bring-your-own-golden-set eval harness — customers run THEIR requirement expectations through the same eval engine EVOLV uses on itself; deterministic shape validation (`--validate-only`) needs no credentials; CLI `python -m Agents.customer_evals --file set.json` | `Agents/customer_evals.py` (`validate_golden_set()`, `load_golden_set()`, `run_customer_evals()`, CSV-061) |
| URS-49.2 | Eval pass-rate history/trending — every suite run (CLI, CI, Dev Portal, dossier) appends to `output/eval_history.jsonl`; exposed via `GET /evals/history` for trend display ("track performance over time") | `Agents/eval_suite.py` (`_persist_history()`, `get_eval_history()`, `EVAL_HISTORY_PATH`), `API/routers/evals.py` (`eval_history()`) |
| URS-49.3 | AI Incident & Deviation Runbook — ownership split (customer owns deviation, EVOLV owns AI-contribution investigation), Logic-Archive-replay protocol with SLAs, closure requires a new pinning eval + changelog entry, written walk-away trigger | `docs/ai-incident-runbook.md` |
| URS-49.4 | EU AI Act high-risk mapping (Articles 9–15, 17, 72) with honest gap inventory (formal QMS docs, Act-text corpus ingestion) | `docs/eu-ai-act-mapping.md` |
| URS-50.1 | Reproducibility proof for the deterministic specialist functions — each engine run repeatedly on fixed input must yield byte-identical output (provenance timestamps normalised, including ISO stamps embedded in narrative strings); wired into the Trusted Evals suite as the ReproducibilityHarness agent (5 evals) so output consistency is proven in CI and the Transparency Dossier. Output-consistency headline + OQ evidence for EVOLV's own validation. | `Agents/reproducibility.py` (`run_reproducibility()`, `_strip_volatile()`, `VOLATILE_KEYS`), `Agents/eval_suite.py` (`run_reproducibility_evals()`) |
| URS-50.2 | Self-validation package assembler — parses the 253-requirement URS Traceability Index from CLAUDE.md, attaches objective verification evidence per requirement, and assembles EVOLV's own Validation Plan (GAMP Cat 5), Installation Qualification (pinned deps + Docker + CVE status), and Operational Qualification (the eval suite executed live) into the standard GxP structure. "EVOLV validated with EVOLV." | `Agents/self_validation.py` (`parse_urs_index()`, `build_iq_baseline()`, `build_oq_summary()`, `build_validation_plan()`, `generate_self_validation_package()`), `API/routers/versions.py` (`GET /versions/self-validation/rtm`) |
| URS-50.3 | Signed self-validation package PDF — 12-page document (Validation Plan · IQ · OQ with live eval results · landscape Requirements Traceability Matrix · 21 CFR Part 11 signature page), latin-1-sanitised for RTM text drawn from the living index. SELF_VALIDATION_* audit triplet. Optional `redacted=true` public-safe mode collapses the RTM implementation column to the broad architecture layer (no file/function map) with a "PUBLIC SUMMARY" banner — legitimate IP protection for public sharing while the full package goes to qualified evaluators. The artifact a customer's QA files before deploying EVOLV. | `utils/pdf_generator.py` (`generate_self_validation_pdf()`, `_latin1_safe()`), `Agents/self_validation.py` (`_redact_impl()`, `_LAYER_MAP`), `API/routers/versions.py` (`POST /versions/self-validation`) |
| URS-48.3 | One-click signed AI Vendor Transparency Dossier — 4-page PDF answering the five procurement governance questions, assembled from LIVE data (eval suite executed, audit chain verified, registry snapshotted at generation time) with 21 CFR Part 11 attestation page | `utils/pdf_generator.py` (`generate_transparency_dossier_pdf()`), `API/routers/versions.py` (`POST /versions/dossier`, DOSSIER_GENERATION_* triplet) |
| URS-51.1 | Detect PII/PHI entities in free-text inputs before external transmission — deterministic regex + Luhn across email, phone, US SSN, credit card, IP, DOB, MRN, patient name; runs at the tenant boundary before text reaches OpenAI/Pinecone. Maps to big-pharma agentic-AI standard "Real-time PII detection" | `Agents/pii_shield.py` (`detect()`, `screen_text()`) |
| URS-51.2 | Redact detected entities deterministically (right-to-left span replacement with `[REDACTED:<CATEGORY>]` tokens) | `Agents/pii_shield.py` (`redact()`) |
| URS-51.3 | Enforce a configurable screen mode at the tenant boundary via `EVOLV_PII_MODE` (off \| warn [default] \| redact \| block); warn is behaviour-neutral so the shield is safe to enable everywhere | `Agents/pii_shield.py` (`configured_mode()`, `ScreenMode`) |
| URS-51.4 | Screen PII/PHI without persisting raw matched values to any log or artifact — findings carry category/sensitivity/offsets only; the audit trail records category counts, never values, so the trail never itself becomes a PII store | `Agents/pii_shield.py` (`PIIFinding`, `_log_screen()`, `PIIScreenResult.summary()`) |
| URS-51.5 | Integrate the shield into external-facing agent calls with a value-free audit event on detection (PII_SCREEN_FLAGGED / PII_SCREEN_BLOCKED) and a hard block option that raises before transmission | `Agents/pii_shield.py` (`screen_for_external_call()`), `Agents/requirement_architect.py` (`search()` pre-flight screen), `Agents/integrity_manager.py` (`_IMPACT_MAP` PII_SCREEN_*), `Agents/eval_suite.py` (`run_pii_shield_evals()`) |
| URS-51.6 | Retry transient external-call failures with bounded exponential backoff (transient-only classification: 429/5xx/timeouts/connection resets; 400/401/403 never retried) | `Agents/resilience.py` (`retry_call()`, `default_retryable()`) |
| URS-51.7 | Circuit-break a failing external dependency — trip OPEN after N consecutive failures, fail fast during the recovery window, then allow a single HALF_OPEN trial before closing | `Agents/resilience.py` (`CircuitBreaker`, `get_breaker()`) |
| URS-51.8 | Wrap the OpenAI embedding and Pinecone query calls with the combined retry + circuit-breaker (per-dependency breakers "openai" / "pinecone"), tunable via `EVOLV_RETRY_*` / `EVOLV_CB_*` | `Agents/resilience.py` (`resilient_call()`), `Agents/requirement_architect.py` (`_get_embedding()`, `_query_pinecone()`), `Agents/integrity_manager.py` (`_IMPACT_MAP` CIRCUIT_BREAKER_* / DEPENDENCY_RETRY_EXHAUSTED), `Agents/eval_suite.py` (`run_resilience_evals()`) |
| URS-51.9 | Expose a dependency health snapshot derived from circuit-breaker state (no live API calls), with an overall healthy flag for monitoring/alerting surfaces | `Agents/resilience.py` (`health_snapshot()`, `CircuitBreaker.snapshot()`) |
| URS-52.1 | Enforce unique-user attribution on human-decision audit events — a signature/approval/attestation recorded against a shared or generic identity is refused (enforce) or flagged (warn) before it enters the trail; closes FDA 211.68(b) "shared login / no attribution" | `Agents/attribution.py` (`guard_attribution()`, `screen_attribution()`), integrated into `Agents/integrity_manager.py` (`log_audit_event()`) |
| URS-52.2 | Deterministic shared/generic identity denylist (SYSTEM, admin, role names, blank, …) distinguishing a unique person from a non-attributable identity | `Agents/attribution.py` (`is_shared_identity()`, `_SHARED_IDS`) |
| URS-52.3 | Classify human-decision (attributable) actions vs system/plumbing events, so error/receipt paths can never trip or recurse through the guard | `Agents/attribution.py` (`is_attributable_action()`) |
| URS-52.4 | Configurable attribution mode off\|warn\|enforce via `EVOLV_ATTRIBUTION_MODE` (default warn = behaviour-neutral) | `Agents/attribution.py` (`configured_mode()`), `Agents/eval_suite.py` (`run_attribution_evals()`) |
| URS-52.5 | Verify the audit-trail chain at startup and refuse to run on a disabled/truncated/tampered chain (enforce via `EVOLV_TRAIL_ENFORCE`); closes FDA 211.68(b) "missing/disabled audit trail" | `Agents/integrity_manager.py` (`verify_trail_on_startup()`, `TrailIntegrityError`, `_IMPACT_MAP` INTEGRITY_VERIFIED/COMPROMISED), `API/main.py` (startup hook) |

## Coding Standards (GAMP 5 / CSA / 21 CFR Part 11)

### 1. Type Safety
Use Python type hints for every function. This prevents data integrity errors before they happen.

```python
# Correct
def calculate_risk(score: int, category: str) -> RiskLevel:
    ...

# Incorrect
def calculate_risk(score, category):
    ...
```

### 2. Documentation (The Traceability Rule)
Every function must have a docstring that includes a `:requirement:` tag linking it back to User Requirements (URS).

```python
def assess_change_risk(change_request: ServiceNowChangeRequest) -> RiskAssessment:
    """
    Evaluate the risk level of a ServiceNow change request.

    :param change_request: The incoming change request to assess.
    :return: RiskAssessment with level and justification.
    :requirement: URS-4.2 - System shall assess risk for all change requests.
    """
    ...
```

### 3. Audit Readiness
All API endpoints must log `user_id`, `timestamp`, and `action` to an immutable audit trail. No operation should occur without a traceable record.

```python
audit_logger.info(
    "API_CALL",
    user_id=user_id,
    timestamp=datetime.utcnow().isoformat(),
    action="CHANGE_REQUEST_RECEIVED",
    details={"cr_id": change_request.cr_id}
)
```

### 4. Error Handling
Use graceful exception handling with specific error codes. Silent failures are prohibited in a validated system.

```python
class CSVEngineError(Exception):
    """Base exception for CSV Engine errors."""
    pass

class ValidationError(CSVEngineError):
    """Error code: CSV-001 - Input validation failed."""
    error_code = "CSV-001"

class AuditLogError(CSVEngineError):
    """Error code: CSV-002 - Audit logging failed."""
    error_code = "CSV-002"
```

### 5. Style
Follow PEP 8 strictly. Readable code is auditable code.

- 4 spaces for indentation
- Maximum line length: 79 characters
- Two blank lines between top-level definitions
- One blank line between method definitions
- Use snake_case for functions and variables
- Use PascalCase for classes

---
> Source: [sreejisworld/CSV-GameChanger](https://github.com/sreejisworld/CSV-GameChanger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
