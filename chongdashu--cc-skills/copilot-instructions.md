## cc-skills

> A curated collection of professional-grade Claude Code skills designed to enhance Claude's capabilities across design, content creation, and web optimization.

# BuilderPack: Claude Code Skills Collection

A curated collection of professional-grade Claude Code skills designed to enhance Claude's capabilities across design, content creation, and web optimization.

## Overview

This repository contains **5 production-ready skills** that follow a philosophy-first design approach, emphasizing quality, variation, and anti-pattern prevention:

1. **favicon-generator** - Professional favicon generation with multi-layer effects
2. **frontend-design** - Distinctive, production-grade frontend interfaces with high design quality
3. **og-image-creator** - Authentic Open Graph social sharing images from your codebase
4. **site-metadata-generator** - Comprehensive SEO optimization and metadata generation
5. **youtube-titles-thumbnails** - YouTube content optimization for titles and thumbnails

Each skill is designed to unlock Claude's capabilities rather than constrain them to rigid templates.

## Quick Reference: What & Why

| Skill | What It Does | Why Use It |
|-------|-------------|-----------|
| **favicon-generator** | Generates professional favicons with depth, lighting, and texture | Get app-quality icons (like Linear/Notion) without design tools |
| **frontend-design** | Creates distinctive, production-grade frontend interfaces | Build visually striking web UIs that avoid generic AI aesthetics |
| **og-image-creator** | Creates brand-aligned social sharing images by analyzing your codebase | Automatic, contextual OG images that match your actual brand |
| **site-metadata-generator** | Comprehensive SEO optimization with meta tags and structured data | Improve search rankings and discoverability with proper metadata |
| **youtube-titles-thumbnails** | Generates high-CTR title/thumbnail pairs using curiosity frameworks | Maximize clicks while maintaining authenticity and trust |

## Skills

### 1. Favicon Generator

**What It Does**: Generates professional-quality favicons with multi-layer visual effects (drop shadows, inner glows, highlights, gradients, noise) that rival icons from Linear, Notion, and Figma.

**Why Use It**:
- Get polished, app-quality icons without opening a design tool
- Match favicons to your existing brand icons (discovers icons from your codebase)
- Complete favicon suite for web, PWA, and mobile (ICO, SVG, PNG at all sizes)
- Quick prototyping or production-ready output depending on your needs

**Key Capabilities**:
- 8 curated design templates (Modern, Vibrant, Minimal, Glass, Neon, Warm, Forest, Mono)
- 18 pre-configured Lucide icons + custom letters/emoji support
- Brand icon discovery workflow (searches your codebase)
- Dual tooling: interactive HTML generator + Python CLI
- Framework integration code (Next.js, HTML, PWA)

**Deliverables**:
- favicon.ico, favicon.svg, multiple PNG sizes (16-512px)
- apple-touch-icon.png (180x180)
- PWA images (192x192, 512x512)
- Integration code snippets

**Philosophy**: "Favicons Are Miniature Design Artifacts" - the difference between mediocre and great favicons is polish, not complexity.

**Location**: `.claude/skills/favicon-generator/`

### 2. OG Image Creator

**What It Does**: Generates authentic, brand-aligned Open Graph (social sharing) images by analyzing your codebase to extract colors, fonts, and logo, then creating contextually appropriate images for each route.

**Why Use It**:
- Automatic social sharing previews that match your actual brand (not generic templates)
- Context-aware designs (landing pages get different treatments than articles)
- Framework-agnostic (Next.js, Astro, React Router, Gatsby, static HTML)
- Complete workflow from analysis to deployment

**Key Capabilities**:
- Automated codebase analysis and route discovery
- Brand identity extraction (colors from CSS/Tailwind, fonts from stylesheets, logo from assets)
- Page type categorization (landing, article, product, documentation, about)
- Component-based generation using Playwright and React
- Platform-specific optimization (Facebook, Twitter/X, LinkedIn, Slack, Discord)

**Deliverables**:
- Analysis JSON (routes, brand colors, fonts, logo paths, page categorization)
- OG images (1200x630px PNG/JPG) in public/og/ directory
- Framework-specific meta tag integration code
- Platform testing checklist

**Philosophy**: "Authentic Over Template" - study the codebase before generating, use actual brand elements, different page types need different visual treatments.

**Location**: `.claude/skills/og-image-creator/`

### 3. Frontend Design

