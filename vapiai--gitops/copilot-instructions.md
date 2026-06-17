## changelog-updates

> Require changelog updates for significant config changes in customer deployments


# Changelog Update Requirement

This rule applies to **customer deployment clones** of the gitops repo, not the root template repo itself.

**How to tell the difference:** If `docs/changelog.md` still contains the placeholder section header `## YYYY-MM-DD`, this is the unmodified template and you should NOT update the changelog. Once that placeholder has been replaced with real dated entries, this is a customer deployment and the rule below applies.

When making significant configuration changes in a customer deployment, update `docs/changelog.md` in the same change.

Significant changes include:
- Assistant changes (prompt/body/frontmatter updates)
- Tool changes (new tools, parameter/behavior changes)
- Squad changes (members, handoffs, overrides, routing)
- Structured output or simulation changes that affect behavior

Changelog entries should include:
- Date section (`YYYY-MM-DD`)
- Resource type and file path(s)
- What changed, why it changed, and expected impact

---
> Source: [VapiAI/gitops](https://github.com/VapiAI/gitops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
