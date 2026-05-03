## fivfold-skills-maintenance

> Mandatory skills.md maintenance when FivFold components or kits change


# Skills Registry Maintenance

When you add, remove, rename, or significantly change:

- **Kits** (UI or API manifests)
- **Skill bundles** (ui/skills/)
- **Component structure** (new exports, new paths in templates)

you **MUST** update [skills.md](../../skills.md) at the repo root.

Update the relevant table (FivFold Project Skills, Developer Project Skills, or Available Kits) and ensure the `npx @fivfold/ui skills` CLI can discover the new or changed skill.

See [AGENTS.md](../../AGENTS.md) Section 11 for the full rule.

---
> Source: [Fivex-Labs/fivfold](https://github.com/Fivex-Labs/fivfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
