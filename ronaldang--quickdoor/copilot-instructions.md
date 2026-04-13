## quickdoor

> A downloadable iOS Shortcut that auto-sends your location + photos via WhatsApp when delivery drivers call.

# QuickDoor - Simple Prototype

## Vision
A downloadable iOS Shortcut that auto-sends your location + photos via WhatsApp when delivery drivers call.

## What We're Building
1. **Static landing page** - explains solution, hosts download
2. **iOS Shortcut file** - pre-configured automation users can install

## Tech Stack
- Landing page: Static HTML + Tailwind CSS
- Hosting: GitHub Pages (free)
- Automation: iOS Shortcuts (native)
- No backend, no database, no ongoing costs

## How It Works

**User Setup (once):**
1. Download .shortcut file from website
2. Select 2-3 photos of their building
3. Type address instructions once
4. Install automation

**Daily Use:**
1. Enable "Delivery Mode" when ordering
2. Unknown number calls → WhatsApp auto-sends:
   - Location pin
   - Pre-selected photos  
   - Custom text message

## Files Needed
```
quickdoor/
├── index.html          # Landing page
├── setup-guide.html    # Detailed instructions
├── QuickDoor.shortcut  # The automation file
└── assets/
    ├── demo.gif        # How it works animation
    └── screenshots/    # Setup tutorial images
```

## Build Steps
1. Create iOS Shortcut automation
2. Test with real delivery scenario
3. Export as shareable file
4. Build simple landing page
5. Deploy to GitHub Pages
6. Share link on Twitter/Reddit

## No Database Because
- Everything stored locally on user's phone
- Each user has their own Shortcut instance
- Photos reference user's camera roll
- Zero infrastructure needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ronaldang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ronaldang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
