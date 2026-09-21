## rules-for-docs

> Documentation rules for RepoMap-Tool with reasoning protocols


# 🤖 **AI REASONING PROTOCOL**

## **Before Taking Any Documentation Action:**
**ALWAYS provide step-by-step reasoning that:**
1. States what documentation action you're about to take
2. Explains which rules from this document apply
3. Shows how you've considered those rules in your approach
4. Outlines your specific plan and next steps

---

# 📚 Documentation Rules for RepoMap-Tool

## General Documentation Principles
- All new features, modules, and workflows **must** be documented in the `docs/` directory.
- Documentation should be clear, concise, and actionable for both users and developers.
- Every major architectural or workflow change requires an update to the relevant `actionplans/` or roadmap files.

## Structure
- **User Guides:** The `docs/` directory is for **user-facing documentation** - guides that help users understand how to use the tool and its features.
- **Roadmaps:** High-level plans and phase breakdowns go in `docs/actionplans/`.
- **Module Docs:** Each major module (e.g., `trees`, `dependencies`, `llm`, `context`) must have a corresponding markdown file describing its purpose, core classes, and usage.
- **API/CLI Docs:** All user-facing commands and APIs must be documented with usage examples.
- **Future Plans:** Optimizations and enhancements should be tracked in `future-optimizations.md`.

**Important:** The `docs/` folder is NOT for tracking feature completion status or implementation details. It's for user guides and documentation that helps users understand how to use the tool.

## Documentation Workflow
1. **Before Implementation:** Update or create an action plan for the new feature or change.
2. **During Implementation:** Maintain up-to-date module-level docs as code evolves.
3. **After Implementation:** Ensure all new/changed functionality is reflected in the docs, including edge cases and limitations.
4. **Testing Docs:** If new test strategies or coverage areas are introduced, document them in a `testing.md` or relevant section.

## Review & Maintenance
- Documentation changes must be reviewed alongside code in PRs.
- Outdated docs should be updated or removed as part of refactoring.
- All documentation must be kept in sync with the current codebase.

## Style
- Use clear section headings, bullet points, and code blocks for clarity.
- Prefer practical examples over abstract descriptions.
- Use consistent terminology matching the codebase.

## Minimum Requirements
- No new feature or module is considered "done" until its documentation is complete and reviewed.
- All public APIs, CLI commands, and configuration options must be documented.

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
