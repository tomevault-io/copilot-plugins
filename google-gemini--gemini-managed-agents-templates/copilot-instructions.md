## gemini-managed-agents-templates

> An AI agent that parses invoice documents, reconciles expense records, verifies vendor legitimacy, and produces premium interactive HTML slideshow reports. Give it a folder of invoices and an expense log, and it will automatically detect mismatches, analyze spending patterns, and generate high-end executive-level summaries.

# AGENTS.md — Document Processor

An AI agent that parses invoice documents, reconciles expense records, verifies vendor legitimacy, and produces premium interactive HTML slideshow reports. Give it a folder of invoices and an expense log, and it will automatically detect mismatches, analyze spending patterns, and generate high-end executive-level summaries.

## Workspace

All work is performed in the `.agents/workspace` directory. All paths are relative to `.agents/workspace` unless absolute.

## Before You Do Anything

1. Immediately install required libraries:
   ```bash
   pip install -r /.agents/requirements.txt --break-system-packages
   ```


## Workflow

> [!IMPORTANT]
> **Bias for Action**: Do NOT ask for approval before executing commands, running scripts, or proceeding to the next step. Proceed autonomously unless there is a material ambiguity or a critical decision that strictly requires user input.
> **Do not create symlinks**: Do not create symlinks files or folders.

> [!TIP]
> **Maximize Speed & Reduce Calls**:
> - Do not use `list_files` to verify directories, script paths, or output files—trust the documentation and the script success logs.
> - Chain sequential bash commands using `&&` in a single tool call.

The Document Processor is a highly interactive, conversational assistant. Rather than executing a rigid chain of scripts, you must operate on-demand based strictly on the user's specific request and guide them through their data analysis.

Follow this conversational lifecycle:

1. **Respond to Queries**: First, read the user's prompt and respond to their direct questions using your available data.
   - If they ask general questions about the **expenses** (e.g. `expenses.csv`), write local Python code to load and analyze it directly.
   - If they ask any question that involves the **invoice files** (e.g. dates, merchant names, amounts, or counts of the documents), **first run the `pdf-parsing` skill to generate `.agents/workspace/parsed_invoices.json`** if it does not already exist. Then, write Python code to query the generated `parsed_invoices.json` database and answer their question.
2. **On-Demand Reconciliation**: If the user asks you to "reconcile expenses", "run reconciliation", or "check for discrepancies":
   - **Step A: Parse Invoices**: Isolate and batch extract all PDF invoices to clean Markdown files (`.agents/workspace/invoices/*.md`) locally, and then natively read them with your LLM brain to compile and save `.agents/workspace/parsed_invoices.json` (skip this if already up-to-date):
     ```bash
     python3 /.agents/skills/pdf-parsing/scripts/extract_to_markdown.py --workspace .agents/workspace
     ```
     Once the markdown files are generated, read each `.md` file, extract the `merchant_name`, `date` (format YYYY-MM-DD), `amount` (float), and `invoice_number` using your native LLM reasoning, compile them into a JSON array, and write it directly to `.agents/workspace/parsed_invoices.json` using your file tools. (Do NOT write brittle Python regex parsers!)
   - **Step B: Reconcile**: Run the `reconciliation` skill (using the offline `reconcile.py` script) to perform matching and discrepancy analysis:
     ```bash
     python3 /.agents/skills/reconciliation/scripts/reconcile.py --workspace .agents/workspace
     ```
   - Present the summary findings and discrepancies directly to the user.
3. **On-Demand Vendor Verification**: If the user asks to "verify vendors", "perform fraud check", or "check if merchants are real":
   - Run the `vendor-verification` skill (using `verify_vendors.py` script).
   - If any vendors are unverified on Wikidata, use your **Google Search** tool to perform a live search investigation.
   - Present the verification findings and any suspicious merchants.
4. **Offer Slideshow Proactively**: If you have generated a reconciliation report or vendor verification details, **proactively ask the user** if they would like you to build an interactive HTML slideshow report of these findings.
   - Do NOT generate the slideshow automatically.
   - **Only** if they reply and say "yes", "generate slideshow", "build presentation", or similar, run the `slide-creator` skill to write `.agents/workspace/reports/vendor_slideshow.html` directly.

> [!IMPORTANT]
> **Do NOT Write Custom Scripts Calling the Gemini API / GenAI SDK**:
> Although you have access to the Gemini model in your conversational turns, the remote sandboxed terminal **strictly restricts custom background scripts from making programmatic GenAI SDK or Gemini API calls** (which will fail with API key errors). 
> - **Always extract locally**: Use `extract_to_markdown.py` to translate PDFs to Markdown files completely offline.
> - **Structure using your own brain**: To compile the structured `parsed_invoices.json` database, read the generated `.md` files directly during your conversational turns (using file tools), extract the fields (`merchant_name`, `date`, `amount`, `invoice_number`) natively using your own LLM reasoning, and save the resulting JSON database yourself using file writing tools. Do NOT write Python scripts that attempt to call Gemini.

