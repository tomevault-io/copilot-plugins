## standard-prd-skill

> Generate detailed, developer-guiding standard PRDs from requirement docs, product plans, prototypes, HTML mockups, and design files. Use when a PRD must be specific enough to guide implementation, especially for page-level or Tab-level products with CRUD flows, validations, state rules, visible fields, hidden fields, export/send behavior, and linked refresh logic. Use when existing PRDs are too vague, too summary-like, or too technical.


# Standard PRD

## Purpose

Turn scattered product material into business-first PRDs that are detailed enough to guide implementation without becoming architecture specs.

## Core rules

### 1. Prefer page-level splitting

When the source clearly contains pages, Tabs, screens, dialogs, or drawers:

- generate one overview PRD
- then split PRDs by page or Tab
- keep modal/drawer PRDs separate when they represent distinct business actions

Do not flatten a page-based product into broad capability buckets if the UI already defines page boundaries.

### 2. Business detail beats summary

Never stop at:

- “supports customer profile”
- “supports create/update/delete/query”
- “supports quote management”

For every page, describe:

- why the page exists
- who uses it
- what the user sees
- what the user can do
- what changes after each action
- what blocks, warns, confirms, or refreshes

If the document still reads like a feature list, it is incomplete.

### 3. CRUD must be explicit

If a page has CRUD-like behavior, explicitly define:

- query trigger rules
- default query behavior
- filter fields and match modes
- sorting rules
- visible table/list fields
- operation column behavior
- detail-view reflection scope
- create defaults
- editable vs read-only fields
- delete scope and confirmation
- save success refresh scope
- failure handling

### 4. Fields must be concrete

For business pages, separate:

- `visible fields`
- `hidden system fields`

For each important field, specify when possible:

- meaning
- whether required
- input type
- format
- length
- enum values
- readonly conditions
- default values
- source of truth
- units

### 5. Generated artifacts must be defined

If the page includes:

- preview
- export
- send
- quote generation
- card generation

define:

- artifact type
- artifact format
- naming rule
- eligibility rules
- whether the action changes business status

### 6. Preserve source precision

If the source gives any of the following, preserve them explicitly:

- exact prompt copy
- current logic vs new logic
- correct examples vs wrong examples
- delimiter rules
- order rules
- state-specific button availability
- imported/exported column order
- close-window restrictions

Do not compress them into generic prose.

## Workflow

### Step 1: Read all source material

Read all relevant docs, prototypes, and design files before drafting.

### Step 2: Treat design as page truth

If a design file exists, use it to identify:

- page names
- section structure
- major visible fields
- action areas

Include screenshots when they materially help anchor the page.

### Step 3: Build a requirement inventory

Extract:

- background and objective
- page structure
- CRUD behavior
- field rules
- visible fields
- hidden fields
- prompts
- state rules
- linkage and refresh
- export/send behavior
- logs and audit
- edge cases

### Step 4: Write the PRD

Default page PRD structure:

1. Document info
2. Page role
3. Users and entry scenarios
4. Page objective
5. Screenshot / prototype reference
6. Page structure and sections
7. Detailed functional behavior
8. CRUD business logic
9. Field and state rules
10. Linked refresh and downstream impact
11. Edge and failure cases
12. Acceptance criteria
13. Risks and pending confirmations

## Quality bar

The result is acceptable only if:

- product can review it without rereading source docs
- frontend can understand page structure and visible behavior
- backend can understand the business effects of actions
- QA can derive tests directly from the document

If key business constraints are missing, use:

`建议默认规则（待确认）`

Do not silently guess.

## References

- `./references/page-led-prd-patterns.md`
- `./references/crud-detail-checklist.md`
- `./references/prd-template.md`

---
> Source: [KaelZhang321/standard-prd-skill](https://github.com/KaelZhang321/standard-prd-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
