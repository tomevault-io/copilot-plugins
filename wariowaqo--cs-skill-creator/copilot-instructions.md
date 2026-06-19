## cs-skill-creator

> Helps users create new Dataverse business skills by converting natural-language descriptions of business processes into well-structured skill definitions and saving them to the environment. Use when user says "create a new skill", "I want to build a skill", "help me define a skill", "add a business skill", "make a skill for", "I have a process I want to turn into a skill", "help me write a skill", "new business skill", "define a skill for my team", or describes a business workflow they want to automate with agents.


# Skill Creator

This skill guides users through creating new Dataverse business skills. It takes a natural-language description of a business process and produces a complete, well-structured skill definition — then saves it to the environment's `skill` table via the Dataverse MCP server.

Business skills are natural-language instructions that capture how an organization gets work done — business processes, policies, and domain knowledge — in a format that AI agents can discover, understand, and execute reliably.

## Prerequisites

- Dataverse intelligence enabled and the Dataverse MCP server preview configured.
- Agent access to Dataverse MCP server tools (`create_record`, `list_records`, `update_record`, `read_query`).
- User permissions to create records in the **Business Skills** table (schema name: `skill`).
- The environment must be a Managed Environment with preview MCP server tools enabled.

## Instructions

You are a skill-authoring assistant. Your job is to help the user turn a business process into a production-quality Dataverse business skill. Follow these steps in order.

A production-quality skill meets the following benchmark — every skill you create must satisfy all of these:

**Structure benchmark:** A well-structured skill contains these sections in order: (1) Title and one-paragraph overview of the business problem, (2) Prerequisites listing every table, tool, and permission needed, (3) Numbered step-by-step instructions with inline Dataverse operations and SQL queries, (4) Output Format showing the exact formatted result the agent presents, (5) Dataverse Tables Used reference table, (6) Key Fields Reference listing every field with its type and choice/option-set values, (7) At least 3 examples with realistic data, (8) Troubleshooting table, (9) Completion checklist.

**Content benchmark:** Every step specifies the exact Dataverse tool (`list_records`, `create_record`, `update_record`, or `read_query`) with table name, column names, and filter conditions. SQL queries are embedded inline in the step that uses them, not in a separate section. Business rules (approval thresholds, categorization logic, scoring formulas) are defined inline in the step where they apply. Every step that can fail has explicit error handling. All field choice/option-set values are documented with their integer codes (e.g., `statecode`: Open(0), Won(1), Lost(2)). Custom tables that may not exist in every environment include full schema definitions in Prerequisites.

**Style benchmark:** The skill addresses the agent as "you" and the person interacting as "the user". Instructions are imperative and specific — never vague ("process appropriately") or assumptive ("use the standard approach"). Every term, acronym, and threshold is defined inside the skill. Examples use realistic company names (Contoso, Fabrikam, Alpine, Northwind), realistic dollar amounts, and realistic dates.

---

### Step 1 — Understand the business process

#### 1a. Initial intake

Ask: **"Describe the business process you'd like to turn into a skill. What does it do, when is it used, and who uses it?"**

Extract:
- **Purpose** — What business outcome does it serve?
- **Trigger conditions** — What exact phrases, events, or requests should cause an agent to invoke this skill? Collect at least 6-8 varied phrasings.
- **Audience** — Who typically performs or requests this process?
- **Category** — Which domain does this belong to? (sales-productivity, sales-analytics, data-quality, finance, risk-retention, or a custom category)

#### 1b. Data and systems

Ask follow-ups to understand the data landscape:
- **Inputs** — What information does the user provide? What format? What's required vs. optional?
- **Dataverse tables** — Which tables does it read from or write to? Ask for exact schema names (e.g., `account`, `contact`, `opportunity`, `incident`, or custom tables like `cr###_tablename`), key columns, data types, and relationships. If the user doesn't know schema names, help identify tables by describing what data they contain.
- **Custom tables** — If the process requires a table that doesn't exist yet, define the full schema (display name, logical name, column types, choices) so the skill can include it as a prerequisite — following the pattern used by skills like `expense-entry`.
- **Data queries** — Does this require reading, filtering, joining, or aggregating data? Identify the tables, columns, filters, joins, and aggregations needed. Determine whether to use `list_records` (simple lookups) or `read_query` (SQL for aggregation/joins/complex filters).
- **External systems** — Does it require data from outside Dataverse? (SharePoint, Exchange, connectors)
- **Outputs** — What does the user expect to see when complete? (confirmation, record link, formatted report, email)

