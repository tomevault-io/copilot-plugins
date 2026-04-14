## retro-team-workflow

> start us response with star enoji.


start us response with star enoji.

RULS
Code Structure & Readability
• Modular design (e.g., SystemInfo, DebateManager, AIResponder).
• Follow PEP 8; use clear, descriptive names.

System Information Module
• Runs first; prints OS, CPU, RAM, Python version, etc.
• Uses rich with headers / bullet lists / tables.

Debate Engine Behavior
• Accepts a topic.
• Spawns four personas with unique stances.
• Assigns each persona a fixed rich color.
• Round-robin replies; 3-5 crisp sentences each; round number + AI name prefixed.

Automatic Project Context Loading
• On every fresh chat, scan docs/ and ingest all files for contextual grounding.

User Interaction
• Simple CLI prompts; validate topic and round count.
• Option to restart or end after debate wraps.

Error Handling & Logging
• Graceful catches for missing modules, bad input, file I/O, etc.
• Log key steps for debugging.

Visual Output with rich
• Panels / columns / tables for AI replies.
• Consistent formatting and color scheme.

Extensibility
• Plug-and-play new personas, scoring, user voting, rebuttal phases.

Change Tracking & Rule Maintenance ⬅️ NEW
• After every major feature addition or structural overhaul, the agent must automatically append, revise, or prune this rule set so it always mirrors reality.
• Commit the updated rule list to the project’s config or docs repository immediately after the change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MotiTheWizerd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
