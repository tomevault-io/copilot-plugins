## planktonsoup-editable-software-diagrams

> Create or update software-development diagrams in Mermaid (`.mmd` or Markdown code fences) or draw.io (`.drawio`) format. Use when an agent needs human-readable, diffable diagrams for system architecture, service boundaries, runtime flows, sequence diagrams, ERDs, deployment layouts, state machines, dependency maps, integration boundaries, or technical documentation, and the output should remain easy for humans and agents to review and edit later. Stay focused on Mermaid and draw.io only unless the user explicitly asks for another format.


# Editable Diagrams

Create software-development diagrams in text-first formats that work well in agentic workflows. Prefer Mermaid for concise source-controlled diagrams; prefer draw.io when layout, positioning, or mixed visual elements matter.

This file is the runtime skill contract used when agents execute this skill.
Repository maintenance and governance instructions belong in `AGENTS.md`, not here.

## Scope

Create or edit:

- Mermaid source in `.mmd` files or Markdown fenced blocks
- draw.io / diagrams.net `.drawio` XML files

For: architecture, service, flow, sequence, deployment, ERD, state, integration, and dependency diagrams.

Do not use PlantUML, Excalidraw, SVG, PNG, Visio, or image-generation unless the user explicitly overrides scope.

## Workflow

1. Inspect the request, code, docs, and existing diagrams before choosing format or abstraction level.
2. Base the diagram on the actual implementation — real modules, services, tables, queues, endpoints, boundaries.
3. Use MCP tools when they materially improve accuracy or save time. Prefer local workspace inspection first; use repository/GitHub MCP for remote context; use product-specific MCP for external APIs or schemas. Do not invent MCP dependencies.
4. Reuse the existing format when editing unless the user asks to migrate.
5. If the diagram type or notation is unclear, research the common convention before inventing a structure.
6. Choose the simplest editable format that preserves intent when no format is specified.
7. Produce diagram source directly — do not describe without creating unless concepts-only is requested.
8. Validate parser status and rendered structure, not just text syntax. If a required rule fails, the diagram is not complete.
9. Keep output diff-friendly.

## Non-Negotiable Rules

- Required rules are mandatory, not preferences.
- If a format-specific rule fails in preview, render, or source review, the diagram is not complete.
- Do not trade correctness, readability, or theme-safe rendering for speed or layout preservation.
- Format-specific rules override general preferences when they conflict.

## Format Selection

Priority order:

1. Use the format the user names.
2. Preserve the format of an existing target file.
3. Prefer Mermaid in Markdown when the output belongs in docs or should be easy to view in place.
4. Prefer draw.io when exact placement, swimlanes, or canvas composition matter.
5. Default to Mermaid — lighter to diff and edit.

**Migration**: Only migrate between formats when the user requests it. Preserve labels, grouping intent, edge meaning, and terminology. Do not silently migrate during unrelated edits.

## Editing Existing Diagrams

- Read the existing file first; preserve current format.
- Make the smallest coherent change that satisfies the request.
- Preserve labels, IDs, layout intent, and surrounding doc structure.
- Do not revamp unless the request requires it or the diagram violates correctness/readability requirements.
- Keep files UTF-8 without BOM.
- For draw.io with compressed/encoded content, decode to raw XML before editing.
- For Mermaid in Markdown, edit only the targeted fenced block unless surrounding prose also needs updates.

## Mermaid

Emit valid Mermaid with a single top-level diagram declaration, short stable node IDs, and human-readable labels. Use the diagram family that matches the request. Keep labels concise and parser-safe. Connect edges to concrete nodes, not subgraph IDs. Keep connector labels background-transparent. **Never use `\n` in labels — use `<br/>` for multiline labels or shorten to one line.**

Mermaid renderer compatibility profile (required by default):

- Target broad renderer support first, especially GitHub Markdown rendering.
- For architecture and system-overview diagrams, use `flowchart`, not `architecture-beta`.
- Do not use Mermaid experimental or unevenly supported diagram families (`architecture-beta`, `c4`, `packet`, `block`, `sankey`, `xyChart`, `quadrantChart`, `requirement`) unless the user explicitly asks and accepts renderer limitations.
- Avoid renderer-sensitive directives and features unless required: prefer no `%%{init: ...}%%` blocks, no edge IDs or animation syntax, and no custom icon packs.

Label line-break rule:

- Never emit literal `\n` inside Mermaid labels.
- For visual label breaks, use `<br/>` where supported, or rewrite to a shorter single-line label.

