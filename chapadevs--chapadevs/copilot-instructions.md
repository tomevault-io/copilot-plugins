## project-context

> Chapadevs (https://chapadevs.com) is a platform that bridges non-technical clients with vetted developers.

# Chapadevs — Project Context & Agent Rules

## What is Chapadevs?
Chapadevs (https://chapadevs.com) is a platform that bridges non-technical clients with vetted developers.
The core value proposition: a client describes their messy idea in plain language → AI generates a live visual preview → a developer is assigned and builds from that preview → the project is managed end-to-end in a shared workspace.

## The Problem We Solve
Non-technical clients struggle to communicate their ideas to developers. This leads to:
- Misaligned builds (devs building the wrong thing)
- No visibility into progress
- Wasted time and money on failed projects

Chapadevs solves this by acting as the translation layer between idea and execution.

---

## Core Features

### 1. AI-Generated Previews (KEY DIFFERENTIATOR)
- Client writes a plain-language description of their project
- AI generates a live visual/website preview
- Developer receives the preview + code as a starting blueprint
- Located under: Workspace > Previews

### 2. Shared Workspace
Each project has a dedicated workspace with:
- Overview — project summary, status (Open/In Progress/Completed), dates, assigned programmer, client info, "I'm Ready" / "Start Development" CTAs
- Previews — AI generation input + list of generated previews with copy code / download zip options
- Timeline — project milestones and schedule
- Team — client and assigned programmer(s)
- Assets — project files and resources
- Calendar — scheduling
- Activity — project activity log
- Chat — direct messaging between client and developer

### 3. Developer Assignment
- Developers are vetted and matched to project type
- "Ready confirmed" system (e.g. "1 of 2") tracks when both parties confirm readiness before development starts

### 4. Project Status Flow
OPEN → (both parties confirm ready) → START DEVELOPMENT → IN PROGRESS → COMPLETED

---

## Tech & UI Conventions

### Design Language
- Primary color: #1D9E75 (teal/green)
- Font style: clean, modern, minimal
- Status badges: uppercase, outlined (e.g. OPEN, COMPLETED)
- Action buttons: uppercase, filled teal for primary, outlined for secondary
- Danger actions (e.g. Leave Project) use red background

### Key UI Patterns
- Sidebar navigation: Overview, Workspace (Previews, Timeline, Team, Assets, Calendar), Activity, Chat
- Project header: title + status badge + primary CTAs
- Project summary card: Start Date, Due Date, Completed Date, Duration, Project Type, Client, Assigned Programmer(s), Ready Confirmed
- Danger Zone section at the bottom of Overview for destructive actions

### Project Types (examples)
- Management Panel / ERP / CRM
- Websites / Landing Pages
- Web Apps

---

## Tone & Messaging Guidelines
- Speak directly to non-technical clients — no jargon
- Lead with the problem (messy ideas, miscommunication, wasted money)
- The AI preview is the hero feature — always highlight it
- Positioning: "You describe it, we visualize it, devs build it"
- Avoid sounding like a generic freelancer marketplace

---

## What Agents Should Always Do
- Follow the workspace structure above when building or modifying any feature
- Keep the client/developer relationship at the center of every feature decision
- The AI preview flow (describe → generate → copy/download code) must always be preserved and prioritized
- Status flows should be respected: never skip the "Ready Confirmed" step before development starts
- Design should match existing conventions: teal primary, minimal UI, uppercase status labels
- When in doubt about a feature's purpose, default to: "does this help a non-technical client understand or control their project?"

---
> Source: [Chapadevs/Chapadevs](https://github.com/Chapadevs/Chapadevs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
