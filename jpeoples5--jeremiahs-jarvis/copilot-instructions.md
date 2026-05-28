## jeremiahs-jarvis

> This project uses a skills system. Skills live in `skills/<name>/[skill-name]-skill.md`. When the user invokes a skill (e.g., `/jarvis`, `/research`), load and follow the corresponding skill file exactly.

# Jeremiah's Jarvis — Claude Instructions

## Skills
This project uses a skills system. Skills live in `skills/<name>/[skill-name]-skill.md`. When the user invokes a skill (e.g., `/jarvis`, `/research`), load and follow the corresponding skill file exactly.

## Agents
Agents live in `agents/<name>/[agent-name]-agent.md`. Load and follow the agent file when invoked.

## Available Skills
- `/onboard` — First-time setup wizard. Builds your voice profile from sample content.
- `/research` — Multi-agent research dossier saved to `references/`.
- `/script-writer` — Drafts a script in your voice using your profile.
- `/table-this` — Turns a script into a side-by-side visual breakdown table.
- `/push-to-notion` — Publishes a script to Notion with status tracking.
- `/on-screen-title` — Curiosity-gap titles for short-form video.
- `/jarvis` — On-demand orchestrator. Runs the full pipeline end-to-end.
- `/jarvis-overnight` — Scheduled overnight orchestrator for batch production.

## Available Agents
- `angle-finder` — Non-obvious angles other creators aren't covering.
- `hook-writer` — Opening hooks before script writing.
- `title-generator` — Bulk titles across longform, short-form, blog, social.
- `fact-checker` — Verifies every specific claim against sources.
- `script-pacing-auditor` — Maps drop-off risk by timestamp.
- `model-auditor` — Adversarial audit of Haiku/Sonnet/Opus assignments.
- `ping-pong` — Free-form ideation partner. No deliverables.
- `jarvis` — Orchestration brain called by `/jarvis`.

## Skill Trigger Phrases
Even without a slash command, activate the matching skill when the user says:
- "onboard", "setup", "first time setup", "configure jarvis" → `/onboard`
- "research [topic]", "research this", "look into", "what are people saying" → `/research`
- "write a script", "draft a script", "write me a script" → `/script-writer`
- "table this", "make a table", "add visuals", "visual breakdown" → `/table-this`
- "push to notion", "send to notion", "publish to notion" → `/push-to-notion`
- "on-screen title", "title for this short" → `/on-screen-title`
- "jarvis go", "jarvis go on [topic]", "run jarvis" → `/jarvis`
- "run overnight", "overnight scripts" → `/jarvis-overnight`
- "find angles", "angle finder" → `angle-finder`
- "hook options", "hook writer" → `hook-writer`
- "title options", "generate titles" → `title-generator`
- "fact check this", "verify this" → `fact-checker`
- "pacing audit", "where will people drop off" → `script-pacing-auditor`
- "model audit", "should this be haiku or sonnet" → `model-auditor`
- "ping pong", "let's riff", "think through this with me" → `ping-pong`

## Memory
Do not write to memory files. This system uses skill files and `references/creator-profile.md` as the source of truth for persistent behavior. If something needs to be remembered, edit the relevant skill or update the creator profile directly.

## Voice Profile (Source of Truth)
All voice-dependent skills and agents read `references/creator-profile.md` before producing output. If that file does not exist, run `/onboard` first. Do not invent a voice. The profile IS the voice.

## File Naming
- Use descriptive, slug-based filenames everywhere
- Scripts: `[slug]-script.md`, never `script.md`
- Research dossiers: `[slug]-research-[YYYY-MM-DD].md` saved to `references/`
- Visual tables: appended to existing script files, never separate

## Writing Rules (apply everywhere)
- Never use em dashes. Use commas, periods, or split sentences.
- Never use the phrase "full stop".
- Never use the "It's not X, it's Y" contrast pattern. Write a direct statement instead.
- Plus any phrases listed in the user's `references/creator-profile.md` under "Words/phrases never used."

## Session Setup
At the start of each session, check if any `.md` files exist in `session-state/`. If yes, read the most recently modified one and summarize its "Next Session Priorities" section before asking what to work on.

If `references/creator-profile.md` does not exist, prompt the user to run `/onboard` before doing any production work.

## General Rules
- Never force push unless explicitly told
- Never commit `.env` files or secrets
- Follow each skill's Critical Rules section exactly
- When responding as an agent, preface with `[Agent Name]:` on its own line before the response

---
> Source: [JPeoples5/jeremiahs-jarvis](https://github.com/JPeoples5/jeremiahs-jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
