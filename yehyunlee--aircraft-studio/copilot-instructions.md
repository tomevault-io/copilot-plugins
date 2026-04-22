## aircraft-studio

> High level description of project


# Aircraft Studio

Design, engineer, and simulate aircraft — mobile-first.

Overview
--------
Aircraft Studio is a mobile-focused studio for designing and testing aircraft. The app helps users create or select aircraft, iterate quickly with AI-assisted image generation, and convert concept images into 3D models for review and AR simulation.

Key user flows
--------------
- Create or select an existing aircraft.
- Create using text prompts with Groq to assist engineering decisions.
- Generate concept imagery rapidly with Fireworks Flux 1 and iterate visually.
- Convert images to 3D (`.glb`) using Spar 3D for previews and quick play.
- Review, refine, and finalize designs.
- Enter AR simulation to pilot, move, and shoot in friendly matches. Record stats and view leaderboards after runs.
- Provide a QR code for quick mobile access.

Technologies (planned)
------------------------------
- Groq — AI text support for engineering prompts
- Fireworks Flux 1 — fast image generation
- Spar 3D — image-to-3D (`.glb`) conversion
- Next.js — frontend framework (this repository)

Design & UX
-----------
- Mobile-first: UI and interactions are optimized for phones and touch.
- Studio vibe: clean, tech-forward visuals and quick iteration paths.


### Command to run locally

```bash
npx next dev --experimental-https
```

### Platform & Infrastructure
https://app.fireworks.ai/
https://platform.stability.ai/
https://cloud.mongodb.com/
https://launchar.app/
https://app.netlify.com/

Windsulf
Groq
Auth0
MongoDB

Flux 1
Spar 3D

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YehyunLee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
