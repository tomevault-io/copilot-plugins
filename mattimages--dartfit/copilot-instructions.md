## dartfit

> A full-stack precision dart-fitting web application. Users scan their hand, answer a questionnaire, upload an arm photo, and receive a scientifically-calculated dart recommendation matched against a database of 22 real pro-grade darts and 8 pro player profiles.

# DartFit — Claude Code Project

## What This Is
A full-stack precision dart-fitting web application. Users scan their hand, answer a questionnaire, upload an arm photo, and receive a scientifically-calculated dart recommendation matched against a database of 22 real pro-grade darts and 8 pro player profiles.

**Live stack:** Node.js + Express + SQLite (better-sqlite3) + vanilla JS frontend (single HTML file).

---

## Project Structure

```
dartfit/
├── server.js              # Express API server — all routes
├── package.json           # Dependencies
├── .env.example           # Environment variable template
├── lib/
│   ├── algorithm.js       # Biomechanical fitting algorithm (core logic)
│   ├── database.js        # SQLite schema + 22 dart catalog + 8 pro players
│   └── notifications.js   # Web Push (VAPID) + Nodemailer email alerts
├── public/
│   ├── index.html         # Full SPA frontend (single file, no build step)
│   ├── sw.js              # Service Worker for push notifications
│   └── manifest.json      # PWA manifest
└── docs/
    └── dartfit_audit.docx # Internet-wide biomechanics research audit
```

---

## Quick Start

```bash
npm install
cp .env.example .env
# Edit .env — set JWT_SECRET at minimum
npm start
# → http://localhost:3000
```

---

## Architecture

### API Routes (server.js)
| Method | Path | Description |
|--------|------|-------------|
| POST | /api/auth/register | Create account |
| POST | /api/auth/login | Login → JWT |
| GET  | /api/auth/me | Current user + profile |
| GET  | /api/darts | Full dart catalog |
| GET  | /api/pros | Pro player profiles |
| POST | /api/fit/arm-scan | Upload arm image → forearm estimate |
| POST | /api/fit/calculate | Run biomechanical fitting algorithm |
| POST | /api/fit/save | Save profile to DB (auth required) |
| GET  | /api/fit/history | User's profile history (auth required) |
| GET  | /api/push/vapid-key | Public VAPID key for push setup |
| POST | /api/push/subscribe | Register push subscription |
| POST | /api/push/toggle | Enable/disable notifications |
| POST | /api/admin/darts | Add new dart + auto-notify matching users |
| GET  | /api/admin/users | List all users (admin JWT required) |

### Core Algorithm (lib/algorithm.js)
The fitting engine takes these inputs and returns ideal dart specs:

**Inputs:**
- `fingerLength`, `palmWidth`, `gripDiameter`, `fingerSpan` — hand biometrics (mm)
- `fingerFlexIndex`, `throwAngleDeg` — kinematic estimates
- `heightCm` — used to calculate natural throw angle via board geometry
- `forearmLengthMm` — from arm image analysis or estimated from height
- `gripPreference` (1–5), `weightPreference` (1–5) — questionnaire
- `throwingStyle` — front/middle/rear barrel grip position
- `playingLevel` — beginner/intermediate/advanced/competitive

**Physics used:**
- Natural throw angle = `atan2(boardHeight - eyeHeight, ocheDistance)` where board=1730mm, oche=2370mm
- Leverage ratio = forearmLength / totalHeight (population mean 0.148)
- Weight formula accounts for: palm width (2.5g range), finger length (0.8g), height (1.2g), leverage (-1.5g), preference (±4g)

**Outputs:** `idealWeight`, `idealLength`, `idealDiameter`, `idealGripType`, `balance`, `barrelShape`, `naturalThrowAngle`, `leverageRatio`

### Dart Scoring
Each dart in the DB is scored against the ideal profile:
- Weight match: **35%**
- Length match: **20%**
- Diameter match: **15%**
- Grip type match: **15%**
- Balance point match: **10%**
- Barrel shape match: **5%**

### Database (lib/database.js)
SQLite auto-seeds on first run. Tables:
- `darts` — 22 real product entries (Target, Winmau, Harrows, Red Dragon, Unicorn, Mission, Loxley)
- `pro_players` — 8 pro profiles (MvG, Taylor, Wright, Price, Anderson, Sherrock, Clayton, Chisnall)
- `users` — accounts with bcrypt passwords
- `profiles` — saved fitting results per user
- `push_subscriptions` — Web Push endpoint/key storage
- `dart_launches` — new dart notifications log

---

## Key Environment Variables (.env)

```bash
PORT=3000
JWT_SECRET=<random-64-char-string>       # Required
VAPID_PUBLIC_KEY=<from npm run generate-vapid>
VAPID_PRIVATE_KEY=<from npm run generate-vapid>
ADMIN_EMAIL=admin@yourdomain.com
SMTP_HOST=smtp.gmail.com                 # Optional — for email alerts
SMTP_USER=your@gmail.com
SMTP_PASS=your-app-password
```

Generate VAPID keys:
```bash
node -e "const wp=require('web-push');const k=wp.generateVAPIDKeys();console.log('VAPID_PUBLIC_KEY='+k.publicKey+'\nVAPID_PRIVATE_KEY='+k.privateKey);"
```

---

## Notification System
When an admin POSTs to `/api/admin/darts`, the system:
1. Inserts the dart into the DB
2. Scores it against every saved user profile
3. Notifies users with match score ≥ 75 via Web Push AND email

---

## Biomechanics Research Basis
The algorithm is grounded in peer-reviewed science. Key references in `docs/dartfit_audit.docx`:
- **Throw angle model:** IFSSH 2007 Wrist Biomechanics Committee (DTM plane 30–45° oblique)
- **Joint kinematics:** Huang et al. 2024, Journal of Human Sport & Exercise
- **Aerodynamics:** Pawar et al. 2024, Experiments in Fluids (IIT Kharagpur wind tunnel)
- **Oscillation tuning:** James & Potts 2018, Sports Engineering (wavelength ~2.16m matches oche)
- **Grip force/wrist angle:** Journal of Neurophysiology 2020
- **Optimal release strategy:** PMC 2017 (optimal angle 17–37° pre-vertical, speed 5.1–5.5 m/s)

---

## Known Upgrade Opportunities
From the research audit, these variables are scientifically valid but not yet modelled:
1. **Wrist extension angle at release** → grip texture modifier
2. **Oscillation wavelength tuning** → shaft + flight length recommendation
3. **Throw speed self-assessment** → weight and CoG fine-tuning
4. **Hand moisture profile** → grip texture modifier
5. **Tungsten % by playing level** → minimum density recommendation
6. **Shaft/flight selection** → extend fitting beyond barrel only

---

## Monetisation
- All 22 dart `buy_url` fields use Amazon Associates format: `?tag=dartfit-21`
- Replace `dartfit-21` with your Associates tracking ID
- Target, Winmau, Harrows, Red Dragon all have direct affiliate programmes too

## Deployment
Standard Node.js app. Works on Railway, Render, Fly.io, or any VPS.
Database file auto-created at `./data/dartfit.db` on first run.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mattimages) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