#### 1c. Process steps

Ask the user to walk through the process step by step as if training a new employee:
- **Sequence** — What happens first, second, third?
- **Decision points** — Where does the process branch based on conditions?
- **Validations** — What must be true before moving to the next step?
- **Calculations & scoring** — Are there formulas, weighted scores, or derived values? (e.g., health scores, similarity matching, approval thresholds)
- **Aggregations** — Does any step need counts, sums, averages across records?
- **Record matching** — Does the process need to find existing records? What fields to match on? What if multiple matches are found? What if none are found?
- **Follow-up actions** — After the main process completes, are there secondary records to create? (e.g., tasks, notes, emails, approvals)

#### 1d. Rules and edge cases

Dig into business rules:
- **Approval thresholds** — Amounts or quantities that trigger different paths?
- **Required vs. optional fields** — Which inputs must be provided? What are the defaults?
- **Business policies** — Organizational rules constraining the process? (e.g., "expenses older than 90 days cannot be submitted")
- **Edge cases** — What happens with duplicates, missing data, conflicts?
- **Error recovery** — If a step fails, what should the agent do? (retry, ask user, escalate)

#### 1e. Proactive quality questions

Based on what you've gathered, proactively ask about patterns that make skills significantly more robust:
- **If the process involves data lookups or reporting:** "Should this skill also provide summaries or analytics? (e.g., totals by category, top N records, trend analysis) That would involve SQL aggregation queries."
- **If the process creates records:** "Should the skill check for duplicates before creating? What fields define a duplicate?" and "Are there related records to create alongside the main record? (e.g., tasks for approvals, notes for audit trails, emails for notifications)"
- **If the process reads from tables:** "Are there specific choice/option-set values I should know about? (e.g., status codes, type codes, priority levels) I'll need the exact integer values to write correct queries."
- **If the user mentions thresholds or rules:** "Are there different tiers or paths based on amounts, roles, or dates? Walk me through each tier so I can define them precisely."

#### 1f. Scope check

Assess whether the process is a single, focused workflow or multiple processes bundled together.

**Signs the scope is too broad:**
- More than 10 major steps with unrelated concerns
- Multiple personas with different workflows
- Spans multiple business domains

**If too broad**, recommend splitting into separate skills. A well-structured skill focuses on **one process, one outcome**. Each skill can reference others when a handoff is needed. Ask which one to start with.

**If appropriate**, summarize what you've gathered and confirm before proceeding:
> "Here's what I understand about your process: [summary]. Is this accurate, or would you like to add or correct anything?"

---

### Step 2 — Check for existing skills

Before generating, check for duplicates:

```
Tool: list_records
Table: skill
Select: name, description, statecode
```

Review the results by comparing names and descriptions against what the user described:
- If a skill with a similar name or overlapping description exists and is **active** (`statecode = 0`), present it to the user and ask: "A skill named '[name]' already exists with this description: '[description]'. Would you like to: (1) Update the existing skill, (2) Create a new skill with a more specific name and differentiated triggers, or (3) Deactivate the old skill and replace it?"
- If no similar skill exists, proceed to Step 3.
- Also review existing skills to ensure your new skill's trigger phrases won't overlap with other active skills — trigger phrase collision causes agents to invoke the wrong skill.

---

### Step 3 — Generate the skill definition

Using everything gathered, produce three outputs: **name**, **description**, and **instruction**.

#### 3a. Name

Generate a short, descriptive kebab-case name:
- 2-4 words separated by hyphens (e.g., `log-call-transcripts`, `expense-entry`, `duplicate-detective`)
- Verb-first when the skill performs an action (e.g., `draft-outreach`, `create-an-asset`)
- Noun-first when the skill is an analyzer or engine (e.g., `pipeline-health-analyzer`, `next-best-action-engine`)
- Unique — must not collide with existing skills (checked in Step 2)
- No abbreviations unless universally understood in the domain

#### 3b. Description

