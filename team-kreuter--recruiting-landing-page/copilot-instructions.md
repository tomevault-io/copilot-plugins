## recruiting-landing-page

> Self-Service Karriereseiten-Template für Unternehmer im deutschen Markt. Admin-Dashboard verwaltet Inhalte als JSON in `data/`. Zielgruppe: Bewerber (nicht B2B-Kunden).

# Recruiting Landing Page

Self-Service Karriereseiten-Template für Unternehmer im deutschen Markt. Admin-Dashboard verwaltet Inhalte als JSON in `data/`. Zielgruppe: Bewerber (nicht B2B-Kunden).

## Tech Stack

- Next.js 16+ (App Router), Tailwind CSS v4 (`@tailwindcss/postcss`), Framer Motion, Lucide React, Nodemailer, crypto-js
- Sprache: Deutsch (UI), Englisch (Code)

## Architektur

```
data/recruiting.json     → Inhalte Recruiting-Landing-Page
data/site.json           → Firmen-Metadaten
data/theme.json          → Farb-Tokens
config/                  → Re-Exports aus data/
types/content.ts         → TypeScript-Interfaces (inkl. JobsContent, CultureContent)
components/sections/     → Section-Komponenten (inkl. Jobs, Culture)
components/ui/           → UI-Komponenten (IconRenderer, Badge, etc.)
components/admin/        → Admin-Dashboard
lib/otp.ts               → HMAC-SHA256 OTP (stateless, kein DB)
app/api/verify-email/    → E-Mail-Verifizierung
app/api/admin/           → Admin-APIs
app/api/contact/         → Bewerbungs-/Kontaktformular-API
```

## Regeln

- Keine neuen Abhängigkeiten ohne Absprache.
- Keine Datenbank. Alles dateibasiert (JSON in `data/`).
- `data/` Dateien nicht manuell editieren — Admin-Dashboard verwaltet sie.
- Icons nur via `IconRenderer` — nie direkt in Section-Komponenten importieren.
- Keine Breaking Changes an der JSON-Struktur. Bestehende Felder nicht umbenennen/entfernen, nur neue optional hinzufügen.
- Keine Environment-Variablen im Client-Code außer `NEXT_PUBLIC_*`.
- TypeScript strict — kein `any` außer wo explizit markiert.
- Tailwind v4 — keine `tailwind.config.ts`.
- Alle Sections müssen vorhanden bleiben. Jobs und Culture sind optional im Interface, aber wenn vorhanden nicht entfernen.
- Mobile-first Layouts.
- Keine externen API-Calls außer SMTP.
- OTP-Logik in `lib/otp.ts` nicht ändern. Kein Twilio/SMS.
- Recruiting-Kontext beibehalten: Texte sprechen Bewerber an, nicht Kunden.

## Befehle

```bash
npm run dev      # Entwicklungsserver
npm run build    # Production Build
npm run lint     # ESLint
npx tsc --noEmit # Type-Check
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/team-kreuter)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/team-kreuter)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
