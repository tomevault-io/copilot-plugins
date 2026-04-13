## autodiag-eu-site

> Mode autonome. Reda ne valide PAS chaque fichier individuellement.

# CLAUDE.md — AUTODIAG EU SITE WEB autodiag.eu
# Version 1.0 — 3 avril 2026
# CE FICHIER EST LA SOURCE UNIQUE DE VERITE POUR CLAUDE CODE
# Lire aussi : SPEC_SITE_AUTODIAG_EU.pdf + ADDENDUM_SPEC_SITE_AUTODIAG_EU.pdf dans le Knowledge du projet

---

## MODE DE TRAVAIL

Mode autonome. Reda ne valide PAS chaque fichier individuellement.
- Tu commit et push a chaque phase terminee
- Si un choix est ambigu, prends la decision la plus conservatrice
- Si une erreur bloque le build, corrige-la toi-meme
- Tu ne poses AUCUNE question — tu avances
- Apres chaque phase, execute le quality gate. Si ca echoue, corrige avant de commit
- Mets a jour PROGRESS.md apres chaque etape

---

## REGLES NON NEGOCIABLES

- R1 : Zero console.log dans le code final (sauf lib/analytics.ts)
- R2 : Zero `any` en TypeScript
- R3 : Tous les textes visibles en francais pour le lancement. Ton : ami mecanicien, rassurant, premium
- R4 : Chaque page DTC >= 500 mots de contenu unique
- R5 : Aucun prix invente — fourchettes realistes basees sur les standards du marche automobile EU
- R6 : npx tsc --noEmit + npx next build + npx next lint doivent passer a ZERO erreur avant chaque commit
- R7 : Pas de Google Fonts — font systeme uniquement (zero requete externe pour les polices)
- R8 : Pas de placeholder, pas de lorem ipsum, pas de TODO dans le code commite
- R9 : GA4 NE CHARGE PAS tant que l'utilisateur n'a pas accepte les cookies (RGPD obligatoire)
- R10 : Les cles API serveur (RESEND_API_KEY, GOOGLE_MAPS_API_KEY) ne sont JAMAIS prefixees NEXT_PUBLIC_

---

## IDENTITE DU SITE

Entreprise : AutoDiag EU Sarl (en cours de creation)
Adresse : Route du Jura 10, 2926 Boncourt, Suisse
Email : info@autodiag.eu
Domaine : autodiag.eu (achete)
Fondateur : Reda Kaouani

Ton : Ami mecanicien. Pas de jargon technique inutile, pas de ton corporate.
Exemples :
- BON : "Votre catalyseur fatigue un peu — on vous explique quoi faire"
- MAUVAIS : "Erreur P0420 detectee dans le systeme catalytique"
- BON : "Scannez votre voiture gratuitement"
- MAUVAIS : "Telecharger l'application"
- BON : "On a ecoute votre moteur — voici ce qu'on a trouve"
- MAUVAIS : "Analyse spectrale FFT terminee"