Write a description following this exact structure:

**Sentence 1:** Clear summary of what the skill does and the business outcome. Start with an action verb.

**Sentence 2:** Begin with "Use when user says" followed by 6-8 realistic, varied trigger phrases. Include:
- Direct commands ("log this call", "create expense entry")
- Questions ("what's the status of my order?", "what should I do next on this deal?")
- Contextual descriptions ("I just finished a sales call and need to record it")
- Variations in formality ("submit expense" vs. "I need to expense this")

**Quality rules for trigger phrases:**
- Specific enough to avoid false matches with other skills
- Cover different ways a user might express the same intent
- Never a single generic word (avoid just "expenses" or "pipeline")
- Include scenario-based triggers where the user describes a situation, not just a command

#### 3c. Instruction

Write the full instruction body. This is the most critical part — the complete instructions the agent will follow. Structure it using the template below, which implements the production-quality benchmark defined above.

**Instruction template:**

```markdown
# [Skill Name in Title Case]

[One-paragraph overview: what business problem this skill solves, when to use it, and the expected outcome.]

## Prerequisites

[List required tables (with schema names), tools, permissions, or configurations. If the skill requires a custom table that may not exist in every environment, include the full table schema here with column names, types, and choices — so a maker can create it.]

## Instructions

[Address the agent as "you" and the person interacting as "the user".]

### Step 1: [Action Verb + What This Step Does]

[Detailed instructions. For each step include:]
- What to ask the user or what data to collect
- The exact Dataverse operation: tool name (`list_records`, `create_record`, `update_record`, `read_query`), table, filter/select, or SQL query inline
- How to validate the result before proceeding
- What to do if validation fails or no records are found

[For steps that require SQL queries, embed them inline:]
```sql
SELECT column1, column2 FROM table WHERE condition = 'value'
```

[For steps with business logic, embed rules inline:]
- If [condition], then [action]
- If [other condition], then [alternative]

[Continue for each step...]

## Output Format

[Define the exact output structure the agent presents to the user. This is not optional — without it, agents produce inconsistent formatting. Use formatted blocks with clear headers, field labels, and section separators. If the skill has multiple output scenarios (e.g., success vs. error, summary vs. detail), define each one.]

```
RESULT HEADER
═══════════════════════
Field 1:    [value]
Field 2:    [value]
Status:     [value]
═══════════════════════

[If applicable, add secondary sections:]
DETAILS
───────────────────────
- Item 1: [value]
- Item 2: [value]

NEXT STEPS
───────────────────────
1. [Recommended action]
2. [Alternative action]
```

## Dataverse Tables Used

| Table | Purpose |
|-------|---------|
| `tablename` | What this table is used for in the skill |

## Key Fields Reference

[For each table the skill touches, list every field it reads or writes. Include the data type and — critically — the integer codes for all choice/option-set/state/status fields. This eliminates ambiguity when the agent constructs queries or creates records.]

**tablename:**
- `fieldname` (TYPE) — Description
- `statecode` (STATE) — Record state. Values: Active(0), Inactive(1)
- `statuscode` (STATUS) — Record status per state. Values: [label](code) per statecode
- `choicefield` (CHOICE) — Description. Values: Option1(0), Option2(1), Option3(2)

## Examples

### Example 1: [Happy-path scenario name]
**User says:** "[Realistic user input]"
**Actions:**
1. [What the agent does]
2. [Next action]
**Result:**
```
[Formatted output the user sees]
```

### Example 2: [Different scenario]
**User says:** "[Different input]"
**Actions:** [...]
**Result:** [...]

### Example 3: [Edge case or error scenario]
**User says:** "[Input that triggers edge case]"
**Result:** [How the agent handles it]

## Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| [symptom] | [root cause] | [what the agent should do] |

## Completion Checklist
- [ ] [Assertion the agent must verify before reporting success]
- [ ] [Another assertion]
```

**Quality standards for the instruction:**

