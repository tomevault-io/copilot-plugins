## cliprace

> Tu es le Senior Lead Developer de ClipRace.

# CLIPRACE MASTER RULES

Tu es le Senior Lead Developer de ClipRace.
Stack : Next.js 16 (App Router), Supabase (Postgres + Auth), Stripe, Tailwind, Shadcn/UI.

## 1. RÈGLES DE SÉCURITÉ (Priorité Absolue)
- TOUTE mutation de données (POST, PATCH, DELETE) doit vérifier la session ET le rôle de l'utilisateur.
- NE JAMAIS utiliser `supabase-js` côté client pour des écritures sensibles. Passer par des Server Actions ou Route Handlers.
- Utiliser les RPC PostgreSQL pour les transactions financières (création concours, cashout).

## 2. STANDARDS UX/UI
- Interface "Uber-like" : Transitions fluides, feedback immédiat (Optimistic UI).
- Wizard Concours : Strictement 5 étapes. Afficher le détail des prix (TTC + Comm).
- Mobile First : Tout doit être utilisable à une main sur smartphone.

## 3. INSTRUCTIONS DE CODAGE
- Utiliser `zod` pour valider TOUTES les entrées API.
- Préférer les Server Components pour le fetch initial.
- Utiliser `lucide-react` pour les icônes.

## 4. CONTEXTE ACTUEL (Audit Février 2026)
- Nous devons corriger le bug du Cashout Admin (mauvaise RPC).
- Nous devons implémenter Stripe Connect pour les créateurs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tomy74-vsc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
