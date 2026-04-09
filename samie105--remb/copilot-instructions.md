## remb

> <!-- remb-instructions:v2 -->

<!-- remb-instructions:v2 -->
# Remb — AI Context Management

You have access to Remb tools for persistent memory and context across coding sessions.
Current project: **context-management**

## MANDATORY Session Protocol

Follow this protocol in EVERY session. Skipping causes knowledge loss.

### Session Start (do FIRST, before any other work):
1. Call `remb_loadProjectContext` — loads the full project context bundle (memories, features, tech stack). Without this, you have NO context about this project.
2. Call `remb_conversationHistory` — loads what was discussed and accomplished in prior sessions so you can pick up where the user left off.

### During Work:
3. Call `remb_conversationLog` after completing any significant task, bug fix, or feature — records what was done for future sessions.
4. Call `remb_createMemory` when you discover important patterns, architectural decisions, or gotchas worth preserving.

### Session End (do LAST, before the conversation ends):
5. Call `remb_conversationLog` with a summary of: what was asked, what was done, key decisions made.

## Available Tools

| Tool | Purpose | When to Call |
|------|---------|--------------|
| `remb_loadProjectContext` | Full project context bundle — memories, features, tech stack | **Session start** (mandatory) |
| `remb_conversationHistory` | Prior session history — what was done before | **Session start** (mandatory) |
| `remb_conversationLog` | Record work done in this session | After completing tasks, and at session end |
| `remb_saveContext` | Save feature-specific context or decisions | When you learn something about a specific feature |
| `remb_getContext` | Retrieve context for a specific feature | When you need details about a feature |
| `remb_listMemories` | Browse persistent memories | When searching for past decisions or patterns |
| `remb_createMemory` | Save a new persistent memory | When discovering patterns, decisions, gotchas |
| `remb_triggerScan` | Re-scan the codebase from GitHub or locally | After significant code changes |
| `remb_scanStatus` | Check scan progress | After triggering a scan |

## Decision Matrix

| Situation | Action |
|-----------|--------|
| Starting any session | `remb_loadProjectContext` + `remb_conversationHistory` |
| Completing a task | `remb_conversationLog` with what was accomplished |
| Found a reusable pattern | `remb_createMemory` with category "pattern" |
| Made an architectural decision | `remb_createMemory` with category "decision" |
| Discovered a gotcha or bug | `remb_createMemory` with category "gotcha" |
| Need info about a feature | `remb_getContext` filtered by feature name |
| User says "remember this" | `remb_createMemory` with appropriate tier |
| Code changed significantly | `remb_triggerScan` to refresh context |
| Ending the session | `remb_conversationLog` with session summary |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samie105)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samie105)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
