## regulatory-rtm-skill

> Create comprehensive Regulatory Requirements Traceability Matrices (RTM) from regulatory documents, legislation, rule changes, policies, or compliance standards. Use this skill whenever the user asks to analyse a regulation, extract requirements from a regulatory document, create a traceability matrix, map regulatory obligations, build a compliance matrix, or trace requirements from any official document like legislation, rules, standards, or policy papers. Also trigger when the user uploads a PDF or document and asks what obligations or requirements it contains, what they need to comply with, or wants the regulatory text broken down into plain English. This skill handles the analysis and structuring only, and is output-format agnostic. The user can request output as a Confluence page, Word document, Excel spreadsheet, or any other format.


# Regulatory Requirements Traceability Matrix (RTM) Skill

This skill analyses regulatory documents (legislation, rule changes, standards, policies) and produces a comprehensive Requirements Traceability Matrix that maps every obligation to its source, explains it in plain English, categorises it by business domain, and clearly flags what is verbatim from the source vs what has been derived or assumed.

## When to Use This Skill

Use this whenever the user provides a regulatory document and wants to understand what it requires. This includes new regulations, rule changes, amendments, compliance standards, policy papers, or any official document that creates obligations.

## Core Principle: Honesty About Sources

The most important principle in this skill is **never making things up and always being transparent about where information came from**. Every single entry in the RTM must be clearly marked as one of the following:

| Source Type | What It Means | When to Use |
|---|---|---|
| **VERBATIM** | Exact text copied directly from the regulatory document, word for word | When quoting the actual clause, rule, or requirement text |
| **DERIVED** | A requirement that is not stated word-for-word but is clearly implied or logically follows from the verbatim text | When the regulation says "participants must submit data monthly" and you derive that they need a data submission system |
| **ASSUMED** | Something that would typically be needed to comply but is not explicitly stated in the document | When you flag that training would likely be needed even though the regulation doesn't mention it. Always explain why this assumption was made |
| **INTERPRETED** | Where ambiguous regulatory language has been interpreted in a specific way | When a clause uses vague terms like "reasonable" or "appropriate" and you've interpreted what that means in practice |

If the user hasn't provided enough information to fill a field, leave it as "To be confirmed" rather than guessing.

## RTM Structure

Every requirement extracted from a regulatory document should be captured with the following fields. Read `references/rtm-fields.md` for detailed descriptions and examples of each field.

### Required Fields (always include these)

1. **Requirement ID** - Unique identifier (e.g., REQ-001, REQ-002)
2. **Source Reference** - Exact clause, section, rule, or schedule reference from the source document (e.g., "Schedule 1, Rule 3.2", "Section 14(2)(b)", "Clause 7.1.1")
3. **Source Document** - Name and version/date of the regulatory document
4. **Verbatim Text** - The exact text from the regulation, copied word for word. No paraphrasing, no shortening. This is the single source of truth
5. **Plain English Summary** - What this requirement actually means in simple, jargon-free language that a non-specialist could understand. Write as if explaining to someone who has never read the regulation
6. **Source Type** - One of: VERBATIM, DERIVED, ASSUMED, INTERPRETED (see table above)
7. **Business Domain** - Which area of the business this applies to (e.g., Billing, Metering, IT Systems, Customer Communications, Market Operations, Settlement, Registration, Compliance, Legal, Finance, Network Operations, Data Management)
8. **Obligation Type** - What kind of obligation this creates:
   - **MUST DO** - Mandatory requirement, no choice
   - **MUST NOT DO** - Explicitly prohibited
   - **MAY DO** - Permissive, optional
   - **CONDITIONAL** - Only applies if certain conditions are met
   - **REPORTING** - Requires reporting or disclosure
   - **RECORD KEEPING** - Requires maintaining records
   - **TIMELINE** - Specifies a deadline or timeframe
   - **ENTITLEMENT** - A right granted to one party that creates a corresponding obligation on another party (e.g., "customers are entitled to receive a bill" creates an obligation on the retailer to provide one)
