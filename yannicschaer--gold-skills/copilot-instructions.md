## gold-skills

> Team Skills Matrix App zum Erfassen und Visualisieren von Ist- und Soll-Skills aller Teammitglieder.

# Gold Skills - Team Skills Matrix

## Projektübersicht

Team Skills Matrix App zum Erfassen und Visualisieren von Ist- und Soll-Skills aller Teammitglieder.

**Tech-Stack:**
- **Frontend**: React 19 + Vite + TypeScript + Tailwind CSS v4
- **Auth & DB**: Supabase (Email/Password Auth, PostgreSQL, Row Level Security)
- **CMS**: Sanity v3 (Skill-Katalog: Kategorien & Skills)
- **State**: Zustand
- **Hosting**: Vercel
- **Routing**: React Router v7

## Setup auf Mac (Schritt für Schritt)

### 1. Voraussetzungen

```bash
# Node.js >= 18 (empfohlen: via nvm)
brew install nvm
nvm install 20
nvm use 20

# Verify
node -v  # >= 18
npm -v
```

### 2. Repo klonen & Dependencies installieren

```bash
git clone <repo-url> Gold-Skills
cd Gold-Skills
npm install
```

### 3. Supabase einrichten

1. Gehe zu https://supabase.com → neues Projekt erstellen
2. Notiere **Project URL** und **anon public key** (Settings → API)
3. Öffne den **SQL Editor** im Supabase Dashboard
4. Kopiere den Inhalt von `supabase/migrations/001_initial_schema.sql` und führe ihn aus
5. Gehe zu **Authentication → Users** und erstelle einen Test-User (Email/Password)

Die Migration erstellt:
- `profiles` Tabelle (wird automatisch bei User-Signup befüllt via Trigger)
- `skill_ratings` Tabelle (Ist/Soll-Bewertungen pro User und Skill)
- Row Level Security Policies (jeder sieht alle Ratings, bearbeitet nur eigene)

### 4. Sanity einrichten

```bash
# Sanity CLI global installieren
npm install -g sanity@latest

# In den Sanity-Ordner wechseln
cd sanity

# Sanity-Projekt initialisieren (wähle "Create new project")
sanity init

# Notiere die Project ID aus der Ausgabe
# Starte das Studio lokal
sanity dev
```

Das Studio läuft auf `http://localhost:3333`. Dort kannst du:
- **Skill-Kategorien** anlegen (z.B. "Frontend", "Backend", "DevOps", "Soft Skills")
- **Skills** pro Kategorie anlegen (z.B. "React", "TypeScript", "Docker")

Jeder Skill hat ein `order`-Feld für die Sortierung.

Zurück ins Hauptverzeichnis:
```bash
cd ..
```

### 5. Environment Variables setzen

```bash
cp .env.example .env
```

Dann `.env` bearbeiten:
```
VITE_SUPABASE_URL=https://dein-projekt.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbG...dein-anon-key
VITE_SANITY_PROJECT_ID=deine-project-id
VITE_SANITY_DATASET=production
VITE_SANITY_API_VERSION=2024-01-01
```

### 6. App starten

```bash
npm run dev
```

App läuft auf `http://localhost:5173`. Login mit dem in Supabase erstellten Test-User.

### 7. Vercel Deployment

```bash
# Vercel CLI installieren
npm i -g vercel

# Projekt verbinden und deployen
vercel

# Environment Variables in Vercel setzen (Dashboard → Settings → Environment Variables)
# Dieselben wie in .env
```

Alternativ: GitHub-Repo in Vercel Dashboard verknüpfen → auto-deploys bei Push.

## Projektstruktur

