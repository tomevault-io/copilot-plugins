## gynaeasy

> - **Next.js 14** App Router + Server Actions (`"use server"`)

# Gynaeasy — Contexte Projet pour Claude

## Stack technique
- **Next.js 14** App Router + Server Actions (`"use server"`)
- **Prisma ORM** + PostgreSQL (Supabase)
- **NextAuth.js** session auth (JWT strategy)
- **LaFricaMobile LAMPUSH API** (SMS Sénégal — tous opérateurs)
- **Coolify** self-hosted sur Hetzner (162.55.162.230) — app UUID `gi7hqgqn9mbaauvn33nv5uf5`
- **AES-256-GCM** (`lib/encryption.ts`) pour données sensibles
- **react-big-calendar** pour l'agenda
- **Zustand** pour les drafts consultation (localStorage)

## Utilisateur
- GitHub: `techbadji` — plan **Pro**
- Langue de travail: **français** (réponses en français ou anglais selon la question)

---

## Thème couleur

Le thème principal est **violet** (`violet-600` / `#7c3aed`).  
Le rose (`pink-600`) ne doit plus apparaître dans l'UI applicative.  
`indigo` ne doit pas non plus apparaître dans l'UI — toujours remplacer par `violet`.  
Exceptions acceptées : gradient décoratif `from-violet-600 to-pink-600` sur le bouton Super Admin uniquement.

---

## LaFricaMobile SMS (LAMPUSH)

Docs : https://developers.lafricamobile.com/docs/sms/introduction

### Configuration Coolify (env vars)

| Variable | Description |
| --- | --- |
| `LAM_ACCESS_KEY` | Access Key fournie par LAM (ex: `DIGITALMATIS.COM_01`) |
| `LAM_ACCESS_PASSWORD` | Access Password LAM |
| `LAM_ENDPOINT` | URL de production fournie par LAM (contacter `assistance@lafricamobile.com`) |
| `LAM_SENDER_ID` | Optionnel — nom affiché (doit être validé par LAM), défaut: `Gynaeasy` |

### Format requête JSON (LAMPUSH)

```json
POST https://lamsms.lafricamobile.com/api
Content-Type: application/json

{
  "accountid": "DIGITALMATIS.COM_01",
  "password": "...",
  "sender": "Gynaeasy",
  "ret_id": "gynaeasy_1234567890",
  "priority": "2",
  "text": "Votre message SMS...",
  "to": [{ "gynaeasy_1234567890": "221771234567" }]
}
```

- `to` : tableau d'objets `{ ret_id: gsm }` — chaque entrée = 1 destinataire
- `gsm` : numéro sans `+` ni `00` (ex: `221771234567`)
- `normalizePhoneNumber()` dans `lib/sms.ts` gère `+221`, `00221`, `0XX`, local
- Mode simulation automatique si `LAM_ACCESS_KEY` ou `LAM_ACCESS_PASSWORD` absent

---

## Architecture rappels SMS & Communications

- `lib/sms.ts` — service bas niveau LaFricaMobile LAMPUSH
- `lib/whatsapp.ts` — service WhatsApp Business API (simulation si non configuré)
- `app/actions/reminders.ts` — rappels RDV, broadcast SMS/WhatsApp, liste par date
- `app/(protected)/sms/page.tsx` — page Communications SMS (médecin + secrétaire)
- `components/sms/sms-broadcast.tsx` — rappels + broadcast + templates + envoi direct 1:1
- `app/api/reminders/cron/route.ts` — endpoint cron sécurisé par `CRON_SECRET`
- `components/admin/super/app-settings.tsx` — test SMS + solde (Super Admin)

---

## Ownership patients — règle critique

Le lien médecin ↔ patient passe par **`Patient.treatingDoctorId`** (pas `Patient.userId`).  
Toujours filtrer par `treatingDoctorId: userId` pour les requêtes patient d'un médecin.  
`Patient.userId` est un champ secondaire (créateur initial), ne pas l'utiliser pour le filtrage métier.

---

## Fixes de sécurité appliqués

| Fichier | Fix |
|---|---|
| `app/actions/settings.ts` | SQL injection → `Prisma.sql` + whitelist de clés |
| `app/actions/user.ts` | SQL injection → `prisma.user.update()` + whitelist |
| `app/api/reminders/sms/route.ts` | Ajout auth guard (`getServerSession`) |
| `app/api/documents/ordonnance/route.ts` | Remplacement données hardcodées par fetch DB réel |
| `lib/encryption.ts` | Erreur au démarrage si `ENCRYPTION_KEY` manquante/invalide |
| `app/actions/consultation.ts` | Validation Zod (`z.record(z.string(), z.unknown())`) |
| `app/actions/onboarding.ts` | Erreurs email remontées au lieu d'être silencieuses |
| `lib/auth.ts` | Suppression `console.log("DEBUG PASS:", ...)` |
| `app/actions/subscription.ts` | Suppression `console.log("DEBUG UPGRADE:", ...)` |
| `app/actions/reminders.ts` | Filtre `treatingDoctorId` (corrige leak inter-médecins) |
| `middleware.ts` | Route `/offline` exclue du guard d'auth (PWA fallback) |

---

## Schéma Prisma notable

- `Consultation.smsReminded: Boolean` — évite les doublons de rappel SMS
- `Consultation.donneesMedicales: Json` — données médicales libres (constantes, écho, etc.)
- `Patient.treatingDoctorId` — médecin traitant principal (relation "TreatingDoctor")
- `Patient.telephone` — nullable (requis pour SMS)
- `Promotion` + `PromoUsage` — codes promo avec tracking par utilisateur
- `Advertisement` — campagnes pub partenaires (clicks + impressions trackés)
- `PlanConfig` — config tarifaire éditable par le Super Admin (prixMensuel + prixAnnuel)

---

## Déploiement Coolify

### Variables d'environnement obligatoires

| Variable | Description |
| --- | --- |
| `DATABASE_URL` | URL PostgreSQL avec `?sslmode=require` |
| `ENCRYPTION_KEY` | 64 chars hex — crash au démarrage si invalide |
| `NEXTAUTH_URL` | URL publique (ex: `https://gynaeasy.digitalmatis.com`) |
| `NEXTAUTH_SECRET` | Secret JWT (min 32 chars) |
| `NEXT_PUBLIC_APP_URL` | Même valeur que NEXTAUTH_URL |

### Variables optionnelles mais importantes

| Variable | Description |
| --- | --- |
| `LAM_ACCESS_KEY` / `LAM_ACCESS_PASSWORD` | SMS réels (simulation si absent) |
| `SMTP_HOST/USER/PASS` | Emails réels (simulation logs si absent) |
| `CRON_SECRET` | Sécurisation endpoint `/api/reminders/cron` |
| `WHATSAPP_API_TOKEN` | WhatsApp Business (simulation si absent) |

- Branche `main` → déploiement automatique sur Coolify
- App UUID Coolify : `gi7hqgqn9mbaauvn33nv5uf5`
- Build : `NODE_OPTIONS='--max-old-space-size=3072'` (Hetzner ~4 Go RAM)

---
> Source: [TechBadji/gynaeasy](https://github.com/TechBadji/gynaeasy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
