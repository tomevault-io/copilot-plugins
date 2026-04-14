## langai

> Tracks active ticket ID in tickets/current.md (local workspace state only, not committed)

# Current Ticket Rule

## Purpose
The `tickets/current.md` file identifies which ticket is currently being worked on, providing a simple way to track the active ticket context.

## File Location
- **Path**: `tickets/current.md`
- **Single file for entire workspace**
- **Git Status**: Ignored (local workspace state only)

## Content Structure
- **Content**: Just the ticket number
- **Format**: Plain text, no markdown formatting needed
- **Example**: `EBASE-11662`

## Usage
- Updated when switching between tickets
- Helps tools and local workspace understand current focus
- Simple reference for identifying active work context
- Local to each developer's workspace (not shared via git)

## Update Frequency
- When starting work on a different ticket
- When switching context between multiple active tickets
- Should always reflect the primary ticket being worked on

## Agent Behavior
- The AI should check this file to understand the current active ticket
- When switching tickets, update this file with the new ticket number
- Reference this file when organizing work context
- Note: This file is not committed to git (workspace-specific state)

## FINAL MUST-PASS CHECKLIST

- [ ] File exists at `tickets/current.md`
- [ ] Contains ONLY the active ticket ID
- [ ] Updated immediately upon starting/switching work
- [ ] Not committed to version control

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ai-Whisperers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