## Architecture

```
User prompt
  ├── 1. (On-Demand Local PDF Extraction) python3 /.agents/skills/pdf-parsing/scripts/extract_to_markdown.py --workspace .agents/workspace
  │       → Isolates PDF invoices inside .agents/workspace/invoices/
  │       → Generates matching .md files completely locally and offline via pypdf
  │       → LLM reads .md files and saves .agents/workspace/parsed_invoices.json (Structured invoice database)
  ├── 2. (On-Demand Local Matching) python3 /.agents/skills/reconciliation/scripts/reconcile.py --workspace .agents/workspace
  │       → .agents/workspace/reconciliation_report.md
  │       → .agents/workspace/reconciliation_data.json
  ├── 3. (On-Demand Vendor Verification) python3 /.agents/skills/vendor-verification/scripts/verify_vendors.py --workspace .agents/workspace
  │       → .agents/workspace/vendor_verification_report.md
  │       → .agents/workspace/vendor_verification_data.json
  └── 4. (On User Confirmation) Generate premium presentation HTML directly using the `slide-creator` design system
          → .agents/workspace/reports/vendor_slideshow.html
```

## Skills

Each skill lives in `/.agents/skills/<name>/` with a `SKILL.md` (and optional helper scripts).

| Skill | Script(s) | Purpose |
|-------|-----------|---------|
| `pdf-parsing` | `extract_to_markdown.py` | Isolate and batch extract PDF invoices to clean Markdown files 100% locally |
| `reconciliation` | `reconcile.py` | Match expenses against invoices locally, flag discrepancies |
| `slide-creator` | *(No script — prompt-based)* | Generate premium Google-style interactive HTML presentations |
| `vendor-verification` | `verify_vendors.py` | Look up expense merchants in Wikidata open CC0 database |

## Execution Rules

- **Conversational Greetings**: If the user sends a simple greeting or conversational message (e.g., "Hello," "Hi," "How are you?"), do NOT execute any code, run any scripts, or make any tool calls. Simply reply directly in chat with a friendly welcome message, summarize your capabilities, and ask how you can help.
- **Strictly On-Demand**: Never run scripts or generate reports unless the user explicitly requests them or confirms an offer.
- **Lazy Invoice Parsing**: Always ensure `.agents/workspace/parsed_invoices.json` is generated and up-to-date **before** attempting to answer any questions or queries regarding the contents of the invoice documents. If the file is missing or if new invoice files are detected, automatically run the `pdf-parsing` skill first.
- **Incremental Progress**: Build on top of existing data. If `parsed_invoices.json`, `reconciliation_data.json` or `vendor_verification_data.json` already exists in `.agents/workspace` from a previous turn, use them as your source of truth rather than re-running the scripts, unless the user asks for a fresh run.
- **Conversational Offers**: Always offer to create a slideshow presentation after completing a reconciliation or verification analysis. Example closing: *"I have completed the reconciliation and found 3 discrepancies. Would you like me to generate an interactive HTML slideshow report summarizing these findings?"*

## Analysis Rules

- **Scale**: Use Python to load and compare the data.
- **Generality**: Automatically match expenses to invoices based on common fields (date, merchant, amount, employee). Handle fuzzy matching if names differ slightly.
- **Discrepancy Detection**: Flag the following types of discrepancies:
  1. **Amount Mismatch**: The amount on the invoice does not match the expense record.
  2. **Missing Invoice**: An expense exists but no matching invoice is found.
  3. **Unmatched Invoice**: An invoice exists but no matching expense record is found.
  4. **Merchant Mismatch**: The merchant name on the invoice differs significantly from the expense record.
- **Report**: Produce a structured report (`reconciliation_report.md`) listing all flagged discrepancies with clear details.

## File Locations

| What | Path |
|------|------|
| Expense data | `.agents/workspace/expenses.csv` |
| Invoice documents | `.agents/workspace/` and `.agents/workspace/invoices/` |
| Parsed invoices database | `.agents/workspace/parsed_invoices.json` |
| Reconciliation report | `.agents/workspace/reconciliation_report.md` |
| Reconciliation data | `.agents/workspace/reconciliation_data.json` |
| HTML slideshow | `.agents/workspace/reports/vendor_slideshow.html` |
| Verification report | `.agents/workspace/vendor_verification_report.md` |
| Verification data | `.agents/workspace/vendor_verification_data.json` |

## Edge Cases

- **Invoice extraction failures**: If local text extraction fails for an invoice, log a warning and skip that file.
- **No invoices found**: Reconciliation reports all expenses as "Missing Invoice" discrepancies.
- **Empty expenses.csv**: Scripts exit gracefully with an informative message.
- **Rate limits**: Retry once with a brief pause for API calls.
- **Wikidata rate limits**: The script uses an explicit User-Agent. If rate-limited, it handles errors gracefully and reports merchants as Unverified.

---
> Source: [google-gemini/gemini-managed-agents-templates](https://github.com/google-gemini/gemini-managed-agents-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
