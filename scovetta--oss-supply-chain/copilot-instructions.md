## oss-supply-chain

> Provides foundations for understanding supply chain security: how software is built today, the threat landscape, historical attacks, and detailed attack patterns.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of **three book manuscripts** on software supply chain security, organized as markdown files. The series covers understanding threats, implementing defenses, and governing supply chain security at an organizational level.

## Structure

The repository is organized into three books, each with its own directory:

```
contents/
  book1/            Understanding the Software Supply Chain (~430 pages)
    chapter-01/     Chapters 1-10 (Parts I-II)
    ...
    chapter-10/

  book2/            Protecting the Software Supply Chain (~505 pages)
    chapter-11/     Chapters 11-22 (Parts III-V partial)
    ...
    chapter-22/

  book3/            Governing the Software Supply Chain (~455 pages)
    chapter-23/     Chapters 23-33 (Parts V-VII)
    ...
    chapter-33/

  appendix/         Appendices
  frontmatter/      Frontmatter (author, legal, etc. pages)
```

Each chapter directory contains section files named `ch-X.Y.md` where X is the chapter number and Y is the section number.

## Book Summaries

### Book 1: Understanding the Software Supply Chain
Provides foundations for understanding supply chain security: how software is built today, the threat landscape, historical attacks, and detailed attack patterns.

**Part I - Foundations (Ch 1-4)**: How software is built today, supply chain threats, historical attacks, ecosystem-specific risks

**Part II - Attack Patterns (Ch 5-10)**: Malicious packages, dependency confusion, typosquatting, build system attacks, insider threats, social engineering

**Appendices**: A (Glossary), B (Resources), F (Incident Timeline)

### Book 2: Protecting the Software Supply Chain
Translates threat knowledge into practical defenses across the development lifecycle, from dependency selection through production deployment.

**Part III - Risk Assessment & Testing (Ch 11-15)**: Risk measurement, SBOMs, dependency management, security testing, red teaming

**Part IV - Defense & Response (Ch 16-20)**: Securing development environments, CI/CD pipelines, distribution, incident response, crisis communication

**Part V - Operationalizing Defense (Ch 21-22)**: Building security programs, platform engineering

**Appendices**: A (Glossary), B (Resources), C (SBOM/AI-BOM Guide), D (Security Checklist), G (Ecosystem Guides)

### Book 3: Governing the Software Supply Chain
Addresses human, policy, and strategic dimensions: organizational commitment, regulatory compliance, economic incentives, and industry collaboration.

**Part V - People & Organizations (Ch 23-25)**: Training and security culture, open source maintainers, vendor risk management

**Part VI - Regulatory & Legal (Ch 26-29)**: Regulatory landscape (EO 14028, EU CRA), compliance frameworks, legal considerations, industry initiatives

**Part VII - Context & Future (Ch 30-33)**: Economics, geopolitics, lessons from other industries, future directions

**Appendices**: A (Glossary), B (Resources), E (Sample Policies/Templates), H (Compliance Mapping)

## Common Workflows

### Reviewing Content

When asked to review or analyze chapters:
1. Identify which book contains the chapter (1-10 = book1, 11-22 = book2, 23-33 = book3)
2. Navigate to the appropriate book and chapter directory
3. Read the relevant `ch-X.Y.md` files
4. Check the book's README.md for chapter/section structure overview
5. Cross-reference TODO.md for known issues

### Finding Specific Topics

Use grep to search across all books:
```bash
grep -r "search term" book*/
```

Or use the Grep tool with appropriate patterns and glob filters.

### Building PDFs

Each book has its own `build-pdf.sh` script. There is also a root `build-all.sh` script.

### Working with TODO Items

The root `TODO.md` tracks outstanding issues and tasks. When making significant changes or identifying problems, update this file.

## Citations and References

Per TODO.md, this book requires:
- Proper citations for all factual claims
- Attribution for all quotes (check for hallucinated quotes)
- References to recent industry reports and research

## Content Standards

When editing content:
- Maintain technical accuracy on security topics
- Avoid repeating use cases across chapters (per TODO.md)
- Ensure consistency in terminology (refer to Appendix A: Glossary)
- Cross-reference related sections appropriately
- Keep section numbering consistent (ch-X.Y format)

**Voice:** Use "we" for recommendations ("We recommend..."), "you" for direct guidance. Professional but accessible tone—authoritative without being academic.

**Structure:**
- Open with context establishing relevance
- Define key terms in **bold** when introduced
- Use bullet points for lists, numbered lists for sequences
- Include real-world examples with specific details (names, dates, impacts)
- Do not hallucinate or refer to events that didn't actually occur
- Include concrete, actionable recommendations
- Prefer to have only one main idea per paragraph

**Evidence:**
- All data sources should be cited using Markdown reference links and must be verified
- Include statistics and research findings
- Reference specific tools/frameworks by name and add a reference link to its URL if possible
- When citing incidents: what happened, when, who was affected, lessons learned

**Formatting:**
- ### for main headings, #### for subsections
- **Bold** for key terms, `code format` for package/command names
- > for block quotes (practitioner quotes, notable findings)
- Include hyperlinks to authoritative sources

## Special Considerations

- **No build system**: This is a markdown-based manuscript; PDFs are built via shell scripts
- **Images**: Include all images in an "img/" folder alongside the Markdown file
- **Git workflow**: Standard git operations for version control
- **Focus on substance**: When reviewing, prioritize factual accuracy, completeness, and logical flow
- **Shared appendices**: Appendices A and B appear in all three books; others are book-specific

---
> Source: [scovetta/oss-supply-chain](https://github.com/scovetta/oss-supply-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
