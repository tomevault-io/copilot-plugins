## midas

> Post progress to Midas Squad Chat during your work. Use this command from the worktree:


## Squad Chat (mandatory)
Post progress to Midas Squad Chat during your work. Use this command from the worktree:
  bash /Users/openclaw_agent_1/workspace/midas/scripts/squad-post.sh "CARD-079" "your update" "YourAgentName"
Replace CARD-XXX with the actual card ID. Post when: (1) you start, (2) major milestone, (3) you complete, (4) you hit a blocker.
Keep messages to 1-2 sentences. Do not over-post.

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.

## Skill routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.
The skill has specialized workflows that produce better results than ad-hoc answers.

Key routing rules:
- Product ideas, "is this worth building", brainstorming -> invoke office-hours
- Bugs, errors, "why is this broken", 500 errors -> invoke investigate
- Ship, deploy, push, create PR -> invoke ship
- QA, test the site, find bugs -> invoke qa
- Code review, check my diff -> invoke review
- Update docs after shipping -> invoke document-release
- Weekly retro -> invoke retro
- Design system, brand -> invoke design-consultation
- Visual audit, design polish -> invoke design-review
- Architecture review -> invoke plan-eng-review
- Save progress, checkpoint, resume -> invoke checkpoint
- Code quality, health check -> invoke health

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmaulana0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
