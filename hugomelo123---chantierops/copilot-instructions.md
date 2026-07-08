## chantierops

> En travaillant dans le secteur de la construction au Luxembourg, j'ai observé un problème qui se répète partout : les directeurs de travaux ne savent pas ce qui se passe sur leurs chantiers en temps réel.

# ChantierOps — Contexte du projet

## Pourquoi ce projet existe

En travaillant dans le secteur de la construction au Luxembourg, j'ai observé un problème qui se répète partout : les directeurs de travaux ne savent pas ce qui se passe sur leurs chantiers en temps réel.

Le flux d'information est cassé. Les chefs d'équipe envoient des messages WhatsApp dispersés. Les rapports papier arrivent le lendemain. Les problèmes sont découverts trop tard.

La solution évidente — des applications de gestion de chantier — ne fonctionne pas sur le terrain, parce qu'elles demandent un changement de comportement que les équipes n'adoptent pas.

**L'insight central du projet :** ne pas changer ce que les gens font. Utiliser WhatsApp, qu'ils utilisent déjà, comme canal d'entrée. Transformer ces messages en données structurées. Donner la visibilité au directeur sans rien demander de plus aux équipes.

---

## Architecture du projet

```
-ChantierOps/
├── frontend/          Next.js 14 — tableau de bord + landing page
├── backend/           NestJS — API REST + webhook WhatsApp + cron jobs
└── docker-compose.yml PostgreSQL + Backend + Frontend
```

### Frontend — `frontend/src/app/`

Le frontend utilise les **Route Groups** de Next.js App Router :

- `/` → landing page (sans sidebar) — `app/page.tsx`
- `/(app)/` → toutes les pages de l'application (avec sidebar)
  - `/dashboard` — KPIs en temps réel, graphiques, alertes
  - `/chantiers` — liste et création de chantiers
  - `/chantiers/[id]` — détail chantier, rapports, équipes
  - `/rapports` — historique des rapports, export PDF
  - `/alertes` — gestion des alertes, résolution
  - `/materiaux` — demandes de matériaux, approbation
  - `/whatsapp` — envoi d'instructions aux équipes
  - `/settings` — configuration des équipes (nom, chef, numéro WhatsApp, heure de rapport)

### Backend — modules NestJS

| Module | Rôle |
|---|---|
| `chantiers` | CRUD chantiers + assignation équipes |
| `equipes` | Configuration des équipes (ConfigEquipe) |
| `rapports` | Rapports journaliers + export PDF (PDFKit) |
| `alertes` | Alertes + cron job 17h05 lun-ven |
| `materiaux` | Demandes matériaux + workflow approbation |
| `whatsapp` | Webhook Twilio + parsing messages entrants |
| `dashboard` | Agrégation KPIs pour le frontend |

### Base de données — schéma Prisma

Entités principales : `Chantier`, `ConfigEquipe`, `ChantierEquipe` (many-to-many), `Rapport`, `Alerte`, `DemandeMateriau`, `Message`

Statuts chantier : `OK | ALERTE | PARTIEL`
Types d'équipe : `CARRELAGE | MACONNERIE | FACADE | ELECTRICITE`
Sources rapport : `WHATSAPP | MANUEL`

---

## Le flux central — rapport WhatsApp → dashboard

1. Chef d'équipe envoie message WhatsApp
2. Twilio reçoit → webhook `/api/whatsapp/webhook`
3. `WhatsappService.traiterMessageEntrant()` identifie l'équipe par numéro
4. Parsing regex : extrait `avancement`, `hj`, `problèmes`
5. Création `Rapport` en base, mise à jour statut `Chantier`
6. Si problèmes détectés → statut passe à `ALERTE`
7. Auto-résolution alerte `NON_RAPPORT` du jour si elle existait
8. Confirmation WhatsApp renvoyée à l'équipe
9. Frontend recharge (polling 60s) → dashboard mis à jour

---

## Cron job alertes manquantes

```typescript
@Cron('5 17 * * 1-5', { timeZone: 'Europe/Luxembourg' })
```

Chaque jour à 17h05 (lun-ven) : pour chaque chantier actif, si une équipe n'a pas envoyé de rapport aujourd'hui → création alerte `NON_RAPPORT` + rappel WhatsApp au chef d'équipe.

---

## Décisions techniques importantes

**Pourquoi WhatsApp / Twilio**
Twilio WhatsApp Business API est la solution la plus accessible pour prototyper une intégration WhatsApp sans passer par l'API officielle Meta (qui demande une validation longue). En production, la migration vers l'API Meta directe est possible.

**Pourquoi Route Groups Next.js**
La landing page (`/`) ne doit pas avoir la sidebar. Les pages app doivent l'avoir. Les route groups `(app)` permettent deux layouts distincts sans duplication de code.

**Parsing flexible des rapports WhatsApp**
Les regex acceptent plusieurs variantes (`hj`, `hommes`, `hommes.jour`) pour que les chefs d'équipe n'aient pas à respecter un format strict. Un rapport imparfait vaut mieux qu'un rapport non envoyé.

**Statut chantier automatique**
Quand un rapport contient un champ `problèmes`, le chantier passe automatiquement en `ALERTE`. Quand le rapport suivant n'a pas de problèmes, il repasse en `OK`. Pas de saisie manuelle du directeur.

---

## Déploiement

- **Production :** Railway (git-based, variables d'environnement dans le dashboard Railway)
- **Local :** `docker-compose up --build` lance PostgreSQL + backend + frontend
- **CORS :** configuré pour accepter `*.railway.app`

Variables requises : `DATABASE_URL`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_WHATSAPP_FROM`, `NEXT_PUBLIC_API_URL`

---

## Ce qu'il reste à faire

- Authentification (infrastructure JWT présente dans le backend, pas encore connectée au frontend)
- Application mobile native (React Native) pour les chefs d'équipe
- Intégration Odoo pour synchroniser avec la comptabilité
- Module planning d'équipes

---
> Source: [Hugomelo123/-ChantierOps](https://github.com/Hugomelo123/-ChantierOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