9. **Priority** - Based on obligation type and consequences:
   - **CRITICAL** - Non-compliance has severe consequences (penalties, licence risk, safety)
   - **HIGH** - Mandatory requirement with significant impact
   - **MEDIUM** - Important but with less severe consequences
   - **LOW** - Minor or permissive requirements
10. **Compliance Status** - Current state (defaults to NOT ASSESSED for new analysis):
    - NOT ASSESSED, COMPLIANT, PARTIALLY COMPLIANT, NON-COMPLIANT, NOT APPLICABLE, IN PROGRESS
11. **Effective Date** - When this requirement comes into effect (if specified in the document)

### Recommended Fields (include when possible)

12. **Responsible Party** - Who within the organisation is likely responsible (if determinable from the regulation, e.g., "Registered Participants", "Retailers", "Network Service Providers")
13. **Dependencies** - Other requirements that this one depends on or is linked to (e.g., "REQ-003 must be met before this can be implemented")
14. **Change Type** - For amendments and rule changes:
    - **NEW** - Brand new requirement
    - **AMENDED** - Modification of an existing requirement
    - **DELETED** - Requirement being removed
    - **UNCHANGED** - Existing requirement not affected by this change
15. **Implementation Notes** - Any practical considerations for implementing this requirement
16. **Verification Method** - How compliance with this requirement would be verified:
    - **INSPECTION** - Review of documents or records
    - **TESTING** - Active testing of systems or processes
    - **DEMONSTRATION** - Showing that a process works
    - **ANALYSIS** - Analytical assessment
17. **Risk if Non-Compliant** - What happens if this requirement is not met (penalties, sanctions, licence conditions, safety risks)

## How to Analyse a Regulatory Document

Follow this process when the user provides a regulatory document:

### Step 1: Understand the Document
- Read the entire document (or section provided) carefully
- Identify what type of document it is (new legislation, amendment, rule change, standard, policy)
- Note the issuing authority and effective dates
- Identify who the document applies to (which types of organisations or roles)
- Read `references/document-types.md` for extraction tips specific to this type of document
- **For documents over 30 pages:** You MUST follow the "Processing Large Documents" section below. Build the Document Map, then delegate extraction to parallel sub-agents. Do NOT attempt to extract requirements from a large document in a single pass — you will skim and miss obligations. This is the #1 cause of incomplete RTMs

### Step 2: Extract Requirements Systematically

### Pre-scan: Understand the document before extracting

Before you start extracting clause by clause, do a quick scan of the full document:

1. **Read the definitions section first** and build a mental lookup. Defined terms change the meaning of words used throughout the document. A "small customer" might not mean what you think it means
2. **Identify the scope provisions** - Who does this document apply to? Which types of organisations, roles, or activities?
3. **Identify commencement and effective date provisions** - When do these obligations start? Are there different dates for different sections?
4. **Note the document structure** - Is it Parts > Divisions > Sections? Chapters > Rules > Clauses? This affects how you write source references
5. **Check the document type** and read the relevant section in `references/document-types.md` for type-specific extraction tips

This pre-scan shapes how you interpret every subsequent clause.

**THE #1 RULE: CAPTURE EVERYTHING. DO NOT SKIP ANYTHING.**

It is far better to include a requirement that turns out to be minor than to miss one that turns out to be critical. You are not the judge of what matters to the user's business. A clause that seems like a trivial administrative detail could be the one that triggers a compliance breach or audit finding. Extract it all and let the humans decide what's important.

