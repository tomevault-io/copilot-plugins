## wannabepolymath

> This rule defines how to use MkDocs for documentation in this project, including standards, best practices, and optimization strategies to ensure a maintainable, user-friendly, and scalable documentation site.

# Cursor Rule: mkdocs.mdc

## Purpose
This rule defines how to use MkDocs for documentation in this project, including standards, best practices, and optimization strategies to ensure a maintainable, user-friendly, and scalable documentation site.

---

## 1. General Guidelines
- **Use MkDocs with the Material theme** for a modern, responsive, and feature-rich documentation experience.
- **All documentation content must be written in Markdown (.md)** for simplicity and compatibility.
- **Organize documentation in the `docs/` directory** following the agreed folder structure (see `folder-structure.mdc`).
- **Keep navigation intuitive and shallow**; avoid deep nesting to ensure users can find content easily.

## 2. Configuration Standards
- **`mkdocs.yml` is the single source of truth** for site configuration, navigation, and theme settings.
- **Enable search, navigation tabs, and section expansion** for better user experience.
- **Use site metadata** (title, description, author, repo_url) for discoverability and professionalism.
- **Configure extra features** (e.g., code highlighting, math support, diagrams) as needed for the project.

## 3. Content Best Practices
- **Use frontmatter in Markdown files** for metadata (title, description, tags, etc.).
- **Write clear, concise, and well-structured documentation**; use headings, lists, and tables for readability.
- **Add a table of contents to long documents** for easier navigation.
- **Cross-link related documents** using relative links.
- **Use images, diagrams, and code blocks** to enhance understanding.
- **Keep file and folder names lowercase and hyphenated** for consistency.
- **Create index files (`index.md` or `_index.md`)** for each major section.

## 4. Navigation & Structure
- **Define navigation in `mkdocs.yml`** to match the logical structure of the docs.
- **Group related content under clear sections** (e.g., Blog, Topics, Deep Dives, About).
- **Avoid more than 3 levels of nesting** in navigation.
- **Update navigation promptly** when adding, removing, or reorganizing content.

## 5. Optimization & Maintenance
- **Regularly review and update documentation** to keep it accurate and relevant.
- **Use version control (Git) for all documentation changes**.
- **Leverage MkDocs plugins** (e.g., for search, tags, diagrams) to enhance functionality.
- **Test the site locally before deploying** to catch formatting or navigation issues.
- **Optimize images and assets** for fast loading.
- **Monitor site performance and user feedback** to guide improvements.

## 6. Accessibility & UX
- **Ensure all content is accessible** (use alt text for images, semantic headings, etc.).
- **Support both light and dark modes** for user preference.
- **Use clear, readable fonts and sufficient contrast**.

## 7. References
- @MkDocs Documentation
- @Material for MkDocs
- @Best Practices for Technical Writing

---

**This rule must be reviewed and updated as the project evolves or as new MkDocs features and best practices emerge.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sh4peSh1fter)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sh4peSh1fter)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
