## sales-skills

> **Sales Skills** is an open-source collection of 21 AI agent skills covering the full B2B sales lifecycle. Built by [Craig Hewitt](https://agentprime.io), founder of Castos ($1.5M ARR) and AgentPrime.

# CLAUDE.md -- Sales Skills

## Repository Overview

**Sales Skills** is an open-source collection of 21 AI agent skills covering the full B2B sales lifecycle. Built by [Craig Hewitt](https://agentprime.io), founder of Castos ($1.5M ARR) and AgentPrime.

- **GitHub:** https://github.com/thecraighewitt/sales-skills
- **License:** MIT
- **Spec:** [Agent Skills Specification](https://agentskills.io/specification.md)

## Philosophy

These skills are opinionated by design. They encode real practitioner experience -- not sales textbook theory or "it depends" hedging.

- **Push back on weak inputs.** If a user gives vague ICP definitions ("we sell to everyone"), lazy value props ("we help companies grow"), or unsubstantiated claims ("we're the best"), challenge them. Good output requires good input.
- **Frameworks over theory.** Every skill gives users a structure they can follow immediately. "Use MEDDIC for enterprise discovery" is more useful than a paragraph about why qualification matters.
- **Direct and opinionated.** "Do this, not that" beats "consider your options." Take a stance. If something doesn't work in practice, say so.
- **Examples over explanations.** Show the cold email. Show the discovery question. Show the objection response. Don't describe what one should look like.
- **Channel-specific.** Each outreach channel has its own skill with channel-specific best practices. Don't bleed LinkedIn advice into the cold email skill or vice versa.

## Repository Structure

```
sales-skills/
├── .claude-plugin/marketplace.json   # Claude Code plugin manifest
├── skills/                           # All 21 skills
│   └── skill-name/SKILL.md          # One SKILL.md per directory
├── CLAUDE.md                         # This file (agent instructions)
├── CONTRIBUTING.md                   # How to contribute
├── VERSIONS.md                       # Version tracking
├── README.md                         # Public-facing docs
├── validate-skills.sh                # Frontmatter validation
└── LICENSE                           # MIT
```

## Skill Invocation

When a user's request matches a skill's trigger phrases (listed in each SKILL.md frontmatter `description` field), load and follow that skill's SKILL.md instructions.

- **Match broadly.** If a user says "help me write an outreach email," that's `cold-email`. If they say "prep me for a call with a prospect," that's `discovery-call`. You don't need an exact trigger phrase match -- use judgment.
- **Load the full SKILL.md.** Don't summarize or paraphrase the skill. Read and follow its instructions, frameworks, and output formats as written.
- **Check for sales context first.** Every non-foundation skill expects `.agents/sales-context.md` to exist. If it does, load it. If it doesn't, see the error handling section below.

## Multi-Skill Workflows

Skills cross-reference each other. When a skill references another skill in its "Related Skills" section or body:

- **Suggest, don't auto-chain.** After completing a skill, tell the user which related skill to run next and why. Example: "Your buyer personas are done. Run `competitive-intel` next to build battle cards against the competitors you mentioned."
- **Don't auto-run the next skill** unless the user explicitly asks for a multi-skill workflow (e.g., "set up my full sales foundation" or "build my complete outbound sequence").
- **Respect the dependency graph.** Foundation skills feed everything. Prospecting skills feed deal execution. Deal intelligence feeds back into foundation. Follow the architecture:

```
sales-context → all skills
buyer-persona → discovery-call, demo-script, proposal-pricing
competitive-intel → objection-handling, demo-script, negotiation
lead-research → cold-email, cold-call, linkedin-outreach, direct-mail
outbound-sequence → orchestrates all channel skills
call-debrief → pipeline-review, win-loss-analysis
win-loss-analysis → competitive-intel, buyer-persona (feedback loop)
```

## Error Handling

### Missing Sales Context

If a user tries to run any non-foundation skill (anything except `sales-context`, `buyer-persona`, or `competitive-intel`) and `.agents/sales-context.md` does not exist:

1. Tell the user: "This skill works best with your sales context set up first. Run `sales-context` to define your ICP, value prop, and sales motion -- it takes about 5 minutes and every other skill reads it automatically."
2. Offer a choice: run `sales-context` now, or proceed without it (the skill will ask basic context questions inline, but the output won't be as tailored).
3. Do not silently skip the context check or pretend it doesn't matter.

### Vague or Incomplete Input

- If the user's input is too vague to produce good output, ask clarifying questions before generating. Don't guess.
- Each SKILL.md has a "Before Starting" or "Context Questions" section listing what to ask. Follow it.
- Two rounds of clarification max. If you still don't have enough, generate with what you have and flag the gaps.

## Output Conventions

- **Format:** Markdown by default. When a skill produces email copy, call scripts, or other content meant to be used directly, output it in a fenced code block so users can copy it cleanly.
- **Location:** Do not write output files unless the user asks. The exception is `sales-context`, which writes to `.agents/sales-context.md` by design.
- **Length:** Match the skill's guidance. Cold emails should be under 100 words. Discovery call plans can be longer. Don't pad output to seem comprehensive.
- **Tone:** Match the skill's persona. Each SKILL.md opens with an expert persona declaration -- adopt that voice for the duration of the skill.

## Skill Categories

### Foundation (3)
- `sales-context` -- Sales context doc (ICP, value prop, motion, proof points). Read by every other skill.
- `buyer-persona` -- Deep buyer personas, buying committee mapping, champion/blocker profiles.
- `competitive-intel` -- Battle cards, positioning, trap-setting questions, win/loss by competitor.

### Prospecting & Outbound (8)
- `cold-email` -- B2B cold emails that get replies.
- `cold-call` -- Cold call scripts, openers, voicemail, objection handling on the phone.
- `linkedin-outreach` -- Connection requests, InMails, DM sequences, social selling.
- `direct-mail` -- Dimensional mailers, handwritten notes, gift sequences.
- `lead-research` -- Account/contact research, trigger events, intent signals, target lists.
- `outbound-sequence` -- Multi-channel sequence design (orchestrates channel skills).
- `referral-intro` -- Warm intro requests, mutual connections, partner referrals.
- `event-networking` -- Conference outreach, pre/during/post-event sequences.

### Deal Execution (3)
- `discovery-call` -- Discovery call planning, question frameworks, qualification.
- `demo-script` -- Demo scripts by persona and stage, story arcs, competitive positioning.
- `objection-handling` -- Objection playbooks by category, response frameworks.

### Deal Close (2)
- `proposal-pricing` -- Proposals, SOWs, pricing, packaging, ROI justification.
- `negotiation` -- BATNA, concession planning, deal structure, multi-stakeholder dynamics.

### Deal Intelligence (2)
- `call-debrief` -- Post-call analysis, next steps, risk signals, coaching notes.
- `win-loss-analysis` -- Pattern recognition across deals, competitive insights, process improvement.

### Sales Ops (3)
- `pipeline-review` -- Deal scoring, risk flagging, stage-appropriate questions.
- `forecast` -- Weighted pipeline, conversion rates, commit/upside/best-case.
- `sales-comp` -- OTE structures, quota setting, accelerators, SPIFs, plan modeling.

## Build / Lint / Test Commands

```bash
# Validate all skills have correct frontmatter
./validate-skills.sh
```

## Agent Skills Specification

All skills follow the [Agent Skills spec](https://agentskills.io/specification.md):

### Frontmatter Rules
- `name`: Required. Max 64 chars. Lowercase letters, numbers, hyphens only. Must match directory name.
- `description`: Required. Max 1024 chars. Include trigger phrases and cross-references.
- `metadata.version`: Semver format.

### Skill Body
- Expert persona declaration
- Context check (`.agents/sales-context.md`)
- Core principles (3-5, opinionated)
- Frameworks and process steps
- Examples and templates
- Related skills cross-references
- Keep under 500 lines

## Writing Style Guidelines

- **Practitioner voice.** Write from experience, not textbooks.
- **Direct and opinionated.** "Do this, not that" beats "consider your options."
- **Frameworks over theory.** Give people structures they can follow immediately.
- **Examples over explanations.** Show the cold email, don't describe what a cold email should feel like.
- **Channel-specific.** Each outreach channel has its own skill -- don't bleed advice across channels.
- **Short sentences.** Sales teams skim. Write accordingly.

## Key Cross-References

- `sales-context` is read by every other skill -- it's the foundation.
- `outbound-sequence` orchestrates `cold-email`, `cold-call`, `linkedin-outreach`, `direct-mail`.
- `lead-research` feeds all prospecting channel skills.
- `buyer-persona` informs `discovery-call`, `demo-script`, `proposal-pricing`.
- `competitive-intel` informs `objection-handling`, `demo-script`, `negotiation`.
- `win-loss-analysis` feeds back into `competitive-intel` and `buyer-persona`.
- `call-debrief` feeds `pipeline-review` and `win-loss-analysis`.

## Git Workflow

- Branch naming: `feature/skill-name` or `fix/skill-name-description`
- Conventional commits: `feat(skill-name):`, `fix(skill-name):`, `docs:`, `chore:`
- Run `./validate-skills.sh` before committing

---
> Source: [TheCraigHewitt/sales-skills](https://github.com/TheCraigHewitt/sales-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
