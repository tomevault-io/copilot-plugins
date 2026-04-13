## ai-workshop

> Writing standards for code analysis documents — structure, metadata table, API descriptions, and pseudocode conventions


# Code Analysis Document Standards

When writing a code-analysis or codebase-walkthrough document, follow the structure below.

## 1. Metadata Table (top of document)

Place a table at the **very beginning** of the document with the following rows:

| Item | Value |
|------|-------|
| Author | `<tool name>` / `<model name>` |
| Created | `<YYYY-MM-DD HH:mm>` (use the current time) |
| Repository | HTTPS URL from `git remote get-url origin` |
| Commit | Short hash + subject, e.g. `a1b2c3d feat: add login` |

- The URL must be **browser-navigable** (clickable and opens the repo homepage):
  - SSH format `git@github.com:org/repo.git` → `https://github.com/org/repo`
  - Strip trailing `.git` suffix if present.
  - For other hosting platforms (GitLab, Gitee, Codeup, etc.), apply the same conversion logic.
- If multiple remotes exist, prefer **origin**.
- Run `git` commands at generation time to fill in Repository and Commit automatically.

## 2. Feature / Module Description Order

For each feature or module, present content in this fixed order:

### 2.1 Overall Logic Flow

- Describe the end-to-end flow in plain language first.
- When the flow has branches, loops, or async steps, add a **Mermaid flowchart** or **pseudocode block** to clarify.

### 2.2 Usage & Trigger Conditions

- Explain how the feature is invoked (UI entry, CLI command, scheduled task, event trigger, etc.).
- **If the feature exposes an API**, document it in open-API style:

| Field | Content |
|-------|---------|
| Endpoint | `METHOD /path` |
| Request params | Table of param / type / required / description |
| Response structure | Table or JSON schema |
| Key logic | Short text describing what the endpoint does internally |
| Request example | Curl or HTTP snippet |
| Response example | JSON snippet |

### 2.3 Implementation Details

- Prefer **pseudocode + source file path** over pasting raw code.
  - Good: "The scheduler reads tasks from the queue and dispatches them (see `worker/dispatcher.py`)."
  - Bad: pasting 50+ lines verbatim.
- When referencing source code, use the format `` `startLine:endLine:filepath` `` and keep snippets ≤ 30 lines.
- Highlight non-obvious design decisions, trade-offs, or known limitations.

## 3. References (end of document)

Close the document with a **References** section listing:

- Official docs, RFCs, or design docs consulted.
- Related internal documents or ADRs.
- Links to relevant source files or directories.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whhe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/whhe)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
