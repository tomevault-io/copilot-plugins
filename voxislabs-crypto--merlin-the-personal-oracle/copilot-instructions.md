## merlin-the-personal-oracle

> While working on merlin


Merlin Development Guide: Standards and Best Practices
Merlin is your personal oracle—a predictive astrology app that starts as a reliable calculator, layers on raw data output, then builds a clean UI, and finally adds visuals inspired by the Predictive Astrology book cover (deep cosmic blue, golden rings, subtle stars, elegant glyphs). We'll build it in sequential sections, each tested and stable before the next. No graphics until the end—keeps the core unbreakable. Each section builds off the last, with clear separation (backend for calc, frontend for display).
General Best Practices

Modular Structure: Use Next.js (app router for v14+). Folders: /app/api (backend), /lib/astrology (core logic), /components (UI), /types (shared types).
Typescript Everywhere: Strong typing prevents bugs. All files .ts or .tsx.
Version Control: Use Git. git init, commit after each section (git add . && git commit -m "Section 1: Calculator complete"). No Surfer overwrites—review changes manually.
Testing: After each section, run npm run dev, input data, check console/output. No errors before moving on.
Dependencies: Install only what's needed per section. Use npm install for prod, -D for dev.
Error Handling: Try/catch in API, console.log for debug, user-friendly messages.
Aesthetic: Dark cosmic blue (#0f172a), golden accents (#fcd34d). No visuals until Section 4.
Data Flow: Calculator outputs JSON. Frontend fetches and displays text first.
Scalability: Keep calculator pure—no UI code. Easy to add transits/LLM later.

Install base: npx create-next-app@latest merlin --ts --app --tailwind --src-dir --import-alias "@/\*".
Section 1: Calculator Backend
Focus: Compute natal chart (planets, houses, aspects). Output JSON. No frontend.
Best Practices:

Use Swiss Ephemeris (sweph) for accuracy.
Validate input (date, time, lat/lon).
Compute: Positions (longitude, sign, degree, minute, house), houses (cusps), aspects (type, orb, exact).
No graphics—pure data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/voxislabs-crypto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