Read [references/mermaid.md](references/mermaid.md) for diagram type selection, label safety, CLI validation, and rendering constraints.
Start from [assets/mermaid-doc-template.md](assets/mermaid-doc-template.md) for new Markdown-hosted diagrams.

## Draw.io

Emit valid `.drawio` XML with `compressed="false"`. Use explicit `source`/`target` connector IDs. Always set an explicit page background color. Use visible connector strokes and arrow colors that contrast with the background. Use dedicated semi-opaque label vertices for edge and branch labels.

Read [references/drawio.md](references/drawio.md) for XML structure, editing rules, label treatment, and validation.
Start from [assets/blank.drawio](assets/blank.drawio) for new diagrams.

## Styling

Use high-contrast presentation: dark saturated fills with white text, a small set of semantic color families, distinct container outlines versus connector lines, and a legend when more than three semantic colors appear. All text must contrast with its immediate surface — treat weak contrast as a correctness failure.

Read [references/styling.md](references/styling.md) for palette strategy, connector treatment, labeling rules, and layout guidelines.

## Output Conventions

- Embed Mermaid in a nearby `.md` file when developers should view it in docs. Use standalone `.mmd` when the diagram is reused, large, or already exists as `.mmd`.
- Wrap Mermaid in ` ```mermaid ` fenced blocks with a short heading or caption above.
- Save draw.io with `.drawio` extension.
- When replacing ASCII diagrams or prose descriptions, preserve original meaning.
- Create or edit a documentation artifact rather than leaving the result only in chat.
- Update the nearest `README.md` or diagram index when adding or renaming persisted diagrams.
- Place diagrams in established project directories (`diagrams/`, `docs/diagrams/`, etc.) or alongside the docs they support. Preserve existing placement conventions.
- Use descriptive file names (`auth-sequence.mmd`, `deployment.drawio`). Avoid generic names like `diagram1.drawio`.
- For Markdown-hosted diagrams: add a concise heading and 1–2 context lines above, keep one primary diagram per section, add a small TOC if the file has multiple diagrams.
- Do not make repository documentation depend on external viewer links unless the user explicitly asks for them.
- Start from [assets/mermaid-doc-template.md](assets/mermaid-doc-template.md) when creating a new diagram-focused Markdown page.

## Validation

Before finalizing, verify:

**Cross-format:**

- Zero parser/lexer/validation errors for every created or edited diagram source.
- For Markdown-hosted Mermaid, validate every edited fenced diagram block, not just one sample block.
- Every edge starts and ends on an actual rendered node or shape.
- Arrowheads visibly touch intended targets in preview.
- All text contrasts with its immediate surface.
- No overlapping lines that read as a single line.
- Containers are visually distinct from connectors.
- If the diagram is too crowded to verify clearly, simplify or split it.
- If preview reveals detached connectors, mis-anchored labels, or ambiguous targets — fix before completing.
- If any preview, renderer, or validator reports a parse/render error, fix the source and re-validate before completion.

**Format-specific validation** is defined in each reference:

- [references/mermaid.md](references/mermaid.md) — label safety, node ID integrity, `mmdc` CLI validation
- [references/drawio.md](references/drawio.md) — XML safety checklist, CLI validation, connector/label rendering

For Mermaid outputs, treat any literal `\n` token in labels as a correctness failure that must be rewritten before completion.

Use local CLI tools (`mmdc`, `drawio`) when available. Probe for them cross-platform before falling back to preview or manual review.
Do not mark the task complete while any parse/render/load error remains unresolved.

## Request Defaults

When underspecified, infer the smallest useful diagram:

- Architecture → main runtime components and edges
- Workflow → major states and decisions
- Data model → entities and cardinality
- Sequence → main actors, messages, key responses/failures
- Deployment → runtime units, boundaries, key dependencies

Use real code names over placeholders. Show system boundaries when crossing services or trust zones. Distinguish internal from external. Favor a single abstraction level: system context, container/service, runtime interaction, deployment topology, or data model.

## Quality Bar

- Answer the user's actual question — do not draw every possible component.
- Optimize for readability at a glance.
- Keep terminology consistent with surrounding code and docs.
- Keep abstraction level explicit; do not mix detail levels unless the request requires both.
- Split crowded diagrams instead of cramming.
- Trust authoritative MCP sources over local assumptions.
- Avoid decorative complexity — the artifact should be easier to maintain after the edit.
- Treat visually detached arrows, labels, or connectors as correctness bugs.

---
> Source: [planktonsoup-labs/planktonsoup-editable-software-diagrams](https://github.com/planktonsoup-labs/planktonsoup-editable-software-diagrams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
