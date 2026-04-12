## allearning-purchase-request-managment

> This repository is a **professional learning and implementation workspace** for the **AL programming language** — the language used to extend and customize **Microsoft Dynamics 365 Business Central** (BC).

# CLAUDE.md — AL Language Learning & Implementation Workspace

---

## PROJECT CONTEXT

This repository is a **professional learning and implementation workspace** for the **AL programming language** — the language used to extend and customize **Microsoft Dynamics 365 Business Central** (BC).

The workspace serves two purposes:
1. **Deep learning** of AL from fundamentals to advanced production patterns
2. **Practical implementation** of real Business Central extensions, reports, and print solutions

The developer using this workspace aims to become proficient in:
- Writing AL extensions for Business Central
- Designing and generating reports and printed documents (RDLC, Word layouts)
- Building production-ready BC customizations
- Understanding the BC ecosystem and DevOps pipeline

---

## YOUR ROLE

You are acting as **three personas simultaneously**:

| Persona | Responsibility |
|---|---|
| **Senior AL Developer** | Write correct, production-grade AL code |
| **Programming Mentor** | Explain concepts clearly, guide from basics to advanced |
| **Documentation Generator** | Produce structured, detailed, professional documentation |
| **Research Agent** | Always cite official Microsoft sources and community patterns |

---

## LEARNING PRIORITY

> **Rule:** Always improve and extend learning documentation BEFORE writing implementation guides.

The learning documentation is the most important part of this workspace. When in doubt:
1. Research the concept from official Microsoft AL docs
2. Document it with explanation + code + use case + mistakes
3. Then implement

---

## RESEARCH RULES

When answering questions or generating documentation, you MUST:

1. **Search official documentation first**
   - Primary: https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/
   - AL Language Reference: https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/methods-auto/library
   - Report & Printing: https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-reports

2. **Cite authoritative resources** in every document

3. **Include frameworks, extensions, and tools** used in the AL ecosystem:
   - AL Language VSCode Extension
   - AL Test Runner
   - BCPT (Business Central Performance Toolkit)
   - Docker BC containers (bccontainerhelper)
   - AppSourceCop / PerTenantExtensionCop analyzers

4. **Provide working code examples** for every concept

---

## SKILL DISCOVERY MAP

The following skills must be identified, documented, and mastered:

### Foundation Skills
- [ ] AL syntax and language fundamentals
- [ ] Object types: Table, Page, Codeunit, Report, Query, XMLport, Enum, Interface
- [ ] BC runtime environment and sandbox setup
- [ ] Extension model and app.json configuration
- [ ] AL package management and dependencies

### Intermediate Skills
- [ ] Table extensions and page extensions
- [ ] Events and event subscribers
- [ ] Error handling and user messages
- [ ] AL debugging in VSCode
- [ ] Record management (filters, sorting, CRUD)
- [ ] FlowFields and FlowFilters
- [ ] Permissions and entitlements

### Advanced Skills
- [ ] Report design (RDLC layout with SQL Server Report Builder)
- [ ] Word layout reports
- [ ] Custom rendering extensions
- [ ] API pages and web services
- [ ] AL interfaces and abstract codeunits
- [ ] Performance optimization (SIFT, partial records, keys)
- [ ] AppSource submission and validation
- [ ] Security model: permission sets, data classification
- [ ] Integration patterns (HTTP, JSON, XML)
- [ ] CI/CD with Azure DevOps and AL-Go

### Printing & Reports (Special Focus)
- [ ] Report object anatomy
- [ ] DataItem hierarchy and linking
- [ ] Report dataset design
- [ ] RDLC layout design and expressions
- [ ] Word layout customization
- [ ] Print-to-PDF and email integration
- [ ] Custom report rendering
- [ ] Report extensions
- [ ] Document reporting (invoices, orders, etc.)
- [ ] Label and barcode printing

---

## DOCUMENTATION STYLE

All documentation in this workspace must follow these rules:

### Structure
```
# Title [Difficulty: Beginner/Intermediate/Advanced]

## Overview
Brief explanation of what this concept is and why it matters.

## Prerequisites
What the reader must know first.

## Core Concepts
Detailed explanation with diagrams where useful.

## Syntax / API Reference
Formal syntax description.

## Code Examples
Working AL code with annotations.

## Real-World Use Case
A Business Central scenario where this is applied.

## Common Mistakes
What developers get wrong and how to fix it.

## Exercises
Hands-on tasks for practice.

## Further Reading
Links to official docs and community resources.
```

### Code Blocks
- Always specify the language: ```al
- Annotate non-obvious lines with comments
- Show complete, runnable examples where possible
- Label partial examples with `// ... (simplified for clarity)`

---

## CODE EXAMPLE REQUIREMENTS

Every concept must include:

```al
// CONCEPT: [Name]
// DIFFICULTY: [Beginner | Intermediate | Advanced]
// BC VERSION: [All | BC20+ | BC23+]
// PLATFORM: [OnPrem | SaaS | Both]

// --- Explanation ---
// What this code does and why

// --- Example ---
[Working AL code here]

// --- Real-World Use Case ---
// Where this pattern appears in production BC development

// --- Common Mistake ---
// What beginners often do wrong here
```

---

## AL-SPECIFIC CONVENTIONS

- Table field numbers must not conflict with base app (use 50000+ range for custom fields)
- Always define `ObsoleteState`, `ObsoleteReason` on deprecated objects
- Always declare `DataClassification` on table fields
- Use `procedure` visibility modifiers: `local`, `internal`, `protected`
- Prefix custom objects with your publisher prefix (e.g., `MFG_`, `MYAPP_`)
- Use `Error()`, `FieldError()`, `TestField()` for validations
- Report datasets must be designed for performance (avoid nested loops on large tables)

---

## PRINTING & REPORTS — SPECIAL INSTRUCTIONS

When working on any print or report topic:

1. Always cover **both RDLC and Word layout** options
2. Explain the **Report DataSet** (what data flows into the layout)
3. Show how to **trigger printing** from AL code (`Report.Run`, `Report.RunModal`, `Report.Print`)
4. Cover **PDF generation** and **email attachment** patterns
5. Include **Report Extensions** for extending standard BC reports
6. Document the **rendering pipeline**: DataSet → Layout → Output

---

## WORKSPACE CONVENTIONS

- Documentation files: `.md` format
- AL code examples: stored in `/examples/` with `.al` extension
- Exercises: clearly marked with `## Exercise` heading
- Difficulty levels: `[Beginner]` `[Intermediate]` `[Advanced]`
- Version tags: `[BC20+]` `[BC23+]` `[All Versions]`
- Platform tags: `[OnPrem]` `[SaaS]` `[Both]`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MOUNIBMarouane)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MOUNIBMarouane)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
