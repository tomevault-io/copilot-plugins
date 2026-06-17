## claude-marketing

> A marketing co-founder for any project, powered by Claude Code.

# Claude Marketing

A marketing co-founder for any project, powered by Claude Code.

## What This Is

A plug-and-play marketing system for Claude Code. Drop it into any project and get:
- 29 expert marketing skills (cold email, SEO, social content, CRO, launch strategy, etc.)
- 51 CLI tools for marketing platforms (Apollo, HubSpot, Instantly, Buffer, etc.)
- Ready-to-use slash commands (`/marketing`, `/content-batch`, `/cold-emails`, `/crm`)
- Cold email templates, content calendar structure, pipeline tracking
- Product marketing context framework that all skills reference

## How to Use

### 1. Set Up Your Product Context
Run `/setup-context` or manually create `.claude/product-marketing-context.md` in your project. This captures your product, audience, positioning, and voice. All marketing skills reference it automatically.

### 2. Run Marketing Commands
| Command | What It Does |
|---------|-------------|
| `/setup-context` | Create your product marketing context (run this first) |
| `/marketing` | Execute today's highest-priority marketing task |
| `/content-batch` | Generate a full week of social content |
| `/cold-emails` | Write cold email sequences |
| `/crm` | Manage your sales pipeline |
| `/launch` | Plan a product launch |
| `/seo` | Run an SEO audit |

### 3. Use Marketing Skills
Skills are loaded automatically when relevant. You can also reference them directly:
- "Write a cold email for my SaaS product" → uses `cold-email` skill
- "Create social content for this week" → uses `social-content` skill
- "Audit my landing page for conversions" → uses `page-cro` skill

## Structure
```
claude-marketing/
├── CLAUDE.md                  # This file
├── .claude/
│   └── commands/              # Slash commands
│       ├── setup-context.md   # Set up product marketing context
│       ├── marketing.md       # Daily marketing co-founder
│       ├── content-batch.md   # Weekly content generation
│       ├── cold-emails.md     # Cold email sequence writing
│       ├── crm.md             # Pipeline management
│       ├── launch.md          # Launch planning
│       └── seo.md             # SEO audit
├── skills/                    # 29 marketing skills
├── tools/                     # 51 CLI tools + integration guides
├── templates/
│   ├── cold-emails/           # Cold email sequence templates
│   └── email-sequences/       # Lifecycle email templates
├── marketing/
│   ├── TRACKER.md             # Pipeline + metrics tracker (template)
│   └── content/               # Generated content goes here
└── docs/
    ├── SETUP.md               # Full setup guide
    ├── MARKETING-STACK.md     # Recommended tool stack
    └── SKILLS-REFERENCE.md    # All 29 skills explained
```

## Skills Available (29)

**Conversion Optimization**: page-cro, signup-flow-cro, onboarding-cro, form-cro, popup-cro, paywall-upgrade-cro

**Content & Copy**: copywriting, copy-editing, cold-email, email-sequence, social-content, content-strategy

**SEO & Discovery**: seo-audit, ai-seo, programmatic-seo, competitor-alternatives, schema-markup

**Paid & Distribution**: paid-ads, ad-creative

**Measurement & Testing**: analytics-tracking, ab-test-setup

**Retention**: churn-prevention

**Growth & Strategy**: free-tool-strategy, referral-program, marketing-ideas, marketing-psychology, launch-strategy, pricing-strategy, product-marketing-context

## CLI Tools (51)

Zero-dependency Node.js CLIs for marketing platforms. Set `{TOOL}_API_KEY` env var and go.

See `tools/REGISTRY.md` for the full list.

## Requirements
- Claude Code
- Node.js 18+ (for CLI tools)

---
> Source: [whyujjwal/claude-marketing](https://github.com/whyujjwal/claude-marketing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