**What to extract:**
- Every "shall", "must", "will", "is required to", "is obligated to" as a mandatory requirement
- Every "must not", "shall not", "is prohibited from" as a prohibition
- Every "may", "can", "is permitted to" as a permissive requirement
- Every "should" as a recommended practice (flag as DERIVED with note about "should" vs "must")
- Every definition that creates or modifies an obligation (definitions sections are full of hidden requirements)
- Every transitional provision, commencement date, or phased implementation detail
- Every record-keeping or documentation requirement, even if mentioned in passing
- Every reporting obligation, notification requirement, or disclosure duty
- Every timeframe, deadline, or period mentioned (even "as soon as practicable" counts)
- Every reference to penalties, enforcement, or consequences of non-compliance
- Every exception, exemption, or carve-out (these are just as important as the obligations themselves because they define the boundaries)
- Every cross-reference to other legislation, rules, or standards (flag these as needing separate review)
- For amendment documents: both the amendment text AND what it amends. If the amendment says "omit section X, substitute section Y", create a DELETED entry for old X and a NEW entry for new Y
- For "after paragraph (a) insert (aa)" style amendments: capture the full resulting text as it will read after the amendment takes effect
- Any references to the existing instrument that the user hasn't provided (flag these as needing the original document for complete analysis)
- Every governance requirement (board approval, committee oversight, delegation authorities)
- Every requirement buried in notes, footnotes, explanatory provisions, or schedules
- Implied requirements that aren't stated with obligation words but are clearly necessary for compliance

**Handling cross-references and regulatory hierarchy:**
- **"Despite" or "Notwithstanding" clauses** override other provisions. Flag both provisions and note the override relationship
- **"Subject to" clauses** are subordinate to another provision. Flag both and note which one takes priority
- **Delegated authority** ("as determined by the Authority", "as specified in guidelines") means additional requirements exist in a separate document. Flag for separate analysis
- **Incorporation by reference** ("in accordance with [standard]", "as specified in [code]") makes the external document's requirements binding. Flag each referenced document as needing its own RTM analysis

For a comprehensive list of obligation trigger words by category and jurisdiction, read `references/obligation-triggers.md`.

**How to extract:**
- Go through the document **sequentially, section by section, clause by clause**
- Do NOT jump around or cherry-pick what looks important
- Do NOT skip preambles, purpose statements, or scope sections as they often contain binding obligations or critical context that changes how other clauses should be interpreted
- Do NOT skip schedules, appendices, annexures, or attachments as these often contain the most detailed operational requirements
- Do NOT assume "administrative" or "procedural" clauses are unimportant
- If a single clause contains multiple obligations, extract each one as a separate requirement with its own row
- If you are unsure whether something is a requirement, include it and flag it with a note explaining your uncertainty
- **For documents over 30 pages:** this extraction MUST happen via parallel sub-agents as described in "Processing Large Documents" below. Each sub-agent focuses on one thematic section with a fresh context window, which prevents the skimming and rushing that happens when one agent tries to process the entire document. The orchestrator's job during extraction is delegation and coordination, not extraction itself

**After completing your extraction, do a completeness check:**
- Count the sections/clauses in the source document
- Verify you have at least one requirement from each substantive section
- If any section yielded zero requirements, go back and re-read it. Either it genuinely contains no obligations (rare) or you missed something
- List any sections you did not extract from and explain why (e.g., "Section 1 - Preliminary: contains definitions only, all captured under relevant requirement rows")

### Step 3: Categorise and Enrich
- Assign each requirement to the appropriate business domain
- Determine obligation type and priority
- Write the plain English summary for each
- Identify dependencies between requirements
- Flag any ambiguities or areas where interpretation was needed

