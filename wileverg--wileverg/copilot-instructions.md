## wileverg

> This file contains high-priority rules for the Antigravity agent in this repository.

# Antigravity Rules

This file contains high-priority rules for the Antigravity agent in this repository.

## Agent Skills Context

Before generating any profile content, **always read the relevant skill files** from `.agent/skills/`:

| Skill File | When to Use |
| :--- | :--- |
| `github_profile_expert.md` | Choosing stats cards, layouts, dynamic widgets |
| `tech_branding_expert.md` | Writing impact bullets (STAR/PAR), UVP formula |
| `narrative_storytelling.md` | Shaping the bio, career arc, tone |
| `ai_ready_seo.md` | JSON-LD block, `llms.txt` instructions, keywords |
| `trust_and_ux.md` | Responsive images, A11y alt-text, security rules |
| `markdown_expert.md` | GFM tables, Mermaid diagrams, Frontmatter |

For adapting a profile for a **new user**, always start from `templates/profile_template.md` and consult `templates/README.md` to select the appropriate layout style.

## Markdown Guidelines
- Always use blank lines between headers and content to ensure correct rendering.
- Keep bullet points concise and avoid long paragraphs within lists.
- For technical explanations, always provide a code example in the relevant syntax.
- If a project plan is requested, always generate an `implementation_plan.md` artifact first.

## Verification
- Before submitting any Markdown change, run the `markdownlint` tool.

## GitHub Profile Guidelines
- Ensure all SVG cards (stats, metrics) have a `theme` that matches the repository's aesthetic.
- Use `<p align="center">` or `div` tags only when necessary for centering layouts.
- Social links must use high-quality icons or clear text labels.
- Verify that placeholders like `YOUR_USERNAME` are never left in final output.

## Branding & Presentation
- **Impact First**: No bullet points describing experience should be purely task-based. Always aim for "Action + Result + Metric".
- **ATS Optimization**: Ensure that technical keywords are woven into descriptions, not just listed.
- **Narrative Consistency**: The Professional Summary must contain a clear Unique Value Proposition (UVP).
- **Quantification**: Use bolding for numbers and percentages to increase scannability for recruiters.

## Security & Privacy
- **Vault First**: Never commit API keys, secrets, or sensitive local paths. Always use a `.gitignore`.
- **Privacy Enforcement**: Scrub personal info from docs unless explicitly requested for a public profile.

## UX & Responsive Design
- **Mobile First**: All tables and images must be optimized for mobile viewports (avoid horizontal scroll).
- **A11y**: Every non-text element (images, diagrams) MUST have a descriptive `alt` attribute.

## Narrative Storytelling
- **The Hook**: The professional summary must start with a high-impact sentence that captures the "Unique Value Proposition" (UVP).
- **Journey Logic**: Every project mentioned should explain *Why* it was built, not just *What* it does.

---
> Source: [wileverg/wileverg](https://github.com/wileverg/wileverg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