**What It Does**: Creates distinctive, production-grade frontend interfaces with high design quality that avoid generic "AI slop" aesthetics.

**Why Use It**:
- Build visually striking web components, pages, and applications
- Avoid cookie-cutter designs with bold aesthetic choices
- Get production-ready code with exceptional attention to detail
- Choose from diverse design philosophies (brutalist, maximalist, minimalist, retro-futuristic, etc.)

**Key Capabilities**:
- Bold aesthetic direction selection and execution
- Distinctive typography choices (avoiding generic fonts)
- Creative color schemes and cohesive theming
- Sophisticated animations and micro-interactions
- Unexpected layouts and spatial composition
- Atmospheric backgrounds and visual effects
- Framework-agnostic (HTML/CSS/JS, React, Vue, etc.)

**Deliverables**:
- Production-grade, functional code
- Visually striking and memorable interfaces
- Cohesive designs with clear aesthetic point-of-view
- Framework integration ready

**Philosophy**: "Bold Intentionality Over Timid Convention" - Choose a clear conceptual direction and execute it with precision. Both maximalism and minimalism work when executed with intentionality and context-specific character.

**Location**: `.claude/skills/frontend-design/`

### 4. Site Metadata Generator

**What It Does**: Provides comprehensive SEO optimization for web applications through proper metadata, structured data, and search engine communication.

**Why Use It**:
- Improve search rankings and discoverability
- Generate proper meta tags for all pages
- Implement structured data (JSON-LD) for rich search results
- Framework-native SEO (Next.js, Astro, React, static HTML)
- Optimize for Core Web Vitals and technical SEO

**Key Capabilities**:
- Automated codebase analysis and route discovery
- Meta tag generation (title, description, OG tags, Twitter cards)
- Structured data implementation (Schema.org)
- Sitemap generation and robots.txt configuration
- Framework-specific optimization patterns
- Technical SEO auditing and recommendations

**Deliverables**:
- Meta tags for all routes
- JSON-LD structured data
- Sitemap.xml and robots.txt
- Framework-specific integration code
- SEO analysis and improvement recommendations

**Philosophy**: "SEO as Semantic Communication" - SEO is about clearly communicating what your content IS to machines (search engines, social platforms, AI crawlers) so they can properly understand and surface it. Accuracy over optimization, user intent first.

**Location**: `.claude/skills/site-metadata-generator/`

### 5. YouTube Titles & Thumbnails

**What It Does**: Creates high-performing YouTube titles and thumbnail text pairs that maximize CTR and virality while maintaining authenticity using the Curiosity-Value Framework.

**Why Use It**:
- Maximize click-through rates without resorting to misleading clickbait
- Generate complementary thumbnail + title pairs (not redundant)
- Adapt to different content types (tutorial, analysis, story, comparison)
- Balance SEO discovery with click-through intrigue

**Key Capabilities**:
- Interactive discovery workflow (asks questions before generating)
- 8 title structure patterns (Curiosity + Specificity, Value Promise + Constraint, etc.)
- 5 thumbnail text patterns (Subject Focus, Revelation, Status/Opinion, etc.)
- Content type adaptation with specific patterns
- Emotional hook optimization (Curiosity, Frustration relief, Ambition, FOMO)
- Channel tone adaptation (Authoritative, Casual, Provocative, Educational)

**Deliverables**:
- 3-5 title options using different patterns
- Complementary thumbnail text for each (3-5 words max)
- Rationale for each option (why it fits audience/hook/priority)
- Trade-off analysis (e.g., "better for SEO but slightly less intriguing")

**Philosophy**: "The Curiosity-Value Framework" - Thumbnail (Visual Hook) + Title (Verbal Promise) = Irresistible Curiosity Gap. Complementarity over redundancy, neutrality builds trust.

**Location**: `.claude/skills/youtube-titles-thumbnails/`

## Project Structure