### Step 4: Quality Check
Before presenting the RTM, verify:
- **Completeness audit** - Cross-check every section, clause, and schedule of the source document against your extracted requirements. If a section has no requirements extracted, explicitly state why
- Every requirement has a valid source reference that can be looked up in the original document
- Verbatim text is genuinely verbatim (not paraphrased)
- Plain English summaries don't introduce meaning that isn't in the original text
- Source type flags are honest (don't mark DERIVED requirements as VERBATIM)
- Business domain assignments make sense
- No requirements were missed (especially in schedules, appendices, definitions, transitional provisions, footnotes, and explanatory notes)
- ASSUMED entries clearly explain the reasoning
- **Extraction density sanity check** - As a loose guide (not a rule): primary legislation typically yields 2-5 requirements per page, detailed rules and standards 5-15 per page, and codes of practice 3-8 per page. If your count is far outside these ranges, re-examine. Some pages legitimately have zero requirements (definitions-only pages, preambles) but if most pages are coming up empty, you are probably missing obligations
- **Source type distribution** - If more than 20% of your entries are ASSUMED, flag this prominently in the summary. It suggests either the document is incomplete or you are over-interpreting. A healthy distribution for most documents is 70-80% VERBATIM, 15-25% DERIVED, 0-5% INTERPRETED, and under 5% ASSUMED
- Single clauses with multiple obligations have been split into separate requirements
- Exceptions and exemptions have been captured as standalone requirements (not just mentioned in passing)

### Step 5: Present the Summary
Before the detailed matrix, always provide a high-level summary that includes:
- Document title, issuing authority, and effective date
- Total number of requirements extracted (broken down by source type)
- Count by obligation type (how many MUST DO vs MAY DO etc.)
- Count by priority level
- Count by business domain
- **Coverage map** - A list of every section/part/schedule in the source document with the number of requirements extracted from each. Any section showing zero requirements must have an explanation (e.g., "Part 1 - Preliminary: 0 requirements - contains only interpretation provisions, definitions captured within relevant requirement rows"). This gives the user confidence that nothing was skipped
- Any key concerns, ambiguities, or areas flagged for human review
- Any limitations of the analysis (e.g., sections that couldn't be read, cross-references to other documents not provided)

## Output Format

This skill is **output-format agnostic**. The analysis and structure are the same regardless of how the user wants the output. Ask the user how they want the output if they haven't specified. Common options:

- **Confluence page** - Use the confluence-beautiful-pages skill if available for professional formatting with coloured headers, status lozenges, and panels
- **Word document (.docx)** - Use the docx skill for a professional document with tables
- **Excel spreadsheet (.xlsx)** - Use the xlsx skill for a filterable, sortable spreadsheet (often preferred for large RTMs)
- **In chat** - Present as formatted tables in the conversation

For very large documents (50+ requirements), recommend Excel or splitting across multiple pages/documents for manageability.

## Processing Large Documents (30+ pages)

Large regulatory documents (30+ pages) **must not** be processed in a single pass. Context pressure causes the AI to skim, rush, and silently skip sections — the exact failure mode that makes an RTM worthless. Instead, use a structured multi-agent pipeline: **Map → Extract (parallel) → Merge → Verify**.

### Phase 1: Document Map (orchestrator does this)

Before extracting a single requirement, the orchestrator must read the full document and produce a **Document Map**. This is a structured outline, not an extraction — it takes minutes and prevents every downstream mistake.

The Document Map must contain:

1. **Document metadata** — title, issuing authority, document type (legislation/amendment/rule change/standard/policy/final decision), publication date
2. **Structure inventory** — every Part, Chapter, Division, Section, Schedule, Appendix, and Annex with page ranges. Use the table of contents if one exists, but verify it against the actual document (tables of contents are sometimes outdated or incomplete)
3. **Definitions register** — every defined term and its definition, copied verbatim. This is critical because defined terms change the meaning of obligation clauses throughout the document. A sub-agent processing Chapter 6 cannot do its job correctly if it doesn't know that "small customer" has a specific regulatory definition from Chapter 1
4. **Scope provisions** — who does this document apply to? Which types of organisations, roles, activities, or jurisdictions?
5. **Effective dates** — all commencement dates, transition periods, and phased implementation schedules. If different sections commence on different dates, map each one
6. **Section boundaries for delegation** — divide the document into extraction chunks along **natural thematic boundaries** (chapters, parts, reform topics). Never split mid-section or mid-clause. Each chunk should be a self-contained regulatory topic that a sub-agent can fully understand

**Splitting rules:**
- Split at chapter, part, or major topic boundaries — never at arbitrary page counts
- If a chapter is very large (20+ pages with dense obligations), split at division or sub-topic boundaries within it
- If several chapters are short and closely related (e.g., "Administrative Updates" with 3 pages), combine them into one chunk
- Schedules, appendices, and annexes are their own chunks unless they are very short
- The goal is chunks that are coherent regulatory topics, roughly 10-30 pages each

### Phase 2: Parallel Extraction (sub-agents)

Spawn one sub-agent per chunk identified in the Document Map. **All sub-agents run in parallel.** Each sub-agent receives:

#### What every sub-agent MUST receive:

1. **Their assigned pages/sections** — the full extracted text of their chunk, every word
2. **The shared context preamble** — this is identical for every sub-agent and contains:
   - Document metadata (title, authority, type, date)
   - The complete definitions register from Phase 1
   - Scope provisions
   - All effective dates and transition schedules
   - The full Document Map (so the agent knows where their chunk sits in the overall document and what other sections exist)
3. **Their extraction assignment** — which sections they own, with explicit boundaries (e.g., "You are extracting requirements from Chapter 5: Protection for Customers Paying Higher Prices, pages 34-39. You own ONLY this section. Other agents are handling other chapters.")
4. **Requirement ID prefix** — each sub-agent gets a unique prefix to avoid ID collisions (e.g., "CH2-REQ-001", "CH5-REQ-001"). These get renumbered in the merge phase
5. **The full RTM field schema** — all required and recommended fields, so every sub-agent produces identically structured output
6. **Cross-reference instructions** — "If your section references another section of this document (e.g., 'as defined in clause 3' or 'subject to Part 6'), note the cross-reference in the Dependencies field but do NOT attempt to extract requirements from the referenced section — another agent owns that section"

#### Sub-agent extraction prompt template:

```
You are extracting regulatory requirements from ONE section of a larger document.

DOCUMENT: [title, authority, date]
YOUR SECTION: [section name, page range]
YOUR ID PREFIX: [e.g., CH5-REQ]

IMPORTANT CONTEXT — these apply to your section even though they come from elsewhere in the document:

DEFINITIONS:
[full definitions register]

SCOPE: [who this document applies to]

EFFECTIVE DATES: [all dates, especially any that apply to this section]

DOCUMENT STRUCTURE (for cross-references):
[Document Map]

YOUR TASK:
Extract EVERY requirement from your assigned section using the RTM schema below.
You are responsible for COMPLETENESS within your section — capture every obligation,
prohibition, permission, timeline, exception, exemption, reporting duty, and record-keeping
requirement. Do not skip anything. Do not summarise. Copy verbatim text exactly.

At the end, list every sub-section/clause in your assigned pages and confirm how many
requirements you extracted from each. If any sub-section yielded zero requirements, explain why.

RTM FIELDS:
[full field schema]
```

#### Sub-agent output format:

Each sub-agent returns:
- A JSON array of requirement objects (one per requirement, all fields populated)
- A coverage report listing every clause/sub-section in their chunk with the count of requirements extracted from each
- A list of cross-references to other sections (for the merge phase to validate)
- A list of any ambiguities or uncertainties flagged for human review

### Phase 3: Merge (orchestrator does this)

After all sub-agents complete, the orchestrator:

1. **Collects all requirement arrays** and concatenates them
2. **Renumbers requirement IDs** sequentially (REQ-001, REQ-002, ...) while preserving the original sub-agent IDs in a "Original ID" field for traceability
3. **Deduplicates** — if two sub-agents extracted the same obligation from overlapping references (e.g., a cross-reference restates an obligation from another section), keep the one with more complete verbatim text and note the duplicate
4. **Resolves cross-references** — where sub-agent A noted "depends on clause X" and sub-agent B extracted clause X as REQ-047, update the dependency to "Depends on REQ-047"
5. **Sequences dependencies** — requirements that depend on others should appear after their dependencies where possible
6. **Builds the coverage map** — combines all sub-agent coverage reports into a single document-wide map showing every section and its requirement count

### Phase 4: Verification (review sub-agent)

Spawn a **separate review sub-agent** that receives:
- The complete merged RTM
- The Document Map from Phase 1
- The full document text (or at minimum, the table of contents and section headers)

The review agent checks:

1. **Coverage completeness** — every section in the Document Map has at least one requirement, or an explicit explanation for why it has zero
2. **Density sanity check** — flag sections where the requirement count seems too low for the page count (e.g., a 15-page chapter with only 2 requirements is suspicious)
3. **ID continuity** — no gaps or duplicates in the final ID sequence
4. **Cross-reference integrity** — every dependency reference points to a valid requirement ID
5. **Source type distribution** — flag if ASSUMED > 20% or VERBATIM < 60%
6. **Missing obligation patterns** — scan the document for obligation trigger words ("must", "shall", "required to", "prohibited") and check that each instance maps to at least one extracted requirement
7. **Effective date consistency** — every requirement with a timeline has an effective date that matches the implementation schedule

The review agent returns a **verification report** with:
- PASS/FAIL for each check
- A list of specific gaps or issues found
- Recommended additional requirements to extract (with section references)

If the review agent identifies gaps, the orchestrator must either extract the missing requirements itself or spawn targeted sub-agents to fill the gaps.

### Output Splitting for Confluence

When outputting to Confluence pages, split along **thematic boundaries**, not page counts or requirement counts:

- **Hub page** — document summary, statistics, coverage map, implementation timeline, and navigation links to child pages
- **Child pages** — one per regulatory topic/chapter/reform area. A child page covers one coherent subject (e.g., "Automatic Best Offer Requirements", "Payment Method Access Requirements")
- If a single topic has more than ~10 requirements (roughly 25-30KB of ADF), split it into sub-pages (e.g., "Automatic Best Offer — Eligibility & Switching", "Automatic Best Offer — Opt-Out & Exceptions")
- Never split a requirement across two pages — each requirement row must be complete on one page
- Each child page should have a brief context paragraph at the top explaining what this section covers and how it fits into the overall regulation
- Page titles should be descriptive and human-readable (e.g., "ESC Energy Consumer Reforms — Ch 5: Higher Prices Protection") not generic ("RTM Page 3 of 8")

### For Excel or Word output

- Group by regulatory topic/chapter, not alphabetically or by requirement ID
- Include a summary/dashboard sheet/section first
- Each topic group should be visually separated with headers

## Important Warnings

- **CAPTURE EVERYTHING.** Do not self-filter. A requirement that seems trivial to AI could be the exact clause an auditor asks about. It is always better to have a row that turns out to be low priority than a gap that turns out to be a compliance breach
- **Never process large documents in a single pass.** For any document over 30 pages, you MUST use the parallel sub-agent pipeline described in "Processing Large Documents". Single-pass extraction of large documents is the #1 cause of incomplete RTMs. The AI skims under context pressure, silently drops sections, and produces a superficially complete but actually gap-ridden matrix. The sub-agent pattern exists specifically to prevent this — each agent gets a fresh context window focused on just their section
- **Never invent requirements.** If it's not in the document (explicitly or by clear implication), don't include it. Flag ASSUMED entries with clear reasoning
- **Never change verbatim text.** Copy the exact text, including any typos in the regulation
- **Always flag uncertainty.** If you're unsure about an interpretation, say so. Add a note like "This clause is ambiguous - the interpretation above assumes X, but it could also mean Y"
- **Cross-references matter.** If the regulation references another document you don't have, flag it as a standalone requirement. Don't guess what the referenced document says
- **Definitions matter.** Check the definitions section and apply those definitions. If a definition creates or narrows an obligation, extract it as a requirement. Pass the full definitions register to every sub-agent — a defined term in Chapter 1 changes the meaning of obligation clauses in Chapter 10
- **Exceptions are requirements too.** An exemption, carve-out, or exception needs its own row because it defines the boundaries of other obligations
- **Every sub-agent must know it has a piece, not the whole.** When delegating to sub-agents, be explicit: "You are extracting from Section X only. Other agents handle other sections. Do not attempt to cover the whole document. Focus exclusively on completeness within your assigned section."
- **Never silently skip sections.** If any section yields zero requirements, explain why. The verification agent will flag unexplained gaps

---
> Source: [paragpandyareal/regulatory-rtm-skill](https://github.com/paragpandyareal/regulatory-rtm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
