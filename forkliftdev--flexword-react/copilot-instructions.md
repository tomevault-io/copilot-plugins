## flexword-react

> Flexword Reddit (React Edition)

Flexword Reddit (React Edition)

This app was build in dart/flutter. But getting that app onto Reddit for their hackathon proved too complicated for flutter. So I vibecoded with Gemini Coding partner this project in React, in hopes that it will interact with Devvit better.📂 Project: FlexWord Reddit (React Edition)
Status: Alpha / Deployment Ready Architecture: Devvit Webview (React + Vite + Tailwind)

please help me get this on Reddit. Tell me what files you need to look at. I'm guessing tsconfig.json and index.html

✅ Completed Milestones
Visual Identity (The "Industrial" Look): Replicated the dark-mode palette using #121212 for backgrounds and #E0E0E0 for text.

Accessibility Tile System: Implemented custom SVG shapes to ensure game logic is visible via shape and color:

Blue Circle (#007ACC): Correct letter/spot.

Yellow Triangle (#FFC72C): Correct letter/wrong spot.

Dark Square (#3A3A3A): Incorrect.

Winged Keyboard: Recreated the unique layout featuring a dedicated Space/Underscore wing for visual learners and an integrated Enter/Delete wing.

Word Engine: Integrated the 2,000+ word answer list and 12,000+ valid guess list into a React-optimized logic hook.

🧠 Logic & Scoring (Handoff Details)
The React engine now mirrors the original Dart GameController:

Contract Bidding: Players select from five tiers: Safe (1.1x), Standard (1.5x), Risky (3x), Hard (5x), and Extreme (10x).

The Overtime Mechanic: If a player exceeds their bid limit, the Pot Value is halved for every additional guess taken.

Animated Bank: The bankScore uses an easeOutExpo curve over 1000ms to roll up winnings, matching the original user experience.

🛠️ Developer Environment Notes
TypeScript Config: The src/server/tsconfig.json is specifically configured with jsxImportSource: "@devvit/public-api" to allow JSX in the Devvit server environment.

Entry Points: * Client: src/client/index.html (Vite)

Server: src/server/main.tsx (Devvit)

Build Command: npm run build compiles both the React frontend and the Devvit backend into the dist folder for deployment.

🚀 Final Deployment Step
To see the game live on your subreddit tonight:

Run devvit play in your VS Code terminal.

Open your test subreddit and select "Create FlexWord Post" from the Moderator menu.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forkliftdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/forkliftdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
