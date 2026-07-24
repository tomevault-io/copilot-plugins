## robot-components

> This repository contains React components from the Robot Design System. Claude Code can help you extend and customize these components even if you're not a developer.

# Robot Components - Claude Code Guide

This repository contains React components from the Robot Design System. Claude Code can help you extend and customize these components even if you're not a developer.

## Project Structure

```
robot-components/
├── app/                    # Next.js demo pages
│   ├── page.tsx           # Homepage with component links
│   ├── taskpanel/         # Task Panel demo
│   └── nodegrid/          # Node Editor Canvas demo
├── src/
│   ├── taskpanel/         # Task Panel component source
│   └── utils/             # Shared utilities (sounds, etc.)
├── public/
│   └── images/            # Static assets
└── .claude/
    └── skills/            # Claude Code skills for this project
```

## Key Files

- **Node Editor**: `app/nodegrid/page.tsx` - The main Node Editor Canvas component (~4500 lines)
- **Task Panel**: `src/taskpanel/TaskPanel.tsx` - Draggable task panel component
- **Global Styles**: `app/globals.css` - Shared design system styles
- **Sound Effects**: `src/utils/SoundEffects.ts` - Audio feedback system

## Design System

This project uses the **Tailwind CSS neutral palette** for all colors:

| Token | Hex | Usage |
|-------|-----|-------|
| neutral-50 | #fafafa | Primary text |
| neutral-200 | #e5e5e5 | Headings, hover text |
| neutral-400 | #a3a3a3 | Secondary text |
| neutral-500 | #737373 | Muted text, icons |
| neutral-600 | #525252 | Subtle icons |
| neutral-700 | #404040 | Borders, hover backgrounds |
| neutral-800 | #262626 | Card backgrounds |
| neutral-900 | #171717 | Page background |

Accent colors:
- **blue-600** (#2563eb) - Processing/active states
- **green-400** (#4ade80) - Success/completed states

## Available Skills

Use `/node-editor-expand` to add features to the Node Editor Canvas. Describe what you want after invoking the skill.

Example:
```
/node-editor-expand Add a glow effect around panels when they are selected
```

## Running the Demo

```bash
npm install
npm run dev
```

Then visit http://localhost:3000/nodegrid

---
> Source: [dashrobotco/robot-components](https://github.com/dashrobotco/robot-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
