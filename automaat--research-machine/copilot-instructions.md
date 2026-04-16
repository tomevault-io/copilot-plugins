## research-machine

> Personal knowledge management using Notion with PARA organization.

# Research & Knowledge Management

## Overview

Personal knowledge management using Notion with PARA organization.
Technical and personal knowledge through Capture → Process → Connect workflow.

**System:** Notion workspace with wikilinks and MOCs
**Structure:** PARA (Projects, Areas, Resources, Archives)
**Primary Use:** Technical research, knowledge synthesis, pattern discovery

---

## Workspace Structure

```text
0_Inbox/          # Capture zone
1_Projects/       # Time-bound work
2_Areas/          # Ongoing responsibilities
3_Resources/      # Reference material (Inbox → here)
4_Archives/       # Completed/inactive
```

**Flow:** Inbox → Process → Resources → Connect via [[wikilinks]] and MOCs

---

## AI Assistance Goals

**Primary functions:**

- **Organizing:** Move from Inbox, suggest structure, identify connections, tag appropriately
- **Summarizing:** Research papers, articles, meetings, technical docs
- **Finding patterns:** Connect notes, spot themes, suggest MOCs, discover gaps
- **Writing:** Draft structure, expand bullets, refine content, create summaries
- **Maintenance:** Find orphans, broken links, missing tags, duplicates
- **Knowledge extraction:** Pull insights, synthesize perspectives, create atomic notes, build connections

---

## Writing Conventions

### Format

- **Use emojis** in note content for readability and categorization
- **Bullet points preferred** over paragraphs
- **Wikilinks** for connections: `[[Related Topic]]`
- **Clear headings** for structure

### File Naming

- Title-based (no strict timestamp convention)
- Natural language, descriptive
- No length restrictions (atomic or comprehensive as needed)

---

## Anti-Patterns

**AVOID:**

- Overcomplicating (keep simple and accessible)
- Oversimplifying (maintain nuance when important)
- Creating information from nothing (base on actual content)
- Ignoring preferences (emojis, bullets, wikilinks, PARA structure)

---

## Note Templates

### Research Note

```markdown
# [Topic Name]

**Source:** [[Source]] or [URL](link)
**Date:** YYYY-MM-DD
**Tags:** #research #[topic]

## Key Points

- Main insight
- Important data
- Connection to [[Concept]]

## Personal Insights

- What this means for [[context]]
- How connects to [[knowledge]]

## Questions

- What unclear?
- What explore next?

## Related

[[MOC]], [[Related Note]]
```

### Technical Note

```markdown
# [Technology/Pattern]

**Category:** #tech #[domain]

## Overview

What this is and why it matters.

## Key Concepts

- Concept 1
- Concept 2

## How It Works

Mechanism/approach in bullets.

## Use Cases

- When to use
- When not to use

## Resources

[[Internal]], [External](URL)

## Connections

[[Pattern]], [[Project]]
```

### MOC (Map of Content)

```markdown
# [Topic] MOC

## Overview

Domain/theme description.

## Core Concepts

[[Concept 1]], [[Concept 2]]

## Deep Dives

### Subtopic A
[[Note 1]], [[Note 2]]

### Subtopic B
[[Note 3]], [[Note 4]]

## Related

[[Broader MOC]], [[Adjacent MOC]]
```

---

## Processing Workflow

**When processing Inbox notes:**

1. Understand content
2. Suggest location (Projects/Areas/Resources/Archives)
3. Identify connections (existing notes, MOCs)
4. Recommend tags
5. Format for consistency (emojis, bullets, wikilinks, headings)

---

## Notion MCP Integration

### Direct Workspace Access

**Workspace:** Connected via Notion MCP

**Operations:**

- Read/write pages
- Search workspace (semantic + keyword)
- Create/update pages and databases
- Move pages between parents
- Add comments
- Update page properties
- Fetch page content
- Query databases

---

## Research Workflow

**My approach when you request research:**

1. **Initial exploration** (2-3 searches): Quick overview, identify key areas
2. **Ask clarifying questions**: Which aspect? Depth needed? Use case?
3. **Deep research** (4-6 searches): Detailed investigation, multiple sources
4. **Synthesize & save**: Comprehensive note with bullets, emojis, wikilinks
5. **Archive locally**: Save to `./findings/[topic]/` (create dir if needed)

**Output format:**

```markdown
# [Research Topic]

**Date:** YYYY-MM-DD
**Tags:** #research #[domain]
**Focus:** [Specific angle]

## Key Findings

- Finding 1 with evidence
- Finding 2 with examples
- Finding 3 with patterns

## Connections

[[Existing Note 1]], [[Existing Note 2]]

## Sources

[Source 1](URL), [Source 2](URL)

## Next Steps

- Explore [[Topic]]
- Create [[MOC]]
- Connect to [[Project]]

---

**Suggested location:** 3_Resources/[Domain]/
**Potential MOCs:** [[Relevant MOC]]
**Tags:** #[tag1] #[tag2]
```

---

## Maintenance Tasks

**Help maintain workspace quality:**

- Find orphaned pages (no links in/out)
- Tag consistency checks
- Duplicate detection
- Inbox overflow assistance
- Database property cleanup

---

## Best Practices

**Do:**

- Use emojis in notes for scanning
- Bullet points over paragraphs
- Wikilink liberally
- Extract insights in own words
- Ask before major restructuring
- Cite sources
- Follow PARA structure
- Create atomic notes
- Suggest tags consistently

**Don't:**

- Create complex hierarchies
- Invent information
- Ignore PARA structure
- Create folders without asking
- Oversimplify technical content
- Remove context when summarizing
- Skip wikilinks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Automaat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