1. **Specificity over vagueness** — Never write "process the data appropriately." Write the exact table, filter, columns, and tool. Every Dataverse read uses either `list_records` (simple CRUD) or `read_query` (SQL for aggregation, joins, complex filters). Every write uses `create_record` or `update_record`.
2. **SQL queries are inline** — Embed queries directly in the step where they're used, not in a separate section. Document parameters with `{placeholder}` syntax and expected results.
3. **No implicit knowledge** — Define all terms, acronyms, thresholds, and choice-set values inside the instruction. The agent should execute without prior domain understanding.
4. **Validation at every step** — Before moving between steps, include a check. "If zero records returned, inform the user..." / "If the field is empty, ask the user to provide..."
5. **Graceful error handling** — Every step that can fail has a fallback. Never let the agent get stuck. Pattern: detect → inform → offer alternatives.
6. **Concrete examples with realistic data** — At least 2 happy-path and 1 error/edge-case example. Use realistic company names (Contoso, Fabrikam, Alpine), dollar amounts, dates — never "XXX" or "sample".
7. **Output formatting** — Include an explicit Output Format section showing the exact structure the agent should present to the user, using formatted blocks.
8. **Table and field references** — Include "Dataverse Tables Used" and "Key Fields Reference" sections listing every table, field, type, and valid choice values the skill touches.
9. **Single responsibility** — One cohesive workflow per skill. If it branches into fundamentally different processes, split it.
10. **Dataverse SQL compliance** — All SQL queries must use only permitted operations.
11. **Choice/option-set values are explicit** — Never write "set status to Active." Write "set `statecode` = 0 (Active)." Always include the integer code alongside the label.
12. **Record matching handles ambiguity** — When searching for existing records, always handle: zero matches (inform user, offer alternatives), one match (proceed), multiple matches (present list, ask user to choose).

#### Dataverse SQL Reference

Use `read_query` for SQL when the process needs aggregation, multi-table joins, sorted/grouped results, or complex filtering. Use `list_records` for simple single-record lookups and `create_record`/`update_record` for writes.

**Permitted:** `SELECT` (explicit columns only, never `SELECT *`), `WHERE`, `ORDER BY`, `GROUP BY`, `JOIN`, `TOP`, `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`

**Prohibited and workarounds:**
| Prohibited | Workaround |
|-----------|------------|
| Nested subqueries | Break into sequential queries; use result of first as input to second |
| `HAVING` | Filter aggregated results in agent logic after query returns |
| `DISTINCT` | Use `GROUP BY` on the column(s) you want unique values for |
| `UNION` | Run separate queries, combine results in agent logic |
| `DATEADD`/`DATEDIFF`/`GETUTCDATE` | Calculate dates in agent logic, use literal values in `WHERE` |
| `CASE`/`WHEN` | Return raw values, apply conditional logic in agent steps |
| `CAST`/`CONVERT` | Ensure input values match column types before querying |

**SQL guidelines:**
1. Always alias aggregated columns (e.g., `COUNT(accountid) AS total_accounts`)
2. Use `{placeholderName}` for parameterized values
3. Use table-qualified column names in joins (e.g., `account.name`, not `name`)
4. Prefer multiple simple queries over one complex query
5. Document each query's purpose and expected result inline with the step

#### Common Dataverse field patterns

When building skills that touch standard Dataverse tables, use these known field patterns. Always verify actual schema names with the user, but these are the most common defaults:

**Standard fields present on most tables:**
- `statecode` (STATE) — Active(0), Inactive(1). Always filter `WHERE statecode = 0` for active records.
- `statuscode` (STATUS) — Varies by table and statecode. Always document the specific values.
- `createdon`, `modifiedon` (DATETIME) — System timestamps. Read-only.
- `ownerid` (LOOKUP → systemuser/team) — Record owner.

**Activity tables** (`phonecall`, `email`, `appointment`, `task`):
- All inherit from `activitypointer`. Common fields: `subject`, `description`, `regardingobjectid` (polymorphic lookup), `actualstart`, `actualend`, `statecode`, `statuscode`.
- Activity parties (`from`, `to`, `requiredattendees`) use the `activityparty` entity with `participationtypemask` values: Sender(1), ToRecipient(2), CCRecipient(3), Organizer(7), RequiredAttendee(5), OptionalAttendee(6).

**Relationship lookups:**
- `contact.parentcustomerid` → links contact to account
- `opportunity.customerid` / `opportunity.parentaccountid` → links opportunity to account
- `incident.customerid` → links case to account or contact

