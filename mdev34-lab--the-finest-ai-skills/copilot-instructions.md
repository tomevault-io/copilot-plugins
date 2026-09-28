## the-finest-ai-skills

> Meta-rules governing how all directives in this workspace are edited, structured, compressed, and maintained. Apply whenever modifying any directive file.


# Agents

## I. Directive Editing Rules

1. **Surgical edits by default.** For full rewrites or large structural changes, request user permission first.
2. **Deduplication & structure.** Always keep sections properly deduplicated and organized. When pruning, target: (a) duplicate content referencing earlier sections, (b) repetitive structural patterns, (c) verbose explanations reducible without meaning loss, (d) expanded paragraphs replaceable by consolidated bullets.
3. **No overfitting.** All gradient updates must be generalizable to all tasks — not tailored to a single edge case or piece of feedback.
4. **Designer.md constraint.** Do not auto-create new extensions or archetypes without explicit user instruction.
5. **Cleanliness.** Edits must preserve or improve the directive's formatting consistency, tone uniformity, and structural clarity. Do not introduce new formatting patterns (e.g., ALL CAPS emphasis, new header styles, mixed list conventions) that conflict with the existing style.

## II. Metadata

All directives must include an Agent Skills metadata block. You may add it without explicit permission:

```markdown
---
name: {name}
description: {dense, routing-optimized description of use cases}
license: CC BY 4.0
metadata:
    author: mdev34-lab
    works_with: {preferably one of "Any LLM", "Any vision LLM", "Vibe coding agents", "LLMs capable of image generation", but can be custom}
    tldr: {proper description of what the skill does}
---
```

The description must densely transmit applicable use cases. Do not use it for update notes or development remarks — it is critical for runtime routing.

## III. Compression & Line Count

- **Line count balance:** Per edit, find compression or pruning opportunities elsewhere in the directive. Target net delta: −5 to +5 lines (tighter for minor edits, up to the higher bound for large rewrites).
- **Token efficiency:** Write directives for density over human readability. Prefer tight, unambiguous phrasing.

Always try **replacing** instead of blindly **adding** to reduce bloat.

## IV. Feedback Application

If a message contains an analysis, feedback, or optimal result with no explicit instruction, assume: *"Apply this feedback to the relevant directive so the mistake doesn't recur."*

Maximize analyzed upsides while minimizing downsides. Do not trade off apparent opposites (e.g., elegance vs. functionality) — maximize both unless absolutely necessary.

## V. Vibe Coding Integration

When working with vibe coding agents (Windsurf, Kilo, etc):
- Treat them as updaters, not creators
- Provide clear, actionable feedback for iterative improvement
- Maintain consistent metadata and formatting standards
- Ensure all modifications preserve the directive's core purpose and structure

## VI. Quality Standards

All directives must maintain:
- **Functional clarity** - Clear purpose and application
- **Structural integrity** - Consistent organization and formatting
- **Metadata completeness** - All required fields populated
- **License compliance** - CC BY 4.0 for all files
- **Version awareness** - Respect archived versions and maintain history

---
> Source: [mdev34-lab/the-finest-ai-skills](https://github.com/mdev34-lab/the-finest-ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
