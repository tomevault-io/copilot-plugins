## docs-edenia-com

> This is a bilingual (English/Spanish) Docusaurus documentation site for Sistemas Edenia, focusing on EVM-compatible blockchain development, Web3, and AI development for software developers. The project serves as a comprehensive developer hub with technical guides, tutorials, and best practices. Legacy EOSIO/Antelope content is maintained in less visible subcategories.

# Cursor Rules for Sstemas Edenia Documentation Project

## Project Context
This is a bilingual (English/Spanish) Docusaurus documentation site for Sistemas Edenia, focusing on EVM-compatible blockchain development, Web3, and AI development for software developers. The project serves as a comprehensive developer hub with technical guides, tutorials, and best practices. Legacy EOSIO/Antelope content is maintained in less visible subcategories.

---

## Modular Rules Structure

**All detailed rules are in `.cursor/rules/` directory** (automatically loaded by Cursor):

- **01-documentation-standards.md** - File structure, frontmatter, writing, code examples, i18n
- **02-technical-standards.md** - Docusaurus, SEO, metadata, performance, accessibility
- **03-content-guidelines.md** - Web3/blockchain, AI tools, tutorials (EVM-focused)
- **04-brand-style-guidelines.md** - Edenia branding, language, tone
- **05-quality-assurance.md** - Publishing checklist, content review
- **06-ai-assistant-guidelines.md** - Content generation, refactoring guidelines
- **07-special-considerations.md** - Costa Rica context, open source, accuracy

---

## Core Principles

### Audience
- **Primary**: Software Developers
- **Focus**: Practical, implementable technical guidance

### Languages
- **English** (primary) & **Spanish** (must use formal "usted" form, never "tú")
- Maintain parallel structure: `docs/` (EN) & `i18n/es/` (ES)
- Cultural context: Costa Rica / Latin America

### Content Focus
- **Primary**: EVM-compatible blockchain (Ethereum, Polygon, BSC)
  - Solidity development & smart contracts
  - Web3.js, Ethers.js, MetaMask integration
  - DeFi, NFTs, dApp development
- **Secondary**: AI Development Tools, Web3, DevOps
- **Legacy**: EOSIO/Antelope (maintained in less visible subcategories)

### Documentation Standards
- **Structure**: Use kebab-case file names, proper frontmatter (id, title, description, keywords)
- **Writing**: Clear, professional, active voice, present tense
- **Code**: Syntax highlighting, comments, runnable examples, error handling
- **i18n**: Identical structure, accurate translation, consistent technical terms

### Brand Guidelines
- **Company**: "Sistemas Edenia" (full) / "Edenia" (informal)
- **Tone**: Professional, technical, accessible
- **Voice**: "we" for Edenia team, inclusive language

### Technical Stack
- **Platform**: Docusaurus v3
- **Content**: Markdown/MDX with Docusaurus components
- **Code Style**: 2 spaces (JS/TS), 4 spaces (others)
- **SEO**: Meta descriptions, keywords, Open Graph, alt text

### Quality Standards
- Test code examples, verify links, check both languages
- Proper heading hierarchy (H1→H2→H3)
- Accessibility (alt text, semantic HTML, ARIA labels)
- Security best practices, version numbers, troubleshooting sections

---

> **Note:** For detailed guidelines on any topic, refer to the corresponding file in `.cursor/rules/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edenia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
