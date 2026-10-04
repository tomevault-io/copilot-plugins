## gudecode-vorschau-kfz-werkstatt

> Slug-basierte, personalisierte Vorschau-Seiten für Outbound-Kampagnen,

# CLAUDE.md — GudeCode Vorschau (Motorrad)

Slug-basierte, personalisierte Vorschau-Seiten für Outbound-Kampagnen,
basierend auf der Bike-Rental-Demo. Siehe `README.md` für Architektur und
`docs/DEPLOY.md` für das Deployment-Runbook.

## Hard rules

- Vorschauseiten bleiben `noindex` und `robots.txt` bleibt Disallow-all —
  hier liegen personalisierte Seiten mit echten Prospect-Daten.
- Keine echten Buchungen/Formulare, keine Tracker, keine Secrets im Repo.
- Demo-Inhalte (Bikes, Touren, Preise) bleiben fiktiv; personalisiert werden
  nur Firmenname, Adresse, Kontakt und Hero-Text aus dem Client-Store.
- Der ERPNext-Sync schreibt nie ungefiltert alle Leads (ERPNEXT_FILTERS Pflicht,
  `--all` nur bewusst).
- Design-Tokens und Monochrom-Fotografie der Motorrad-Vorlage beibehalten;
  Branchen-Themes (z.B. `.theme-salon` für Friseure) überschreiben nur die
  CSS-Variablen in `global.css`, keine neuen Utility-Systeme.

## Stack & Commands

Astro 6 · TypeScript · Tailwind CSS v4 · React-Islands · Vercel (static + eine
SSR-Funktion für `/[slug]`).

```bash
npm ci
npm run dev        # nutzt src/data/clients.local.json
npm run build
npm run test:e2e   # Mock-ERPNext → Sync → Live-Seite
npm run sync       # ERPNext → Edge Config (braucht Env, siehe README)
```

---
> Source: [fillsoko/gudecode-vorschau-kfz-werkstatt](https://github.com/fillsoko/gudecode-vorschau-kfz-werkstatt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