```
builderpack-cc-skills/
└── .claude/
    └── skills/
        ├── favicon-generator/
        │   ├── SKILL.md              # Main skill documentation
        │   ├── references/
        │   │   └── effects-guide.md  # Technical effect details
        │   └── scripts/
        │       ├── generate_favicon.html   # Interactive HTML tool
        │       └── generate_favicon.py     # Python CLI tool
        │
        ├── frontend-design/
        │   └── SKILL.md              # Design philosophy and guidelines
        │
        ├── og-image-creator/
        │   ├── SKILL.md              # Complete workflow guide
        │   ├── references/
        │   │   ├── og-specifications.md      # OG standards & testing
        │   │   ├── best-practices.md         # Design & content strategy
        │   │   └── framework-workflows.md    # Framework-specific guides
        │   └── scripts/
        │       ├── analyze_codebase.py      # Route & brand analysis
        │       └── generate_og_images.py    # Image generation
        │
        ├── site-metadata-generator/
        │   ├── SKILL.md              # SEO optimization guide
        │   └── references/
        │       └── structured-data-schemas.md  # Schema.org templates
        │
        └── youtube-titles-thumbnails/
            ├── SKILL.md              # Comprehensive guide
            └── references/           # (Reserved for future use)
```

## Skill Architecture Pattern

Each skill follows a consistent modular structure:

1. **SKILL.md** (Core Documentation)
   - Philosophy section establishing mental frameworks
   - Actionable guidelines organized by category
   - Anti-patterns with specific examples
   - Variation encouragement
   - Empowering conclusion

2. **references/** (Progressive Disclosure)
   - Detailed technical guides
   - Domain-specific knowledge
   - Loaded only when needed to keep main skill concise

3. **scripts/** (Automation)
   - Executable tools for deterministic operations
   - Complex procedures requiring precision
   - Reusable utilities

4. **examples/** (Learning Resources - Optional)
   - Annotated analysis of effective patterns
   - Before/after transformations
   - Pattern demonstrations

## Design Principles

All skills in this collection follow these core principles:

### 1. Philosophy Before Procedure
Every skill starts with a philosophy section that establishes mental frameworks before diving into procedures. This helps Claude understand the "why" behind the "what."

### 2. Anti-Pattern Prevention
Each skill explicitly names what NOT to do with specific examples, helping avoid common pitfalls and ensuring quality outputs.

### 3. Variation Encouragement
Skills explicitly instruct to vary outputs and avoid convergence on "favorite" patterns, ensuring diverse and creative solutions.

### 4. Progressive Disclosure
Main SKILL.md files stay concise (500-700 lines max), with detailed content moved to references/ directory for on-demand loading.

### 5. Degrees of Freedom
Guidance specificity matches task fragility:
- **High freedom**: Creative/contextual tasks (text guidance)
- **Medium freedom**: Structured tasks (pseudocode/parameters)
- **Low freedom**: Fragile operations (specific scripts)

### 6. Empowerment Over Constraint
Skills unlock Claude's capabilities rather than constraining them to rigid templates or checklists.

## Usage

### Invoking Skills

Skills can be invoked in Claude Code conversations:

```
# Invoke a skill
Please use the favicon-generator skill to create an icon

# Or directly
@skill favicon-generator
```

### Using Scripts

Some skills include executable scripts for specific operations:

```bash
# Favicon generator (Python CLI)
python .claude/skills/favicon-generator/scripts/generate_favicon.py

# OG Image codebase analysis
python .claude/skills/og-image-creator/scripts/analyze_codebase.py

# OG Image generation
python .claude/skills/og-image-creator/scripts/generate_og_images.py
```

Interactive HTML tools (like the favicon generator) can be opened directly in a browser.

## Quality Standards

All skills in this collection follow philosophy-first design principles:

- **Philosophy foundation** - Clear mental frameworks before procedures
- **Anti-pattern prevention** - Explicit examples of what NOT to do
- **Variation encouragement** - Diverse outputs adapted to context
- **Progressive disclosure** - Concise main files with detailed references
- **Empowerment over constraint** - Unlock capabilities, don't limit them

## Contributing

When adding new skills to this collection:

1. Follow the established architecture pattern (SKILL.md, references/, scripts/)
2. Start with philosophy before procedures
3. Include anti-patterns with specific examples
4. Encourage variation and context-specific adaptation
5. Keep main SKILL.md concise (500-700 lines max)
6. Move detailed content to references/ for progressive disclosure

## License

See individual skill files for licensing information.

## Repository Status

- **Platform**: macOS
- **Git**: Public repository
- **Skills**: 5 production-ready skills
- **Supported Frameworks**: Next.js, Astro, React, Vue, Gatsby, Static HTML

---

**Note**: This is a pure Claude Code skills repository with no external dependencies at the root level. Individual skills may have their own dependencies (e.g., Pillow for favicon generation, Playwright for OG image generation).

---
> Source: [chongdashu/cc-skills](https://github.com/chongdashu/cc-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
