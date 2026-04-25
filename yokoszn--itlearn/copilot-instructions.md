## itlearn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ITLearn is a Quartz-based digital garden that serves as the learning wing of TWN Systems. It transforms structured IT/cybersecurity learning content from Obsidian notes into a professional web platform focused on digital sovereignty fundamentals.

**Tech Stack:**
- Quartz 4.5.1 (Static Site Generator)
- TypeScript/Node.js
- Obsidian-flavored Markdown
- Cloudflare Pages deployment

## Development Commands

### Core Operations
- `npm run quartz build` - Build the static site
- `npm run quartz build --serve` - Build and serve locally (typically at http://localhost:8080)
- `npm run docs` - Build and serve docs (alias for build --serve -d docs)
- `npm run check` - Run TypeScript checks and Prettier formatting validation
- `npm run format` - Format code with Prettier
- `npm run test` - Run test suite

### Content Management
- Content lives in `content/` directory
- Use `draft: true` in frontmatter for incomplete/private notes
- Quartz automatically filters drafts via RemoveDrafts() plugin

## Architecture

### Quartz Configuration Files
- `quartz.config.ts` - Main site configuration (title, theme, plugins, base URL)
- `quartz.layout.ts` - Component layouts for pages and shared elements
- `content/` - All markdown content and assets
- `quartz/` - Quartz framework code (avoid modifying unless necessary)

### Content Structure
```
content/
├── index.md                    # Homepage
├── getting-started.md         # User onboarding
├── paths/                     # Structured learning paths
│   ├── networking.md
│   ├── linux.md
│   └── sre.md
├── 00 Learn/                  # Legacy Obsidian structure (being reorganized)
├── 01 Computing Fundamentals/ # Subject-based content
└── [numbered directories]/    # Various learning modules
```

### Key Features
- **ObsidianFlavoredMarkdown**: Supports `[[wikilinks]]`, callouts, embeds
- **GitHubFlavoredMarkdown**: Tables, task lists, strikethrough
- **Callouts**: Use `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]` syntax
- **Graph visualization**: Automatic linking and relationship mapping
- **Search**: FlexSearch integration
- **Analytics**: Configured for Plausible

## TWN Systems Integration

### Branding
- Site title: "TWN Learn" 
- Tagline: "Guided paths into digital sovereignty, with Commons support"
- Primary CTA: Direct users to TWN Commons Discord for help/community

### Content Strategy
- Each learning path should end with a "Bridge to TWN" section
- Highlight TWN providers and tools within relevant learning content  
- Community integration via Discord callouts and help sections

### Footer Links Template
```typescript
footer: Component.Footer({
  links: {
    "TWN Systems": "https://twn.systems",
    "TWN Commons": "https://discord.gg/kgaMm6WJya", 
    "GitHub": "https://github.com/[repository]",
  },
}),
```

## Content Authoring Guidelines

### Learning Path Structure
Each path in `content/paths/` should follow:
1. **Overview** - Why it matters, who it's for
2. **Prerequisites** - What learners need first  
3. **Milestones** - Clear checkpoints with actionable items
4. **Resources** - Links to tools, documentation, practice labs
5. **Community Support** - Discord integration callout
6. **Bridge to TWN** - Connection to relevant TWN tools/providers

### Callout Usage
```markdown
> [!TIP] Need guidance?
> Join the [TWN Commons on Discord](https://discord.gg/kgaMm6WJya) to ask questions and connect with other learners.

> [!NOTE] Bridge to TWN
> Deploy sovereign infrastructure: explore providers on [TWN Systems](https://twn.systems)
```

### Draft Management
- Add `draft: true` to frontmatter for incomplete content
- RemoveDrafts() plugin automatically excludes from builds
- Helps maintain professional appearance while content is in development

## Common Tasks

### Adding New Learning Path
1. Create `content/paths/[topic].md`
2. Follow learning path structure template
3. Add TWN integration callouts
4. Update navigation if needed

### Theme Customization
- Colors defined in `quartz.config.ts` under `theme.colors`
- Typography: Schibsted Grotesk (headers), Source Sans Pro (body), IBM Plex Mono (code)
- Custom components in `quartz/components/` (modify carefully)

### Local Development
1. `npm run quartz build --serve` 
2. Navigate to http://localhost:8080
3. Changes to content rebuild automatically
4. Config changes require restart

## Deployment

- Deployed via Cloudflare Pages
- Base URL: configured in quartz.config.ts
- Build command: `npm run quartz build`
- Output directory: `public/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yokoszn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