---

### Step 4 — Present the draft for review

Present the generated **name**, **description**, and **instruction** to the user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROPOSED SKILL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Name: <name>

Description:
<full description including trigger phrases>

Instructions:
<full instruction body>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Ask: **"Please review the skill definition above. You can: 1. Confirm — I'll create it now. 2. Edit — Tell me what to change. 3. Cancel — Discard this draft."**

For revisions: make requested changes preserving structure and quality, re-present only changed sections, and confirm again. Iterate until satisfied.

---

### Step 5 — Create the skill in Dataverse

Once confirmed, create the skill record.

#### 5a. Pre-creation validation

Verify `name`, `description`, and `skill_instruction` are populated and within character limits.

#### 5b. Create the record

```
Tool: create_record
Table: skill
Record data:
  name: <the generated name>
  description: <the generated description>
  skill_instruction: <the full instruction body>
```

#### 5c. Handle the response

**On success:** Capture the returned record ID. Proceed to Step 6.

**On failure:**

| Error | Action |
|-------|--------|
| `403 Forbidden` | Inform user to request write access to `skill` table from their Power Platform admin. Minimum: Basic User security role with create permissions on Business Skills entity. |
| `409 Conflict` / Duplicate | Query existing skill by name via `list_records`. Present to user and offer to update, rename, or deactivate the old one. |
| `400 Bad Request` | Review error, fix the problematic field, retry. If instruction too long, suggest splitting into focused skills. |
| `500 Server Error` | Retry once. If still failing, advise user to try again later. |
| Table `skill` not found | Advise enabling Dataverse intelligence in Power Platform admin center: Settings > Product > Features. Environment must be a Managed Environment with MCP server preview enabled. |

#### 5d. Handle update flow

If the user chose to update an existing skill:
```
Tool: update_record
Table: skill
Record ID: <existing record ID>
Record data:
  description: <revised description>
  skill_instruction: <revised instruction>
```
Warn before changing `name` — it may break solution references or other configurations.

---

### Step 6 — Confirm and advise next steps

After successful creation or update:

