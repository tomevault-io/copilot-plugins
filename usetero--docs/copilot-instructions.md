## docs

> This is Tero's public documentation repository, built with [Mintlify](https://mintlify.com). It contains customer-facing documentation for understanding, integrating, and using Tero.

# Tero Documentation Repository

This is Tero's public documentation repository, built with [Mintlify](https://mintlify.com). It contains customer-facing documentation for understanding, integrating, and using Tero.

## What This Repo Is

A Mintlify-based documentation site with:
- **MDX files** - Documentation pages (content)
- **docs.json** - Navigation and site configuration
- **docs/** - Internal planning docs (organization, writing guide, outlines)
- **Taskfile.yml** - Development commands
- **Hermit** - Tool management (node, pnpm, task)

This is a public repository. Customers may contribute. Write accordingly.

## Repository Structure

```
docs/
├── docs/               # Internal planning (not published)
│   ├── organization.md # How docs are structured and why
│   ├── manual.md       # Complete Manual outline
│   └── writing.md      # Writing guidelines
├── manual/             # Manual section (progressive journey)
├── integrations/       # Vendor-specific guides
├── reference/          # Component documentation
├── trust/              # Trust Center content
├── snippets/           # Reusable content blocks
├── docs.json           # Navigation configuration
└── *.mdx               # Documentation pages
```

## Documentation Organization

We follow a three-section structure:

**Manual** - Progressive journey from first contact to organizational rollout. Designed to be read sequentially. Each section builds on the last.

**Integrations** - Vendor-specific guides (Datadog, Splunk, etc.). Self-contained, tell the complete integration story.

**Reference** - Component documentation (CLI, Edge, API). Non-progressive, random-access lookup.

See `docs/organization.md` for the complete thinking behind this structure.

## Writing Documentation

We follow the [Diataxis framework](https://diataxis.fr/) for documentation types and [Google's Developer Documentation Style Guide](https://developers.google.com/style) for technical writing.

Tero's style:
- Conversational but not casual
- Direct, no corporate speak
- Action-oriented, respect reader's intelligence
- Progressive disclosure (simple → complex)

See `docs/writing.md` for complete guidelines and type-specific guidance (Tutorial vs How-to vs Reference vs Explanation).

## Key Principles

**Only publish what exists** - Don't create pages marked "Coming Soon." Add sections as features ship. This shows momentum instead of a TODO list.

**Progressive trust** - The Manual follows how customers actually adopt: understand → analyze (read-only) → configure → deploy edge → scale org-wide. Each step builds trust.

**Self-contained integrations** - Each vendor integration page tells the complete story. Don't scatter setup across multiple sections.

**Reference is for lookup** - Not progressive. Jump to what you need. Self-contained pages.

## Common Tasks

### Writing a New Doc Page

1. Determine the type (Tutorial, How-to, Reference, Explanation) - see Diataxis
2. Check `docs/manual.md` or relevant outline for where it fits
3. Write following `docs/writing.md` guidelines
4. Add to `docs.json` navigation
5. Use snippets for reusable content (permissions, verification steps)

### Updating Docs

1. Read `docs/writing.md` for editing principles
2. Rewrite sections to stay coherent - don't just append
3. Maintain voice throughout
4. Update `docs.json` if structure changes

### Checking Quality

Run `task check:links` to find broken links.

Format and lint tasks are placeholders - add tooling as needed (prettier, vale, etc.).

## Development

**Setup:**
```bash
# Hermit activates automatically with direnv
# Or manually: source bin/activate-hermit

pnpm install
```

**Common commands:**
```bash
task dev          # Start Mintlify dev server
task do           # Fast dev loop (format + lint)
task check:links  # Check for broken links
task build        # Build documentation
```

**Preview:** Dev server runs at `http://localhost:3000`

## Navigation (docs.json)

Navigation follows this structure:
```json
{
  "navigation": {
    "tabs": [
      { "tab": "Manual", "groups": [...] },
      { "tab": "Integrations", "groups": [...] },
      { "tab": "Reference", "groups": [...] }
    ],
    "global": {
      "anchors": [
        { "anchor": "Trust Center", ... }
      ]
    }
  }
}
```

Tabs appear in top navigation. Global anchors are persistent sidebar items (Trust Center, Support, Status).

## For Tero Employees

Reference these internal docs for deeper context:
- `../knowledge-base/meta/writing.md` - Company writing style
- `../.github-private/manual/documentation.md` - Repository documentation practices

## Mintlify-Specific

**Components:** Mintlify provides rich components (Card, Accordion, Tabs, CodeGroup, etc.). See [Mintlify docs](https://mintlify.com/docs) for usage.

**Snippets:** Create reusable content in `snippets/` and import with `<Snippet file="name.mdx" />`

**Frontmatter:** Every page needs:
```yaml
---
title: "Page Title"
description: "Brief description"
---
```

**Links:** Use relative paths: `[Link text](/path/to/page)` (no .mdx extension)

## The Goal

These docs are a sales tool. Prospects read them to evaluate Tero. Well-written docs signal clear product thinking, practical understanding of customer problems, and respect for reader intelligence.

Get customers to success as quickly as possible while building the understanding they need. That's what we're building.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/usetero)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/usetero)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
