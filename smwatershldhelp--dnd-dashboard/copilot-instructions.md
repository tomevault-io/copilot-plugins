## dnd-dashboard

> D&D DM Tool is a React-based web application for Dungeon Masters to manage campaigns, characters, sessions, and NPCs. Built with React 18.3.1 and Vite for fast development.

# D&D DM Tool - AI Agent Documentation

## Project Overview
D&D DM Tool is a React-based web application for Dungeon Masters to manage campaigns, characters, sessions, and NPCs. Built with React 18.3.1 and Vite for fast development.

## Tech Stack
- **Framework**: React 18.3.1
- **Build Tool**: Vite 5.x
- **UI Library**: HeroUI v3 (react, styles)
- **Styling**: Custom CSS with utility classes
- **State Management**: React useState, useEffect hooks

## Project Structure
```
src/
├── App.jsx                 # Main app component
├── components/             # Reusable components
│   ├── CampaignForm.jsx
│   ├── CampaignList.jsx
│   ├── CharacterForm.jsx
│   ├── CharacterList.jsx
│   ├── CharacterInventory.jsx
│   ├── HistoryLog.jsx
│   ├── NPCForm.jsx
│   ├── NPCList.jsx
│   ├── SessionForm.jsx
│   ├── SessionList.jsx
│   ├── SessionPlanner.jsx
│   ├── Timeline.jsx
│   └── ui/                 # Shared UI components
├── pages/                  # Page components
│   ├── CampaignDashboard.jsx
│   ├── CampaignManagerPage.jsx
│   └── CharacterDashboard.jsx
└── App.css                 # Global styles
```

## Key Features
- Campaign & World Building (campaign creation, lore management)
- Session Management (session planning, timeline tracking)
- NPC Tracker (character management with status tracking)
- Player & Character Tracking (character sheets, inventory)
- Combat & Encounter Tools (combat management in SessionPlanner)
- Timeline/History Log (session recording and continuity)
- Local state management with no external state library

## Current Status
The project has UI components built for all main features but is failing to start due to npm dependency integrity issues (EINTEGRITY errors with react-scripts and other packages). Known issues include:
- npm install failing with EINTEGRITY errors
- react-scripts dependency not resolving properly
- May need to clear npm cache and reinstall dependencies

## Development Commands
- `npm install` - Install dependencies (currently broken due to EINTEGRITY errors)
- `npm run dev` - Start Vite dev server
- `npm run build` - Build for production

## Agent-Specific Notes
- HeroUI components available via @heroui/react and @heroui/styles
- Project uses local CSS files rather than CSS-in-JS solutions
- All components follow React functional component patterns with hooks
- Form components use controlled input patterns
- Status indicators use color-coded badges (green/yellow/red for NPC status)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SMwaterSHLDhelp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SMwaterSHLDhelp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