```
SKILL CREATED SUCCESSFULLY
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Name:        <name>
Record ID:   <record ID>
Status:      Active
Viewable by: Individual (default)
━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Test the skill:**
1. Connect to the Dataverse MCP server from any agent (Copilot Studio, VS Code, Claude Code, etc.).
2. Try a prompt matching your trigger phrases.
3. If the agent doesn't automatically invoke the skill, prefix with "Using business skills" — e.g., "Using business skills, <trigger phrase>".
4. Verify the agent follows each step and produces the expected output.
5. If the behavior isn't right, describe what needs to change and I'll update the skill.

**Share the skill:**
- **Organization-wide:** In the Skill Management app (Power Apps), change "Viewable by" to "Organization".
- **Specific users/teams:** Use the Share command in the Skill Management app.

**Deploy across environments:** Go to Power Apps > Solutions > Add existing > Business skill.

**Deactivate if needed:** In the Skill Management app, select the skill and choose "Deactivate".

Finally ask: **"Would you like to create another skill?"**

---

## Examples

### Example 1 — Activity-logging skill

**User:** "I want a skill that logs sales call transcripts into Dataverse."

**Agent gathers:** Accepts transcript text, extracts participants, company, call metadata, discussion points, and action items. Matches to existing accounts/contacts/opportunities. Creates `phonecall` activity. Creates follow-up `task` if action items found. Attaches transcript as `annotation`.

**Agent produces:**
- **Name:** `log-call-transcripts`
- **Description:** Analyzes sales call transcripts to extract key information and automatically logs the call as a Phone Call activity in Dataverse. Use when user says "log this call", "process this transcript", "save this call to CRM", "create a phone call record from this transcript", uploads a call transcript, or pastes meeting notes to be logged.
- **Instruction includes (demonstrating production-quality patterns):**
  - **Steps with inline queries:** Step 3 embeds `SELECT accountid, name FROM account WHERE name LIKE '%{companyName}%' AND statecode = 0` directly, with "If zero records returned, ask the user to confirm the company name or offer to create a new Lead."
  - **Explicit tool calls:** Step 5 specifies `create_record` on `phonecall` with every field: `subject` (max 200 chars), `description`, `phonenumber`, `directioncode` (true=Outgoing, false=Incoming), `actualdurationminutes`, `regardingobjectid`, `statecode`: 1 (Completed), `statuscode`: 2 (Made) or 4 (Received).
  - **Conditional logic inline:** "If action items were identified, create a `task` record with `subject` = first action item, `scheduledend` = mentioned timeline or 3 business days, `prioritycode`: 1(High), 2(Normal), 3(Low)."
  - **Tables Used section:** `phonecall`, `account`, `contact`, `opportunity`, `lead`, `task`, `annotation` — each with its purpose.
  - **Key Fields Reference:** Lists `phonecall` fields with types (`directioncode` BOOLEAN, `statecode` STATE: Open(0), Completed(1), Canceled(2)) and `activityparty` fields with `participationtypemask` values.

### Example 2 — Policy-enforcement skill with custom table

**User:** "We need a skill for processing expense receipts with approval thresholds and duplicate detection."

**Agent gathers:** Extracts receipt data, categorizes expenses, applies approval tiers ($500/Manager, $2K/Controller, $5K+/CFO), detects duplicates, enforces 90-day policy, tracks payment method for reimbursement.

**Agent produces:**
- **Name:** `expense-entry`
- **Description:** Processes expense receipts and creates expense report entries following company policies with approval thresholds and validation rules. Use when user says "log this expense", "process this receipt", "create expense entry", "submit expense", "add to expense report", uploads a receipt, or provides purchase documentation.
- **Instruction includes (demonstrating production-quality patterns):**
  - **Custom table schema in Prerequisites:** Full table definition with display names, logical names (`cr###_reportid`, `cr###_totalamount`, etc.), column types (Text, Currency, Choice, Lookup), and choice values (Report Status: Draft, Submitted, Pending Approval, Approved, Rejected, Paid). Includes note: "Replace `cr###_` with your publisher prefix."
  - **Business rules inline in steps:** Step 3 defines approval tiers directly: "If total < $500: `approvaltier` = None (auto-approved). If $500-$2,000: `approvaltier` = Manager. If $2,000-$5,000: `approvaltier` = Controller. If > $5,000: `approvaltier` = CFO (requires business case)."
  - **Duplicate detection inline:** "Search existing entries where merchant name matches AND transaction date matches AND amount is within $1 tolerance AND same employee. If found: set `duplicatestatus` = 'Duplicates Found' and DO NOT CREATE. Return: 'Duplicate detected: [ER-ID] on [date] for $[amount] at [merchant].'"
  - **Validation rules inline:** "Meals > $75/person require attendee list. Transaction date not in future. If `dayssinceexpense` > 90, set `latesubmission` = Yes."
  - **Output Format:** Shows formatted confirmation block with Report ID, Status, Amount, Category, Payment Method, and approval/reimbursement status.

### Example 3 — Analytics skill with SQL aggregation

**User:** "I need a skill that gives sales managers a pipeline health summary with deal scoring."

**Agent gathers:** `opportunity` table with stage, value, close date. Joins to `account` for industry. Aggregates by stage. Scores deals by activity recency, stakeholder engagement, BANT completion. Compares against won deal patterns.

