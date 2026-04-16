## retap

> ReTap permette a un cliente di usare un unica carta NFC (o un pass Wallet) in qualunque negozio affiliato.

ReTap:

ReTap permette a un cliente di usare un unica carta NFC (o un pass Wallet) in qualunque negozio affiliato.

Il POS ZCS Z91 legge / scrive la carta e chiama le nostre API:

POST /cards → attiva carta nuova

POST /tx → accredita punti

GET /cards/:uid → saldo & data

I dati vivono su Supabase Postgres con Row Level Security: ogni merchant vede solo i propri record.

Il saldo punti non è salvato sul chip ma calcolato on the fly con la view SQL card_points_totals.

Touchpoints:

POS Flutter app (Android) operazioni veloci in negozio.

PWA Cliente (Web NFC / Barcode)  saldo personale e mappa negozi.

Admin Dashboard (Next.js)  gestione clienti, carte, transazioni.

Struttura del monorepo:

retap/
├─ apps/                         # Runtime applications
│   ├─ api-bff/                  # NestJS endpoints (/cards, /tx, /auth…)
│   ├─ website/                  # Next.js 15 + shadcn/ui
│   ├─ flutter-pos/              # Flutter app per POS Z91
│   └─ flutter-web-client/       # Flutter Web PWA (placeholder)
│
├─ packages/                     # Code condiviso
│   ├─ api-types/                # db.ts generato da Supabase (Zod + TS)
│   ├─ ui/                       # Design system (shadcn + Tailwind)
│   └─ shared-utils/             # Date, currency, helpers TS
│
├─ infra/                        # Infrastructure as Code & config
│   ├─ supabase/                 # schema.sql, seed.sql, supabase.toml
│   ├─ docker/                   # docker compose dev stack
│   └─ terraform/                # (facolt.) Cloud Run / VPC
│
├─ .github/                      # GitHub Actions (lint, build, Flutter CI)
├─ pnpm-workspace.yaml           # workspace roots
├─ turborepo.json                # build/lint/test pipeline
└─ README.md

Istruzioni per il progetto:

- per ogni modifica al database, aggiungi una migration diversa in infra/supabase/migrations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/retaployalty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