Design : Mode sombre (#050510), glassmorphism, Aurora UI (blobs animes), micro-interactions hover, bento grid, gradient text (cyan #00e5ff → vert #00c853), boutons avec shimmer et glow.
Couleur texte secondaire : #8a90b0 (PAS #6b7194 — ratio contraste insuffisant)

---

## STACK TECHNIQUE

- Framework : Next.js 14+ (App Router)
- Langage : TypeScript strict
- Style : Tailwind CSS
- Animations : Framer Motion
- i18n : next-intl (FR au lancement, EN en Phase 4)
- Deploiement : Vercel
- Email : Resend (gratuit 100/jour)
- Analytics : Google Analytics 4 (avec consentement bloquant)
- PWA : @serwist/next (verifier compatibilite, sinon next-pwa)
- Blog : MDX via @next/mdx ou velite
- Donnees : JSON statiques (zero base de donnees pour le site)
- Supabase : tables beta_requests + ios_waitlist uniquement (projet existant wikoprywqzpovgknqihz)
- Monitoring : Sentry (meme compte eurodiag-eu, nouveau projet autodiag-site)

---

## STRUCTURE DU PROJET

```
autodiag-eu-site/
├── middleware.ts                         # Redirection locale auto (Accept-Language → /fr)
├── app/
│   ├── not-found.tsx                    # Page 404 personnalisee
│   ├── error.tsx                        # Page erreur personnalisee
│   ├── [locale]/
│   │   ├── page.tsx                     # Landing page
│   │   ├── layout.tsx                   # Layout avec nav + footer
│   │   ├── codes/
│   │   │   ├── page.tsx                 # Encyclopedie DTC (recherche + filtres)
│   │   │   └── [code]/page.tsx          # Page DTC individuelle (SSG 250 pages)
│   │   ├── compatibilite/page.tsx       # Verificateur vehicule 677 entries
│   │   ├── demo/page.tsx                # Simulateur app interactif
│   │   ├── stethoscope/page.tsx         # Quiz audio gamifie 3 niveaux
│   │   ├── economies/page.tsx           # Calculateur d'economies
│   │   ├── controle-technique/page.tsx  # Pre-CT interactif 5 pays
│   │   ├── quiz/page.tsx                # Quiz voiture 15 questions
│   │   ├── garages/page.tsx             # Carte garages Google Maps
│   │   ├── prix/page.tsx                # Pricing transparent
│   │   ├── a-propos/page.tsx            # Histoire de Reda
│   │   ├── mentions-legales/page.tsx    # Impressum suisse
│   │   ├── confidentialite/page.tsx     # RGPD + nDSG
│   │   ├── cgv/page.tsx                 # Conditions de vente
│   │   └── blog/
│   │       ├── page.tsx                 # Liste articles
│   │       └── [slug]/page.tsx          # Article MDX
│   └── api/
│       ├── og/route.tsx                 # Open Graph images dynamiques
│       ├── beta/route.tsx               # API formulaire beta (+ Resend email)
│       └── ios-waitlist/route.tsx       # API formulaire iOS waitlist
├── components/
│   ├── layout/
│   │   ├── Navigation.tsx               # 5 entrees, sticky, hide on scroll down
│   │   ├── MobileMenu.tsx               # Hamburger anime + focus trap
│   │   ├── Footer.tsx                   # Liens legaux + langues + copyright
│   │   ├── LiveCounter.tsx              # Compteur anime (estime au lancement)
│   │   └── CTABanner.tsx                # Bandeau telechargement sticky
│   ├── landing/
│   │   ├── HeroSection.tsx              # Aurora blobs + titre gradient + CTA shimmer
│   │   ├── StatsBar.tsx                 # 677/250/5/34
│   │   ├── DiagnosticExpress.tsx        # 8 symptomes interactifs → causes + couts
│   │   ├── InnovationCards.tsx          # 12 innovations bento grid
│   │   ├── PricingSection.tsx           # Gratuit + Premium 49 CHF/an
│   │   ├── BetaForm.tsx                 # Formulaire nom/email/vehicule/pays
│   │   ├── IOSWaitlist.tsx              # Formulaire email iOS coming soon
│   │   └── TestimonialSection.tsx       # Placeholder avis (quand dispo)
│   ├── codes/
│   │   ├── DTCSearch.tsx                # Recherche instantanee cote client
│   │   ├── DTCFilters.tsx               # Filtres par categorie
│   │   ├── DTCCard.tsx                  # Carte resume code
│   │   └── DTCDetail.tsx                # Detail avec VehicleSelector + couts
│   ├── tools/
│   │   ├── VehicleSelector.tsx          # 3 dropdowns cascade marque/modele/annee
│   │   ├── CostCalculator.tsx           # Calculateur economies par pays
│   │   ├── CTChecklist.tsx              # Checklist controle technique
│   │   ├── AppSimulator.tsx             # Demo pipeline OBD2 en boucle 30s
│   │   ├── AudioQuiz.tsx                # Stethoscope 3 niveaux
│   │   ├── CarQuiz.tsx                  # Quiz 15 questions
│   │   └── GarageMap.tsx                # Google Maps (lazy loaded)
│   └── shared/
│       ├── CookieConsent.tsx            # Bandeau BLOQUANT GA4
│       ├── ScrollToTop.tsx              # Bouton remonter
│       ├── Breadcrumbs.tsx              # Fil d'Ariane
│       ├── HoneypotField.tsx            # Anti-spam invisible
│       ├── SeverityBadge.tsx            # Badge gravite DTC
│       ├── CostRange.tsx                # Fourchette prix
│       ├── ShareButton.tsx              # Partage reseaux sociaux
│       ├── DownloadButton.tsx           # Bouton Play Store
│       └── SchemaMarkup.tsx             # JSON-LD structured data
├── data/
│   ├── dtc/
│   │   ├── dtc_codes.json              # 250 codes DTC avec causes, couts, gravite
│   │   ├── dtc_categories.json         # Categories (moteur, transmission, etc.)
│   │   └── dtc_vehicles.json           # Couts specifiques par marque/modele
│   ├── vehicles/
│   │   ├── brands.json                 # 30 marques
│   │   ├── models.json                 # Modeles par marque
│   │   └── compatibility.json          # 677 vehicules (exporter de obd2_protocol_database.ts)
│   ├── pricing/
│   │   ├── plans.json                  # Plans Gratuit + Premium
│   │   └── repair_costs.json           # Couts reparation par pays FR/DE/CH/ES/PT
│   ├── ct/
│   │   ├── fr.json                     # Points CT France
│   │   ├── de.json                     # TUV Allemagne
│   │   ├── ch.json                     # MFK Suisse
│   │   ├── es.json                     # ITV Espagne
│   │   └── pt.json                     # IPO Portugal
│   ├── diagnostic_trees.json           # 8 symptomes × arbre decision
│   ├── audio/samples.json              # Metadonnees echantillons quiz
│   ├── quiz/questions.json             # 15 questions quiz voiture
│   └── features/innovations.json       # 12 innovations avec descriptions
├── content/
│   └── blog/                           # Articles en MDX
│       ├── comment-lire-code-defaut-obd2.mdx
│       ├── preparer-controle-technique.mdx
│       ├── 5-bruits-moteur-a-ne-pas-ignorer.mdx
│       ├── obd2-comprendre-en-5-minutes.mdx
│       └── autodiag-vs-garage-economies.mdx
├── i18n/
│   └── fr.json                         # Traductions francaises (EN en Phase 4)
├── lib/
│   ├── dtc.ts                          # Utilitaires codes DTC
│   ├── vehicles.ts                     # Utilitaires vehicules
│   ├── seo.ts                          # Generation meta tags + JSON-LD
│   ├── i18n.ts                         # Configuration next-intl
│   ├── analytics.ts                    # Events GA4 types avec verification consentement
│   ├── email.ts                        # Service Resend
│   └── rate-limit.ts                   # Rate limiting API routes
├── scripts/
│   ├── quality-gate.sh                 # Verification automatique post-phase
│   └── validate-dtc-data.ts            # Validation donnees DTC
├── public/
│   ├── manifest.json                   # PWA manifest
│   ├── robots.txt                      # Directives crawlers
│   ├── audio/                          # MP3 quiz stethoscope (9 fichiers)
│   └── images/                         # Logo, favicon, OG default
├── PROGRESS.md                         # Avancement mis a jour par Claude Code
├── CLAUDE.md                           # Ce fichier
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## PLANS ET TARIFICATION

### Plan Gratuit
- Scan audio moteur : 1 scan / semaine
- Scanner audio habitacle : 1 scan / semaine
- Lecture codes DTC : illimite
- Live data : 4 PIDs basiques (RPM, vitesse, temperature, charge)
- IA mecanicien : 3 questions / jour
- Historique : 7 derniers jours

### Plan Premium — 49 CHF / an
- Tout illimite
- Effacement codes DTC
- Devis + couts reparation
- Pre-controle technique 5 pays
- Export PDF rapports
- Drive Test sans dongle
- Tous les PIDs
- Historique illimite

Essai gratuit : 7 jours Premium complet. Pas de carte bancaire requise.
Paiement : Google Play Billing exclusivement (pas de Stripe sur le site).
Remboursement : politique Google Play (48h auto, apres sur demande a info@autodiag.eu).

---

## TEXTE "A PROPOS" — MOTS DE REDA (NE PAS REECRIRE)

Bloc 1 — L'histoire :
"Etant un passionne de voitures depuis tout petit, j'ai toujours aime tout ce qui concerne les metiers de l'automobile. J'ai fait plein de metiers differents dans le monde de l'automobile. Les outils ont evolue avec le temps, la mecanique ne se fait plus comme il y a 20 ans. Or les outils permettant un diagnostic clair ne sont pas toujours bon marche et pas toujours mis a jour."

Bloc 2 — La vision :
"J'ai decide de creer un outil simple d'utilisation, fiable et qui sera toujours a jour. AutoDiag EU est une communaute de passionnes et de moins passionnes, d'experts en mecanique et de novices qui veulent juste savoir ou en est leur vehicule dans sa sante. Les professionnels aussi adherent au projet et c'est avec un grand enthousiasme que AutoDiag viendra combler les besoins de ces derniers avec de nouvelles technologies de diagnostics 'acoustique' et 'vision IA'. La revolution est en marche, vous etes sur le bon chemin, adoptez-nous et donnez-nous votre avis !"

---

## 12 INNOVATIONS (afficher meme si pas encore codees dans l'app)

01. Scan audio moteur IA — SVM 91.3%, 11 classes de pannes par le son
02. Scan audio habitacle — 7 classes de bruits interieurs
03. Vision AI / OCR — reconnaissance instantanee codes et pieces
04. IA mecanicien — reponses personnalisees au vehicule (Claude Sonnet)
05. Drive Test sans dongle — accelerometre + gyroscope + GPS + micro du telephone
06. RPM par FFT audio — f = (RPM × N_cyl) / (2 × 60)
07. Amortisseurs par rebond — ratio amortissement zeta
08. Virtual Dyno — P = m·a·v + forces resistives
09. Patinage embrayage — correlation audio RPM × GPS vitesse
10. Disque frein voile — analyse frequentielle pendant freinage
11. Empreinte acoustique vehicule — distance Kullback-Leibler
12. Score predictif flotte — modele de Weibull

---

## FORMULAIRE BETA TESTEURS — PARCOURS COMPLET

1. Utilisateur remplit le formulaire : nom, email, vehicule (optionnel), pays
2. Champ honeypot invisible (si rempli = bot → rejet silencieux)
3. Rate limit : max 3 soumissions par IP par heure
4. Avertissement affiche : "Important : vous aurez besoin d'un compte Google (Gmail) pour installer l'app"
5. Donnees stockees dans Supabase table beta_requests
6. Email automatique envoye via Resend avec le lien opt-in Google Play
7. Si Resend echoue : afficher "Contactez-nous directement a info@autodiag.eu"
8. Reda recoit une notification email pour chaque inscription

Table Supabase beta_requests :
- id (uuid PK), name (text NOT NULL), email (text NOT NULL UNIQUE)
- vehicle (text NULL), country (text NOT NULL)
- email_sent (boolean DEFAULT false), created_at (timestamptz DEFAULT now())
- RLS : INSERT pour anon, SELECT/UPDATE/DELETE pour service_role uniquement

Table Supabase ios_waitlist :
- id (uuid PK), email (text NOT NULL UNIQUE), created_at (timestamptz DEFAULT now())
- RLS : INSERT pour anon, SELECT pour service_role uniquement

---

## MENTIONS LEGALES

Impressum : AutoDiag EU Sarl (en cours de creation), Route du Jura 10, 2926 Boncourt, Suisse.
Email : info@autodiag.eu. Responsable : Reda Kaouani. IDE/CHE : en attente.

Politique confidentialite : Double conformite RGPD (UE) + nDSG (Suisse sept 2023).
Donnees collectees : email/nom (formulaire beta, consentement), GA4 (navigation anonymisee, consentement cookie).
Duree : emails jusqu'a desinscription, GA4 26 mois, diagnostics app purge 30/90j.
Pas de cookies tiers sauf GA4. Bandeau consentement BLOQUANT obligatoire.

CGV : Produit = Premium 49 CHF/an. Facturation Google Play. Essai 7j gratuit.
Renouvellement auto. Annulation via Play Store. Retractation 14j UE. Age 18+.
Tribunal : Canton du Jura, Suisse.

---

## SEO — REGLES CRITIQUES

- 250 codes DTC × langues disponibles = pages SSG via generateStaticParams()
- Chaque page DTC : minimum 500 mots contenu unique, JSON-LD FAQPage, meta tags uniques, hreflang, canonical
- Pas de contenu copie-colle entre pages — chaque page a ses propres causes, couts, description
- Les prix sont des fourchettes realistes (pas de chiffres inventes)
- URL toujours en minuscule (/fr/codes/p0420 pas P0420) — middleware redirige
- Sitemap XML genere automatiquement avec next-sitemap
- robots.txt autorise tout sauf /api/
- Internal linking : chaque page DTC lie vers les codes associes
- Blog : 5 articles MDX de 1500+ mots, ton ami mecanicien

---

## COMPARAISON CONCURRENTS (page dediee)

Comparer ouvertement avec Carly, Car Scanner, OBDeleven.
NE PAS mentionner Skanyx.
Mettre en avant : scan audio IA, vision AI, diagnostic sans dongle, fusion capteurs, Pre-CT 5 pays.
Ces fonctionnalites sont UNIQUES a AutoDiag EU — aucun concurrent ne les a.

---

## CSS CRITIQUE

```css
/* Fallback glassmorphism pour navigateurs sans backdrop-filter */
@supports not (backdrop-filter: blur(10px)) {
  .glass { background: rgba(12, 12, 30, 0.95) !important; }
}

/* Desactiver animations si l'utilisateur le demande */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

/* Print stylesheet pour les pages DTC */
@media print {
  nav, footer, .cta-banner, .cookie-consent, .scroll-to-top { display: none !important; }
  body { background: white !important; color: black !important; }
  * { box-shadow: none !important; text-shadow: none !important; }
}
```

---

## VARIABLES D'ENVIRONNEMENT

```env
# Client (exposees au navigateur)
NEXT_PUBLIC_GA_ID=                    # A remplir au deploiement
NEXT_PUBLIC_SITE_URL=https://autodiag.eu
NEXT_PUBLIC_PLAY_STORE_URL=           # A remplir quand dispo
NEXT_PUBLIC_BETA_OPTIN_URL=           # A remplir quand closed testing actif
NEXT_PUBLIC_SUPABASE_URL=https://wikoprywqzpovgknqihz.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=        # A remplir

# Serveur uniquement (JAMAIS NEXT_PUBLIC_)
RESEND_API_KEY=                       # A remplir
GOOGLE_MAPS_API_KEY=                  # A remplir Phase 5
SENTRY_DSN=                           # A remplir Phase 4
```

---

## QUALITY GATE — EXECUTER APRES CHAQUE PHASE

```bash
#!/bin/bash
# quality-gate.sh — Verification automatique post-phase
set -e

echo "=== QUALITY GATE ==="

echo "[G1] TypeScript..."
npx tsc --noEmit
echo "  PASS"

echo "[G2] Next.js build..."
npx next build
echo "  PASS"

echo "[G3] ESLint..."
npx next lint
echo "  PASS"

echo "[G4] Zero console.log..."
COUNT=$(grep -rn "console\.\(log\|debug\|info\)" src/ app/ components/ lib/ --include="*.ts" --include="*.tsx" | grep -v "analytics.ts" | grep -v "node_modules" | wc -l)
if [ "$COUNT" -gt 0 ]; then
  echo "  FAIL — $COUNT console.log trouves"
  grep -rn "console\.\(log\|debug\|info\)" src/ app/ components/ lib/ --include="*.ts" --include="*.tsx" | grep -v "analytics.ts" | grep -v "node_modules"
  exit 1
fi
echo "  PASS"

echo "[G5] Zero TODO/FIXME..."
COUNT=$(grep -rn "TODO\|FIXME\|HACK\|XXX" src/ app/ components/ lib/ --include="*.ts" --include="*.tsx" | grep -v "node_modules" | wc -l)
if [ "$COUNT" -gt 0 ]; then
  echo "  FAIL — $COUNT TODO trouves"
  exit 1
fi
echo "  PASS"

echo "[G6] Validation donnees DTC..."
if [ -f "scripts/validate-dtc-data.ts" ]; then
  npx ts-node scripts/validate-dtc-data.ts
  echo "  PASS"
else
  echo "  SKIP (script pas encore cree)"
fi

echo "=== TOUS LES GATES PASSENT ==="
```

---

## SCRIPT VALIDATION DTC

```typescript
// scripts/validate-dtc-data.ts
// Verifie l'integrite des donnees DTC avant commit
import dtcCodes from '../data/dtc/dtc_codes.json';

let errors = 0;

for (const dtc of dtcCodes) {
  // Verifier que chaque code a un contenu suffisant
  const wordCount = dtc.description.fr.split(/\s+/).length;
  if (wordCount < 80) {
    console.error(`${dtc.code}: description FR trop courte (${wordCount} mots, min 80)`);
    errors++;
  }

  // Verifier que les prix sont realistes
  for (const cause of dtc.causes) {
    for (const country of ['fr', 'de', 'ch', 'es', 'pt']) {
      const min = cause.costMin?.[country];
      const max = cause.costMax?.[country];
      if (min !== undefined && max !== undefined) {
        if (min <= 0 || max <= 0) {
          console.error(`${dtc.code}: prix <= 0 pour ${country}`);
          errors++;
        }
        if (min > max) {
          console.error(`${dtc.code}: costMin > costMax pour ${country}`);
          errors++;
        }
        if (max > 10000) {
          console.error(`${dtc.code}: prix > 10000 EUR pour ${country} — verifier`);
          errors++;
        }
      }
    }
  }

  // Verifier la severite
  if (!['low', 'medium', 'high', 'critical'].includes(dtc.severity)) {
    console.error(`${dtc.code}: severite invalide "${dtc.severity}"`);
    errors++;
  }

  // Verifier canDrive
  if (typeof dtc.canDrive !== 'boolean') {
    console.error(`${dtc.code}: canDrive manquant ou invalide`);
    errors++;
  }
}

if (errors > 0) {
  console.error(`\n${errors} erreurs trouvees. Corrige avant de commit.`);
  process.exit(1);
} else {
  console.log(`${dtcCodes.length} codes DTC valides. Aucune erreur.`);
}
```

---

## PROGRESS.md — TEMPLATE

```markdown
# PROGRESS — Site web autodiag.eu
# Mis a jour automatiquement par Claude Code

## Phase 1 — Fondation
- [ ] Init Next.js + Tailwind + next-intl + framer-motion
- [ ] middleware.ts (redirection locale)
- [ ] Layout : Navigation + MobileMenu + Footer
- [ ] Landing : HeroSection + StatsBar
- [ ] Landing : DiagnosticExpress (8 symptomes)
- [ ] Landing : InnovationCards (12 innovations bento)
- [ ] Landing : PricingSection (Gratuit + Premium)
- [ ] Landing : BetaForm + IOSWaitlist
- [ ] Landing : CTABanner + LiveCounter
- [ ] CookieConsent bloquant
- [ ] Page 404 + Page erreur
- [ ] Donnees : dtc_codes.json (250 codes FR)
- [ ] Donnees : compatibility.json (677 vehicules)
- [ ] Deploy Vercel (preview)
- [ ] quality-gate.sh PASS

## Phase 2 — Outils
- [ ] DTCSearch + DTCFilters + DTCCard
- [ ] DTCDetail avec VehicleSelector + couts par pays
- [ ] Pages DTC SSG (250 pages FR)
- [ ] Schema JSON-LD FAQ sur chaque page DTC
- [ ] Verificateur compatibilite vehicule
- [ ] Calculateur economies
- [ ] validate-dtc-data.ts PASS
- [ ] quality-gate.sh PASS

## Phase 3 — Gamification
- [ ] AudioQuiz (stethoscope 3 niveaux)
- [ ] CarQuiz (15 questions)
- [ ] CTChecklist (Pre-CT 5 pays)
- [ ] AppSimulator (demo pipeline 30s)
- [ ] quality-gate.sh PASS

## Phase 4 — SEO + i18n
- [ ] Traduction EN complete
- [ ] Sitemap XML + robots.txt
- [ ] Open Graph images dynamiques
- [ ] PWA manifest
- [ ] Sentry integration
- [ ] GA4 events types (lib/analytics.ts)
- [ ] quality-gate.sh PASS

## Phase 5 — Contenu + Legal
- [ ] 5 articles blog MDX (FR)
- [ ] Page A propos (texte Reda)
- [ ] Page comparaison concurrents
- [ ] Mentions legales
- [ ] Politique confidentialite RGPD + nDSG
- [ ] CGV
- [ ] Carte garages (Google Maps lazy)
- [ ] Breadcrumbs + ScrollToTop
- [ ] Print stylesheet
- [ ] Lighthouse >= 95 sur les 4 scores
- [ ] quality-gate.sh PASS

## Statut actuel
Phase : NON COMMENCE
Dernier commit : N/A
Derniere mise a jour : N/A
```

---

## PLAN D'IMPLEMENTATION

### Phase 1 — Fondation (J1-J2)
Init projet, layout complet, landing page complete avec tous les composants,
formulaire beta fonctionnel, donnees DTC 250 codes en JSON, export 677 vehicules,
deploy Vercel preview. Commit : "feat: Phase 1 — fondation + landing page"

### Phase 2 — Outils DTC (J3-J4)
Encyclopedie DTC avec recherche, pages individuelles SSG, Schema JSON-LD,
verificateur compatibilite, calculateur economies. Commit : "feat: Phase 2 — encyclopedie DTC + outils"

### Phase 3 — Gamification (J5-J6)
Quiz audio, quiz voiture, Pre-CT interactif, simulateur app.
Commit : "feat: Phase 3 — gamification"

### Phase 4 — SEO + i18n (J7-J8)
Traductions EN, sitemap, OG images, PWA, Sentry, GA4 events.
Commit : "feat: Phase 4 — SEO + i18n EN"

### Phase 5 — Contenu + Legal (J9-J10)
Blog, a propos, comparaison concurrents, mentions legales, RGPD, CGV,
carte garages, polish final, Lighthouse. Commit : "feat: Phase 5 — contenu + legal"

---

## DONNEES A EXPORTER DE L'APP PRINCIPALE

Le fichier obd2_protocol_database.ts dans C:/Users/rkaou/Desktop/autodiag-eu/app/data/
contient les 677 vehicules. Exporter en JSON pour le site :
- brands.json : liste unique des marques
- models.json : modeles par marque
- compatibility.json : marque + modele + annees + protocole + PIDs supportes

---

## COMMANDE DE LANCEMENT

```
cd C:/Users/rkaou/Desktop
npx create-next-app@latest autodiag-eu-site --typescript --tailwind --app --src-dir --no-eslint
cd autodiag-eu-site
npm install next-intl framer-motion @next/mdx resend @supabase/supabase-js @sentry/nextjs
```

Ensuite copier ce CLAUDE.md a la racine du projet et commencer Phase 1.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/autodiag-eu-org)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/autodiag-eu-org)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
