## safeneighbor-app

> SafeNeighbor is a community safety React PWA that helps users with emergency situations, de-escalation techniques, legal information, community incident reporting, and personal safety tools.

# SafeNeighbor App - Claude Context

## Project Overview
SafeNeighbor is a community safety React PWA that helps users with emergency situations, de-escalation techniques, legal information, community incident reporting, and personal safety tools.

## Tech Stack
- **Frontend**: React 19 with Create React App
- **Styling**: Tailwind CSS
- **Icons**: Phosphor Icons (@phosphor-icons/react), Lucide React
- **Animation**: Framer Motion
- **Maps**: Leaflet
- **Backend**: Firebase (Firestore, Hosting)
- **Storage**: AWS S3 (video backup)
- **AI**: Gemini API (for legal Q&A feature)

## Key Files & Structure
```
src/
├── App.js                    # Main app with routing/navigation
├── CommunityReports.js       # Incident reporting with Leaflet maps
├── components/
│   ├── Home.js               # Landing page
│   ├── Record.js             # Video recording feature
│   ├── Legal.js              # Legal information with AI Q&A
│   ├── DeEscalation.js       # De-escalation scenarios
│   ├── Scenarios.js          # Emergency scenario list
│   ├── ScenarioDetail.js     # Individual scenario details
│   ├── Whistle.js            # Emergency whistle/signals
│   ├── BackupSettings.js     # Video backup configuration
│   ├── SecuritySettings.js   # App security settings
│   ├── PinSetup.js           # PIN setup for security
│   ├── PinEntry.js           # PIN entry screen
│   ├── Welcome.js            # Welcome modal
│   └── VideoCard.js          # Video display component
├── firebase.js               # Firebase configuration
├── utils/                    # Utility functions
└── data/                     # Static data files
```

## Deployment
- **Firebase Project**: safeneighbor-33bb0
- **Hosting URL**: https://safeneighbor-33bb0.web.app
- **Deploy Command**: `npx firebase-tools deploy --only hosting --project safeneighbor-33bb0`
- **Build Command**: `npm run build`

## Environment Variables (.env)
- `REACT_APP_GEMINI_API_KEY` - Gemini API for legal AI feature
- Firebase config is in src/firebase.js

## Design Patterns
- Dark theme with slate backgrounds and red accent colors
- Glassmorphism styling with backdrop blur
- Mobile-first responsive design
- Phosphor Bold icons preferred (replacing emojis where possible)
- Uppercase tracking-widest labels for form fields
- Active scale-95 transitions on buttons

## Common Tasks
- **Local dev**: `npm start` (runs on localhost:3000)
- **Build**: `npm run build`
- **Deploy**: Build first, then run Firebase deploy command

---
> Source: [keyfram3dev/safeneighbor-app](https://github.com/keyfram3dev/safeneighbor-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
