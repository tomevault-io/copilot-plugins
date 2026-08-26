## aesthetix

> **Esylana** ist eine Multi-Branchen SaaS-Plattform für Dienstleister.

# Esylana - Monorepo

## Überblick

**Esylana** ist eine Multi-Branchen SaaS-Plattform für Dienstleister.
- Multi-Tenant Architektur (ein Deployment = alle Kunden)
- **Monorepo mit pnpm Workspaces**
- **Multi-Branchen:** Kliniken, Gastronomie, Friseure, Spätkauf
- Online-Terminbuchung / Tischreservierung mit Subdomain-Support
- Luxus-Landingpages (Dark Theme)
- KI-Chatbot mit branchenspezifischen Prompts

**Repository:** github.com/xhemo1976/aesthetix

## Tech Stack

- **Monorepo:** pnpm Workspaces
- **Framework:** Next.js 16.1.1 (App Router, Standalone Output)
- **Database:** Supabase (PostgreSQL + Auth + Storage)
- **Styling:** Tailwind CSS + shadcn/ui
- **Hosting:** Hostinger VPS (KVM 4)
- **Process Manager:** PM2
- **Reverse Proxy:** Traefik (mit Let's Encrypt SSL)
- **KI:** OpenAI GPT-4o-mini für Chat

## Monorepo-Struktur

```
/esylana (root)
├── pnpm-workspace.yaml       # Workspace-Konfiguration
├── package.json              # Root package.json
├── CLAUDE.md                 # Diese Datei
│
├── apps/
│   └── web/                  # @esylana/web - Haupt-App
│       ├── app/              # Next.js App Router
│       ├── components/       # App-spezifische Komponenten
│       ├── lib/              # Actions, Supabase, Utils
│       ├── public/           # Static files
│       ├── package.json
│       ├── next.config.ts
│       └── .env.local        # Environment Variables
│
└── packages/
    └── social-media/         # @esylana/social-media
        ├── src/
        │   ├── index.ts
        │   ├── types.ts
        │   ├── utils.ts
        │   └── components/
        │       ├── instagram-feed.tsx
        │       ├── social-links.tsx
        │       └── google-reviews.tsx
        ├── package.json
        └── tsconfig.json
```

## Package-Imports

In `apps/web` kannst du Packages so importieren:

```typescript
// Social Media Komponenten
import { InstagramFeed, SocialLinks, GoogleReviews } from '@esylana/social-media'

// Typen
import type { SocialPost, SocialPlatform } from '@esylana/social-media'

// Utilities
import { formatSocialDate } from '@esylana/social-media'
```

## Häufige Befehle

```bash
# Dependencies installieren (vom Root)
pnpm install

# Entwicklung starten
pnpm dev

# Build (nur web)
pnpm build

# Einzelnes Package/App
pnpm --filter @esylana/web dev
pnpm --filter @esylana/social-media type-check

# Alle Packages aufräumen
pnpm clean
```

## VPS Setup

**Server:** 72.60.36.113 (Hostinger KVM 4, bis 2027-08-22)
**SSH-Passwort:** Donaidan1(2025)

### Deploy-Befehl (Monorepo)

```bash
# Auf VPS - einfach!
cd /var/www/esylana
git pull
pnpm run deploy:prod       # Build + .env kopieren + PM2 restart

# Oder mit Dependencies:
pnpm run deploy:prod:full  # pnpm install + deploy:prod
```

### PM2 Config (einmalig nach Umstellung!)

```bash
# PM2 neu konfigurieren für Monorepo (server.js ist in standalone/apps/web/)
pm2 delete esylana
cd /var/www/esylana/apps/web/.next/standalone/apps/web
PORT=3000 pm2 start server.js --name esylana
pm2 save
```

### Deploy via Python

```python
import pexpect
child = pexpect.spawn('ssh -o StrictHostKeyChecking=no root@72.60.36.113', timeout=300)
child.expect('password:')
child.sendline('Donaidan1(2025)')
child.expect(r'\$|#')
child.sendline('cd /var/www/esylana && git pull && pnpm run deploy:prod')
child.expect(r'\$|#', timeout=300)
print(child.before.decode())
child.close()
```

## Packages

### @esylana/social-media

Social Media Integration Module.

**Komponenten:**
- `InstagramFeed` - Instagram Posts Grid
- `SocialLinks` - Social Media Links (Icons/Buttons/Pills)
- `GoogleReviews` - Google Bewertungen Widget

**Typen:**
- `SocialPlatform` - instagram | facebook | tiktok | google
- `SocialPost` - Post mit Bild, Likes, Comments
- `SocialFeed` - Feed mit mehreren Posts

**Utilities:**
- `formatSocialDate()` - "vor 2 Stunden"
- `truncateText()` - Text kürzen mit ...
- `extractHashtags()` - Hashtags extrahieren
- `formatCount()` - 1.2k, 3.4M

## Neues Package erstellen

```bash
# 1. Ordner erstellen
mkdir -p packages/neues-modul/src

# 2. package.json erstellen
cat > packages/neues-modul/package.json << 'EOF'
{
  "name": "@esylana/neues-modul",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts"
}
EOF

# 3. In apps/web hinzufügen
cd apps/web
pnpm add @esylana/neues-modul@workspace:*
```

## Branchen-Support

| Branche | business_type | Demo-URL |
|---------|---------------|----------|
| Schönheitsklinik | `beauty_clinic` | demo.esylana.de |
| Restaurant | `gastronomy` | gastro.esylana.de |
| Friseur | `hairdresser` | - |
| Spätkauf | `late_shop` | - |

## Features Status

### Fertig ✅
- Multi-Tenant Dashboard
- Multi-Branchen-Support
- Online-Terminbuchung
- Tischreservierung für Gastro
- Chat-Widget mit Booking-Flow
- WhatsApp-Integration
- Kategorie-Bilder
- CSV Export/Import für Gerichte
- **Monorepo-Struktur** mit pnpm Workspaces
- **@esylana/social-media Package** (Gerüst)

### In Arbeit 🚧
- Social Media Integration (Instagram Feed, Google Reviews)

### Geplant 📋
- Stripe-Zahlung
- Analytics Dashboard
- Newsletter-System

## Supabase

**URL:** https://hccoltgswaqhpyzswvwa.supabase.co

### Wichtige Tabellen
| Tabelle | Beschreibung |
|---------|--------------|
| `tenants` | Kliniken/Restaurants (mit `business_type`) |
| `users` | Staff/Admin Accounts |
| `customers` | Endkunden |
| `services` | Behandlungen/Gerichte |
| `appointments` | Termine/Reservierungen |
| `employees` | Mitarbeiter |

## Environment Variables (apps/web/.env.local)

```env
NEXT_PUBLIC_SUPABASE_URL=https://hccoltgswaqhpyzswvwa.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...
NEXT_PUBLIC_APP_URL=https://esylana.de
OPENAI_API_KEY=sk-...
```

## Bekannte Lösungen

### Nach Monorepo-Umstellung: VPS neu konfigurieren
```bash
# Auf VPS
cd /var/www/esylana
npm install -g pnpm
pnpm install
pnpm run build
cp .env.local apps/web/.next/standalone/apps/web/
pm2 delete esylana
cd apps/web/.next/standalone/apps/web
PORT=3000 pm2 start server.js --name esylana
pm2 save
```

### pnpm nicht gefunden auf VPS
```bash
npm install -g pnpm
```

### "Module not found" nach Umstellung
```bash
pnpm install  # Im Root ausführen
```

## Letzte Änderungen (Januar 2026)

1. **Monorepo-Umstellung** - pnpm Workspaces
2. **@esylana/social-media** - Neues Package mit Komponenten
3. **Struktur:** apps/web + packages/social-media

---
> Source: [xhemo1976/aesthetix](https://github.com/xhemo1976/aesthetix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
