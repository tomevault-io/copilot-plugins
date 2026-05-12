## biological-twin

> A biological twin that explains you to yourself. Upload DNA + bloodwork → your twin reads your genome thematically, finds patterns across hundreds of variants, and connects your biology to your lived experience. Not health analysis — self-knowledge through biology. Built for "Built with Opus 4.6: a Claude Code Hackathon" (Feb 10-16, 2026).

# Biological Twin — Opus 4.6 Hackathon

## Project
A biological twin that explains you to yourself. Upload DNA + bloodwork → your twin reads your genome thematically, finds patterns across hundreds of variants, and connects your biology to your lived experience. Not health analysis — self-knowledge through biology. Built for "Built with Opus 4.6: a Claude Code Hackathon" (Feb 10-16, 2026).

**Read `VISION.md` for the full philosophical frame. Read `spec/PRODUCT-SPEC.md` for features and scope.**

## Team
- **b** — Designer, builder, project lead
- **Claude** — Co-founder, architect, implementation partner

## Conventions
- b uses `//B:` for inline feedback, Claude uses `//C:` for replies
- Plans and decisions live in markdown files, not in chat
- All code is new — no pre-existing code from any prior project
- We track progress in `PROGRESS.md` with daily standups
- Design inspo goes in `design/inspo/` (screenshots, links, references)
- Free-form thoughts go in `NOTES.md`
- Process observations for submission go in `meta/PROCESS-LOG.md`

## Project Structure
```
├── CLAUDE.md                ← You are here
├── VISION.md                ← The soul (read first for WHY)
├── TICKETS.md               ← Active work tracker (start here for WHAT to build)
├── PROGRESS.md              ← Daily log (what shipped)
├── MASTER-PLAN.md           ← Week timeline
│
├── app/                     ← APPLICATION CODE
│   ├── src/app/page.tsx     ← Main page (landing → parsing → reading)
│   ├── src/app/api/reading/ ← Opus SSE streaming route
│   ├── src/components/      ← UI components (scene/, ui/)
│   ├── src/hooks/           ← useGenomeReading SSE hook
│   ├── src/lib/             ← types, store, dna-parser
│   └── public/snp-reference/← SNP JSON files served to client
│
├── spec/                    ← Product specs
├── knowledge-base/          ← Research & reference docs
├── data/                    ← Sample genomes + SNP reference source
├── design/inspo/            ← b's mood board
└── meta/                    ← Process log, submission, tools
```

## Key Rules
1. **All new code.** Nothing from prior projects. The idea is ours; the implementation is fresh.
2. **Self-knowledge, not medical advice.** We're a mirror, not a doctor. Frame everything as understanding, not diagnosis.
3. **Validation over anxiety.** "You're not broken. You're running different code." Lead with recognition, not risk.
4. **Evidence under every claim.** Every theme requires 3+ convergent variants. Tiers visible. Contradictions shown.
5. **Privacy-first.** Full genome parsed client-side. Only relevant SNP subset sent to Opus API.
6. **Beautiful.** Something you'd share with family, friends, AND your doctor — for different reasons each time.
7. **Ship daily.** Something working every day. Perfect is the enemy of submitted.

## Living Docs (actively updated)
- `TICKETS.md` — **Start here.** Active work items. Update when completing work.
- `PROGRESS.md` — Daily standup log. What shipped, what's next, blockers.
- `MASTER-PLAN.md` — Week timeline and milestones
- `VISION.md` — The soul of the project (WHY we're building this)
- `spec/PRODUCT-SPEC.md` — MVP definition, user stories, scope
- `spec/DESIGN-SPEC.md` — Visual direction (evolving with b's feedback)
- `meta/PROCESS-LOG.md` — How we used Opus/Claude Code (for submission narrative)
- `meta/SUBMISSION.md` — Demo script, repo checklist, 200-word description

## Reference Docs (read for context, don't update)
- `knowledge-base/genetics/READING-PROMPT.md` — Twin's reading prompt (embedded in API route)
- `knowledge-base/apis/OPUS-INTEGRATION.md` — Opus 4.6 API patterns (comprehensive)
- `knowledge-base/apis/OPUS-API-PATTERNS.md` — Earlier API research (overlaps with above)
- `knowledge-base/genetics/GROUNDING-THE-READING.md` — Scientific grounding framework
- `knowledge-base/competitors/LANDSCAPE.md` — Competitive analysis
- `knowledge-base/privacy/ARCHITECTURE.md` — Privacy model
- `knowledge-base/libraries/*.md` — Library research (decisions made, stack installed)
- `spec/TECHNICAL-SPEC.md` — Architecture (partially stale, code is source of truth now)
- `spec/SETUP-AND-WORKFLOW.md` — Tech stack decisions (all decided)
- `spec/TWIST-IDEAS.md` — Brainstorm archive (twist decided)
- `meta/SKILLS-AND-TOOLS.md` — Library/tool tracking (partially stale)
- `HOW-WE-GOT-HERE.md` — Origin story for submission

## Workflow
1. Check `TICKETS.md` → pick next P0 ticket
2. Move ticket to "In Progress" in TICKETS.md
3. Read relevant spec + knowledge-base files for context
4. Build the thing
5. Move ticket to "Done" with date in TICKETS.md
6. Update `PROGRESS.md` with what was accomplished
7. Log noteworthy process observations in `meta/PROCESS-LOG.md`
8. Commit + push

## Hackathon Details
- **Deadline:** Monday, February 16, 3:00 PM EST (2:00 PM CST)
- **Submission:** 3-min demo video + GitHub repo + 200-word description
- **Judging:** Demo (30%), Impact (25%), Opus 4.6 Use (25%), Depth (20%)
- **Special prizes:** "Most Creative Opus 4.6 Exploration" ($5K), "Keep Thinking" ($5K)

---
> Source: [brb-dreaming/biological-twin](https://github.com/brb-dreaming/biological-twin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