```
src/
├── components/
│   ├── Layout.tsx            # App-Shell mit Navbar (Navigation, Logout)
│   ├── ProtectedRoute.tsx    # Redirect zu /login wenn nicht eingeloggt
│   └── SkillLevelSelect.tsx  # Dropdown für Skill-Level 0-5
├── pages/
│   ├── LoginPage.tsx         # Email/Password Login
│   ├── MySkillsPage.tsx      # Eigene Ist/Soll-Skills bearbeiten
│   └── TeamOverviewPage.tsx  # Team-Durchschnitte mit Progress-Bars
├── store/
│   ├── auth.ts               # Zustand Store: Auth-State, Login/Logout
│   └── skills.ts             # Zustand Store: Skill-Katalog + Ratings
├── lib/
│   ├── supabase.ts           # Supabase Client (typisiert)
│   └── sanity.ts             # Sanity Client (CDN-cached)
├── types/
│   ├── database.ts           # TypeScript-Typen für Supabase-Tabellen
│   └── sanity.ts             # TypeScript-Typen für Sanity-Dokumente
├── App.tsx                   # Router-Setup mit geschützten Routes
├── main.tsx                  # Entry Point
└── index.css                 # Tailwind CSS Import

sanity/
├── schemas/
│   ├── skillCategory.ts      # Sanity Schema: Skill-Kategorie
│   ├── skill.ts              # Sanity Schema: Skill (referenziert Kategorie)
│   └── index.ts              # Schema-Export
├── sanity.config.ts          # Studio-Konfiguration
└── package.json              # Separates Package für Sanity Studio

supabase/
└── migrations/
    └── 001_initial_schema.sql  # DB-Schema, RLS-Policies, Trigger
```

## Datenmodell

**Sanity (CMS - Skill-Katalog):**
- `skillCategory`: Titel, Slug, Beschreibung, Reihenfolge
- `skill`: Titel, Slug, Referenz auf Kategorie, Beschreibung, Reihenfolge

**Supabase (DB - User-Daten):**
- `profiles`: id (= auth.users.id), email, full_name, role (admin/member)
- `skill_ratings`: user_id, skill_id (= Sanity _id), current_level (0-5), target_level (0-5)
  - Unique Constraint auf (user_id, skill_id)

## Befehle

| Befehl | Beschreibung |
|--------|-------------|
| `npm run dev` | Startet Vite Dev-Server (Port 5173) |
| `npm run build` | Production Build nach `dist/` |
| `npm run lint` | ESLint ausführen |
| `npm run preview` | Production Build lokal testen |
| `cd sanity && sanity dev` | Sanity Studio starten (Port 3333) |

## Skill-Level Skala

| Level | Bedeutung |
|-------|-----------|
| 0 | Keine Kenntnisse |
| 1 | Grundlagen |
| 2 | Fortgeschritten |
| 3 | Kompetent |
| 4 | Experte |
| 5 | Meister |

## Architektur-Entscheidungen

- **Sanity für Skill-Katalog**: Admins pflegen Skills im Studio, CDN-Caching hält Kosten niedrig. Skill-Definitionen ändern sich selten.
- **Supabase für User-Daten**: Auth + relationale Daten (Ratings) in einem Service. RLS sorgt dafür, dass jeder nur eigene Ratings bearbeiten kann, aber alle lesen darf (für Team-Ansicht).
- **Zustand statt Context**: Minimaler Boilerplate, kein Re-Rendering-Problem, ~1 KB Bundle.
- **Kein ORM**: Supabase Client ist typisiert genug, kein zusätzlicher Layer nötig.
- **Separates Sanity-Package**: Studio wird unabhängig deployed (z.B. auf Sanity selbst), belastet nicht den Frontend-Build.

## Erweiterungsideen (nächste Schritte)

- Dashboard mit Charts (Radar-Chart pro Mitarbeiter, Heatmap fürs Team)
- Admin-Bereich zum Einladen neuer Mitarbeiter (Role `admin` ist vorbereitet)
- Skill-Historie / Zeitverlauf (neue Tabelle + Trigger auf skill_ratings)
- CSV/PDF-Export der Team-Matrix
- Benachrichtigungen bei Skill-Updates

## Kostenübersicht (Free Tiers)

| Service | Free Tier | Reicht für |
|---------|-----------|------------|
| Supabase | 50k MAU, 500 MB DB | Team-Tool mit < 100 Usern problemlos |
| Sanity | 100k API CDN-Requests/Monat | Skill-Katalog wird gecached |
| Vercel | 100 GB Bandwidth | Internes Tool locker |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yannicschaer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
