## rainpipe

> Kiro-style Spec Driven Development implementation on AI-DLC (AI Development Life Cycle)

# AI-DLC and Spec-Driven Development

Kiro-style Spec Driven Development implementation on AI-DLC (AI Development Life Cycle)

## Project Context

### Paths
- Steering: `.kiro/steering/`
- Specs: `.kiro/specs/`

### Steering vs Specification

**Steering** (`.kiro/steering/`) - Guide AI with project-wide rules and context
**Specs** (`.kiro/specs/`) - Formalize development process for individual features

### Active Specifications
- Check `.kiro/specs/` for active specifications
- Use `/kiro:spec-status [feature-name]` to check progress

## Development Guidelines
- Think in English, but generate responses in Japanese (思考は英語、回答の生成は日本語で行うように)

## Minimal Workflow
- Phase 0 (optional): `/kiro:steering`, `/kiro:steering-custom`
- Phase 1 (Specification):
  - `/kiro:spec-init "description"`
  - `/kiro:spec-requirements {feature}`
  - `/kiro:validate-gap {feature}` (optional: for existing codebase)
  - `/kiro:spec-design {feature} [-y]`
  - `/kiro:validate-design {feature}` (optional: design review)
  - `/kiro:spec-tasks {feature} [-y]`
- Phase 2 (Implementation): `/kiro:spec-impl {feature} [tasks]`
  - `/kiro:validate-impl {feature}` (optional: after implementation)
- Progress check: `/kiro:spec-status {feature}` (use anytime)

## Development Rules
- 3-phase approval workflow: Requirements → Design → Tasks → Implementation
- Human review required each phase; use `-y` only for intentional fast-track
- Keep steering current and verify alignment with `/kiro:spec-status`

## Steering Configuration
- Load entire `.kiro/steering/` as project memory
- Default files: `product.md`, `tech.md`, `structure.md`
- Custom files are supported (managed via `/kiro:steering-custom`)

## Weekly PDF Generation

### Single Source of Truth
- **Main file**: `weekly_pdf_generator.rb`
- **Entry point**: `generate_weekly_pdf.rb` (called by cron)
- **DO NOT create** multiple versions like `generate_last_week_*.rb`, `weekly_kindle_send_*.rb`
- Always improve the main file instead of creating variations

### PDF Format Specification
Based on `generate_last_week_final.rb` (reference format):

1. **Header Section**
   - Title: "WEEKLY BOOKMARKS DIGEST" (large, bold, centered)
   - Period: "Period: YYYY-MM-DD - YYYY-MM-DD"
   - Stats: "Total Items: N" and "With Summary: N/M"

2. **Weekly Insights Section** (if `overall_insights` exists)
   - Header: "WEEKLY INSIGHTS" (separator line above/below)
   - Content: Overall summary of the week's trends

3. **Peripheral Keywords Section** (if `related_clusters` exists)
   - Header: "PERIPHERAL KEYWORDS / RELATED TOPICS" (separator line above/below)
   - Format:
     ```
     • Topic Name
       Related: keyword1, keyword2, keyword3
     ```

4. **Table of Contents**
   - Header: "TABLE OF CONTENTS"
   - Format: "N. Title\n   Date: MM/DD"

5. **Bookmark Details** (one per page or section)
   - Format:
     ```
     [N] Title
     Date: YYYY-MM-DD
     Link: URL
     Tags: #tag1 #tag2

     Summary:
       - bullet point 1
       - bullet point 2
     ```
   - Use "[Summary not available]" if no content

6. **Footer**
   - Page numbers: "Page N / Total"

### Content Requirements
- All bookmarks must have summaries fetched via BookmarkContentFetcher
- Wait up to 30 minutes for Gatherly API to fetch missing content
- Display progress during content fetch
- Save summaries to database for future use

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/waterlilyllc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/waterlilyllc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
