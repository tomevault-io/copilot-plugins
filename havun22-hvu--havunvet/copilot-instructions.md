## havunvet

> > Dierenartspraktijk-beheer (Laravel 11 + Livewire 3 + Tailwind, MySQL)

# HavunVet — Claude Instructions

> Dierenartspraktijk-beheer (Laravel 11 + Livewire 3 + Tailwind, MySQL)
> **Staging:** https://staging.havunvet.havun.nl
> **Onveranderlijke regels:** [`CONTRACTS.md`](CONTRACTS.md) — eerst raadplegen.
> **Detail-docs:** `docs/INDEX.md`

## ⛔ Kritieke Gedragsregels (herhaling = overtreding)

| Situatie | Wat Claude doet |
|----------|----------------|
| **Overleg/discussie** | Luisteren, analyseren, samenvatten + plan maken — NOOIT halverwege code schrijven. Code pas na expliciet "ga maar". |
| **Technische beslissing** | Zelf beslissen, kort melden wat er gedaan is — NOOIT vragen aan Henk. |
| **MD bijwerken** (handover/context/KB) | Gewoon doen — NOOIT "mag ik dit documenteren?" vragen. |


## De 5 Onschendbare Regels

1. NOOIT code schrijven zonder KB + kwaliteitsnormen te raadplegen
2. NOOIT features/UI-elementen verwijderen zonder instructie
3. NOOIT credentials/keys/env aanraken
4. ALTIJD tests draaien voor én na wijzigingen (coverage >80%)
5. ALTIJD toestemming vragen bij grote wijzigingen

## Architectuur (samenvatting)

HavunVet (eigenaren, patiënten, behandelingen, medicijnen, afspraken) ↔ HavunAdmin (klanten, facturatie, BTW). Volledig diagram + relaties: `docs/DATABASE.md`.

## Kerndocs

- `docs/SERVER.md` — server + deploy
- `docs/HAVUNADMIN_INTEGRATION.md` — API-koppeling
- `docs/FEATURES.md` — features & roadmap

## Development snelreferentie

```bash
composer install && npm install
npm run dev                      # Terminal 1
php artisan serve --port=8008    # Terminal 2
```

## Deploy (staging)

```bash
cd /var/www/havunvet/staging && git pull
composer install --no-dev --optimize-autoloader
npm run build && php artisan migrate --force
php artisan config:clear && php artisan cache:clear
```

## Havun Standaarden (verplicht — zie HavunCore KB)

Bij elke code-wijziging gelden de centrale Havun-normen. Lees bij twijfel de relevante doc:

| Norm | Centrale doc |
|------|-------------|
| 6 Onschendbare Regels | `HavunCore/CLAUDE.md` |
| Auth-standaard (magic + bio/QR + wachtwoord-optin) | `HavunCore/docs/kb/reference/authentication-methods.md` |
| Test-quality policy (kritieke paden 100 %, MSI ≥ 80 %) | `HavunCore/docs/kb/reference/test-quality-policy.md` |
| Quality standards (>80 % coverage nieuwe code, form requests, rate-limit) | `HavunCore/docs/kb/reference/havun-quality-standards.md` |
| Productie-deploy eisen (SSL/SecHeaders/Mozilla/Hardenize/Internet.nl + OS+app-hardening) | `HavunCore/docs/kb/reference/productie-deploy-eisen.md` |
| V&K-systeem (qv:scan + qv:log) | `HavunCore/docs/kb/reference/qv-scan-latest.md` |
| Test-repair anti-pattern (VP-17) | `HavunCore/docs/kb/runbooks/test-repair-anti-pattern.md` |
| Universal login screen | `HavunCore/docs/kb/patterns/universal-login-screen.md` |
| Werkwijze + beschermingslagen + DO NOT REMOVE | `HavunCore/docs/kb/runbooks/claude-werkwijze.md` |

> **Bij twijfel:** `cd D:/GitHub/HavunCore && php artisan docs:search "<onderwerp>"`


## AI Werkwijze — Gemini + Claude

- **`/arch [opdracht]`** — Gemini blauwdruk genereren (groot contextvenster)
- **`/mpc ga maar`** — blauwdruk uitvoeren
- Blauwdruk landt in `.claude/blueprint.md`, `/start` detecteert dit automatisch

Zie `docs/kb/runbooks/gemini-claude-workflow.md` voor de volledige pipeline.

---
> Source: [havun22-hvu/HavunVet](https://github.com/havun22-hvu/HavunVet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