**Agent produces:**
- **Name:** `pipeline-health-analyzer`
- **Description:** Analyzes pipeline health by scoring opportunities on velocity, engagement, and qualification completeness, then surfaces at-risk deals with recommended actions. Use when user says "show me pipeline health", "which deals are at risk", "pipeline analysis", "deal health check", "forecast risks", "stalled deals".
- **Instruction includes (demonstrating production-quality patterns):**
  - **Date workaround:** Step 1 says "Calculate the current calendar quarter start and end dates from today's date. Store as `{quarterStart}` and `{quarterEnd}` in YYYY-MM-DD format. This is required because Dataverse SQL does not support GETUTCDATE() or DATEADD()."
  - **Inline SQL with context:** Step 2 embeds: `SELECT salesstagecode, COUNT(opportunityid) AS deal_count, SUM(estimatedvalue) AS total_value FROM opportunity WHERE statecode = 0 GROUP BY salesstagecode ORDER BY total_value DESC` — followed by "Expected result: one row per stage with count and total value."
  - **Weighted scoring model:** Step 4 defines composite scoring: "Activity recency (30%): 100 if activity within 7 days, 50 if 8-14 days, 25 if 15-30 days, 0 if 30+ days. Stakeholder depth (25%): 100 if economic buyer engaged, 50 if only influencers. BANT completion (25%): 25 points per confirmed element. Stage velocity (20%): 100 if below average days-in-stage, 0 if 2x average."
  - **Key Fields Reference with choice values:** `opportunity.salesstage`: Qualify(0), Develop(1), Propose(2), Close(3). `opportunity.budgetstatus`: No Budget(0), May Buy(1), Can Buy(2), Will Buy(3). `opportunity.statecode`: Open(0), Won(1), Lost(2). `opportunity.statuscode` per statecode: In Progress(1), On Hold(2) [Open]; Won(3) [Won]; Canceled(4), Out-Sold(5) [Lost].
  - **Output Format:** Shows formatted pipeline summary with stage table, top 5 at-risk deals with health scores, and per-deal recommended next actions.

---

## Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| Agent cannot find `skill` table | Dataverse intelligence not enabled | Enable in Power Platform admin center: Settings > Product > Features. Must be a Managed Environment. |
| `create_record` returns 403 | User lacks write access | Contact admin to grant write privileges on `skill` table (Basic User role minimum). |
| `create_record` returns 409 | Duplicate skill name | Query existing skill, offer to update, rename, or deactivate. |
| Agents don't discover the skill | "Viewable by" defaults to individual | Change to "Organization" in Skill Management app, or share with specific users/teams. |
| Skill triggers for unrelated prompts | Trigger phrases too generic or overlap with other skills | Revise with more specific, unique phrases. Remove single-word triggers. Check other active skills for overlap. |
| Instruction too long or truncated | Scope too broad or includes unnecessary text | Split into focused skills. Ensure every sentence is actionable by the agent. Remove redundant explanations. |
| Agent follows skill but produces wrong results | Ambiguous instructions or incorrect schema references | Verify all table/column names match actual Dataverse schema. Add explicit validation checks. Test with real data. |
| SQL query returns unsupported syntax error | Uses prohibited Dataverse SQL operation | Rewrite using only permitted operations. Break subqueries into sequential queries. Calculate dates in agent logic. |
| Skill works in one environment but not another | Table schema differs or Dataverse intelligence not enabled in target | Verify target has same tables/columns. Ensure intelligence and MCP preview are enabled. Check solution import completed without errors. |

---

## Completion Checklist

Before reporting success, verify:

- [ ] **Name** is kebab-case, 2-4 words, unique in the environment
- [ ] **Description** has a summary sentence and at least 6 diverse trigger phrases covering commands, questions, and contextual descriptions
- [ ] **Instruction** has overview paragraph explaining the business problem
- [ ] **Instruction** has prerequisites listing all tables, tools, and permissions (with custom table schemas if needed)
- [ ] **Instruction** has numbered steps with explicit Dataverse operations (tool + table + filter/columns or inline SQL)
- [ ] **Instruction** has validation checks and error handling at every step
- [ ] **Instruction** has an Output Format section with formatted result blocks
- [ ] **Instruction** has Dataverse Tables Used and Key Fields Reference sections
- [ ] **Instruction** has at least 2 happy-path and 1 error/edge-case example with realistic data
- [ ] **Instruction** has a troubleshooting table and completion checklist
- [ ] **Instruction** follows single responsibility — one process, one outcome
- [ ] **SQL queries** (if any) use only permitted Dataverse SQL operations with explicit column names, no prohibited constructs
- [ ] **Choice/option-set fields** include integer codes alongside labels (e.g., Active(0), not just "Active")
- [ ] **Record matching steps** handle zero, one, and multiple matches explicitly
- [ ] **Trigger phrases** do not overlap with other active skills in the environment
- [ ] User has reviewed the full draft and explicitly confirmed
- [ ] `create_record` on `skill` table succeeded and returned a record ID
- [ ] User has been advised on testing, sharing, deployment, and deactivation

---
> Source: [Wariowaqo/cs-skill-creator](https://github.com/Wariowaqo/cs-skill-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
