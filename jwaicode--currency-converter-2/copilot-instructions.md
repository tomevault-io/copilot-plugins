## currency-converter-2

> description: Pre-commit checklist invoked automatically before code commits.

rule_type: agentRequested
name: selfcheck
description: Pre-commit checklist invoked automatically before code commits.

Purpose: Final checklist before committing any change to the project.

Pre-Commit Checklist:

Am I only making changes that were explicitly approved?

Did I reflect using reflection.mdc before this?

Are the changes modular, minimal, and clear?

Does this affect any project design docs? If so, has the user approved and confirmed updates to them?

Have I avoided hallucinations or unrelated additions?

Did I check if this needs logging in workhistory.mdc?

If the change is a cursor self-improvement, did I log it to cursor_upgrade_history.mdc instead?

Does any new or updated memory conform to the Memory Guidelines?

If I modified dependencies or versions, have I verified compatibility and documented the reasoning?

Will this commit keep history clean and follow git-automation.mdc?

If any item fails, re-evaluate or discuss with the user before proceeding.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jwaicode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
