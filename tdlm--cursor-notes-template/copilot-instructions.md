## cursor-notes-template

> All meeting notes files in `02-Meetings/` must follow this format:

# Notes Organization Rules

## Meeting Notes Naming Convention

All meeting notes files in `02-Meetings/` must follow this format:

**Format:** `[YYYY]-[MM]-[DD]-[title].md`

**Examples:**
- `2025-01-15-team-standup.md`
- `2025-02-10-client-discussion.md`
- `2025-03-20-coaching-session.md`

**Rules:**
- Date format: YYYY-MM-DD (ISO 8601)
- Date comes first for chronological sorting
- Title should be lowercase with hyphens separating words
- Title should be descriptive but concise
- No spaces in filenames (use hyphens instead)

## File Organization

- **00-Inbox/**: Single inbox for all files awaiting processing
- **01-General/**: General notes and references
- **02-Meetings/**: **ALL meeting notes** (follow naming convention above)
  - Organized by year and month: `02-Meetings/[YYYY]/[MM]/`
  - All business meetings, personal meetings, discussions, and coaching sessions
  - Business-related meeting notes go here (NOT in `08-Business/`)
- **03-Emails/**: Important personal and business emails
  - Organized by year and month: `03-Emails/[YYYY]/[MM]/`
- **04-Documents/**: PDFs, contracts, receipts, statements converted to Markdown
  - Organized by year and month: `04-Documents/[YYYY]/[MM]/`
- **05-People/**: Notes about people in your life (contacts, relationship notes, etc.)
  - Reference materials about people
  - Meeting notes about people should go to `02-Meetings/`
- **06-Projects/**: Project-specific documentation
  - Side projects and personal projects
- **07-Ideas/**: Ideas, brainstorms, and concepts (see Idea Processing section below)
- **08-Business/**: Business-related **information and documentation only** (NOT meeting notes)
  - Reference materials, business plans, ongoing documentation
  - Meeting notes about business should go to `02-Meetings/`
- **09-Personal/**: Personal reference documents only (NOT meeting notes)
  - Career planning, learning plans, etc. (non-meeting reference materials)
  - Meeting notes go to `02-Meetings/`
  - **recipes/**: Recipe collection (timeless reference materials, no date-based folders)

## Inbox Processing

**IMPORTANT:** When files are present in `00-Inbox/`, automatically process them without requiring explicit user request.

### 00-Inbox Processing

When processing files in `00-Inbox/`:
- **Automatically process all files** (except `README.md` which stays in inbox)
- **PDF/DOCX files**: Convert to Markdown first using conversion scripts:
  - PDF files → `node scripts/convert_pdf.js <file> --output-dir 00-Inbox/`
  - DOCX files → `node scripts/convert_docx.js <file> --output-dir 00-Inbox/`
  - Original files are deleted after successful conversion
- **PDF/DOCX files in attachment folders**: Convert to Markdown using conversion scripts:
  - PDF files → `node scripts/convert_pdf.js <file> --output-dir <attachment-folder>/`
  - DOCX files → `node scripts/convert_docx.js <file> --output-dir <attachment-folder>/`
  - Original files are deleted after successful conversion
  - **IMPORTANT**: All PDF and DOCX files in attachment folders must be converted to Markdown. Only Markdown files should remain in attachment folders after processing.
- **Markdown files**: Format for better readability:
  - Normalize spacing (consistent line breaks between sections)
  - Ensure proper heading hierarchy
  - Clean up excessive whitespace
  - Fix common markdown formatting issues
  - Improve list formatting consistency
  - Add appropriate spacing around code blocks, tables, etc.
- **Content detection**: Analyze files to determine appropriate destination based on keywords and content:
  - Check for keywords in filenames and content: "meeting", "email", "document", "project", "idea", "business", "personal", "recipe", etc.
  - Route files based on detected content type:
    - Meeting notes (dated discussions, syncs, sessions, keywords: "meeting", "call", "discussion", "session", "sync") → `02-Meetings/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md`
    - Email content (keywords: "email", "message", "correspondence") → `03-Emails/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md`
    - Documents (keywords: "document", "contract", "agreement", "letter", "statement", "receipt", "invoice") → `04-Documents/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md`
    - Project documentation (keywords: "project", "task", "milestone") → `06-Projects/` (or date-based `06-Projects/[YYYY]/[MM]/` if applicable)
    - Ideas and concepts (keywords: "idea", "concept", "brainstorm", "what if", "app idea", "game idea", "business idea") → **Trigger Idea Flow** (see Idea Processing section)
    - Business reference materials (keywords: "business", "company", "corporate", "strategy") → `08-Business/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md` (NOT meeting notes)
    - Personal reference materials (keywords: "personal", "career", "resume", "cv") → `09-Personal/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md` (NOT meeting notes)
    - Recipes (keywords: "recipe", "cooking", "dish", "meal", "ingredients") → `09-Personal/recipes/` with descriptive filename (no date prefix, e.g., `chicken-tikka-masala.md`)
    - People-related notes (keywords: "person", "contact", "relationship") → `05-People/` (reference materials only, NOT meeting notes)
  - **If destination is unclear**: Prompt the user to clarify where the file should be placed
- **Date extraction**: Extract dates from filenames, content, file metadata, or email headers
- **File organization**: 
  - Format dates as `YYYY-MM-DD` for meeting notes, emails, documents, business docs, and personal docs
  - Organize into year/month folders: `[YYYY]/[MM]/` for all date-based destinations
  - Rename files to follow naming conventions: `[YYYY]-[MM]-[DD]-[title].md` before moving
  - For emails with attachments, move attachments (in folders) with the email to maintain relationship
- **Delete original files from inbox** after successful processing
- Only `README.md` should remain in the inbox folder

**Date-Based Structure:**
All sub-folders use `[YYYY]/[MM]/` structure when sorting files:
- `02-Meetings/[YYYY]/[MM]/` ✓
- `03-Emails/[YYYY]/[MM]/` ✓
- `04-Documents/[YYYY]/[MM]/` ✓
- `08-Business/[YYYY]/[MM]/` ✓
- `09-Personal/[YYYY]/[MM]/` ✓

**Conversion Script Requirements:**
- Scripts are located in `scripts/` directory
- Dependencies must be installed: `cd scripts && npm install`
- Scripts use `pdf-parse` for PDF conversion and `mammoth` for DOCX conversion

## Idea Processing

**CRITICAL:** When an idea is detected in `00-Inbox/` OR when the user describes an idea conversationally, trigger the Idea Flow.

### Trigger Detection

Detect ideas via:
- **Inbox files**: Keywords in filename or content: "idea", "concept", "what if", "business idea", "app idea", "game idea", "brainstorm"
- **User prompts**: Phrases like "I have an idea", "what if we built", "I'm thinking about", "here's a concept", or any description of a potential product/project/improvement

### Idea Categories

| Type | Description | Validation Level |
|------|-------------|------------------|
| `business` | Revenue-generating ideas | Full validation framework |
| `project` | Side projects, tools, apps | Scope + MVP definition |
| `game` | Game concepts | Core loop + prototype |
| `home` | Home improvement, organization | Light - notes + next step |
| `misc` | Everything else | Minimal - capture + next step |

### Universal Rule

**Every idea MUST have a `next_step`.** An idea without a next step is just noise. No exceptions.

### Idea File Template

All idea files go to `07-Ideas/` with this frontmatter structure:

```yaml
---
title: Idea Name
idea_type: business | project | game | home | misc
status: raw | validating | validated | active | paused | killed
created: YYYY-MM-DD
updated: YYYY-MM-DD
next_step: "The one concrete thing to do next"
deadline: YYYY-MM-DD  # Required for business ideas (fail-fast deadline)
---
```

**Status definitions:**
- `raw` - Just captured, not yet processed
- `validating` - Actively running validation experiments
- `validated` - Passed validation, ready to build
- `active` - Currently being worked on
- `paused` - On hold, but not dead
- `killed` - Failed validation or abandoned

### Idea Flow Process

1. **Detect idea** (inbox or prompt)
2. **Ask idea type** if not obvious from context
3. **Ask type-specific questions** (see below)
4. **Create structured file** with frontmatter
5. **Confirm next step** - do not finalize without one

### Business Idea Validation Framework

For `business` ideas, ask these questions and populate sections:

**Required Questions:**
1. What's the one-sentence hypothesis? (Format: "My target customer [SPECIFIC PERSON] struggles with [PAINFUL PROBLEM] and would pay for [SOLUTION]")
2. Who specifically is the target customer? (Be specific - not "small businesses" but "solo consultants making $100-200k who struggle with invoicing")
3. What's the cheapest validation experiment? (Landing page, concierge service, or social smoke test)
4. What would silence or low engagement mean?
5. What's the fail-fast deadline? (When do we evaluate and decide to pivot or kill?)

**File Sections for Business Ideas:**
```markdown
## Hypothesis
[One-sentence hypothesis]

## Target Customer
[Specific description]

## Validation Experiment
[The cheapest way to test - pick ONE]:
- Landing page test (Carrd/Notion + email signup)
- Concierge service (deliver manually before building)
- Social smoke test (post about the problem, measure response)

## Success Signals
- [What would indicate this idea has legs]

## Failure Signals
- [What silence or lack of interest means]
- [Red flags that would kill this idea]

## Fail-Fast Deadline
[Date] - On this date, evaluate results and decide: continue, pivot, or kill.

## Next Step
[Single, concrete action]
```

**Anti-Elaboration Rule:** For unvalidated business ideas:
- Do NOT discuss elaborate features until hypothesis is tested
- Do NOT build anything until there's signal
- If user starts building castles, redirect to validation experiments
- Keep it simple. Keep it cheap. Test fast.

### Project Idea Questions

For `project` ideas:
1. What problem does this solve for you?
2. What's the absolute minimum viable version?
3. What's the next step?

**File Sections:**
```markdown
## Problem
[What this solves]

## MVP Scope
[Absolute minimum to be useful]

## Next Step
[Single, concrete action]
```

### Game Idea Questions

For `game` ideas:
1. What's the core loop? (The repeating action that is the game)
2. Why would this be fun?
3. What's the smallest playable prototype?

**File Sections:**
```markdown
## Core Loop
[The repeating action]

## Why It's Fun
[The hook]

## Smallest Prototype
[What to build first]

## Next Step
[Single, concrete action]
```

### Home / Misc Idea Questions

For `home` and `misc` ideas, keep it simple:
1. Any notes to capture?
2. What's the next step?

**File Sections:**
```markdown
## Notes
[Any relevant details]

## Next Step
[Single, concrete action]
```

### Validation Experiment Types

Reference for business idea validation:

**Landing Page Test:**
- Build a one-page site (Carrd, Notion, or simple HTML)
- Focus on the PROBLEM and OUTCOME, not features
- Add "Get Early Access" button that collects emails
- Success signal: 100+ email signups
- Timeline: 3-7 days to set up and run

**Concierge Service:**
- Sell the solution, deliver it manually
- Use spreadsheets, Zapier, manual processes
- Customers don't need to know it's not automated
- Success signal: Paying customers who stay
- Timeline: 1-2 weeks to get first customers

**Social Smoke Test:**
- Post about the PROBLEM on LinkedIn, Twitter, Reddit
- Don't pitch - just discuss the pain
- "Anyone else hate how long it takes to [X]?"
- Success signal: Strong engagement, comments agreeing
- Timeline: 1-3 days for initial signal

### Idea Lifecycle

```
raw → validating → validated → active → completed
                ↘ killed
                ↘ paused
```

- Move from `raw` to `validating` when starting experiments
- Move to `validated` only with positive signals
- Move to `killed` if fail-fast deadline passes without signal
- `paused` is for ideas with potential but not priority

---
> Source: [tdlm/cursor-notes-template](https://github.com/tdlm/cursor-notes-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
