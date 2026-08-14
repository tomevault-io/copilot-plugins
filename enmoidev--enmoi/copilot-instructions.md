## enmoi

> > Anciennement **inYou**. La marque est désormais **EnMoi**. Le code, les assets et la

# EnMoi — Application web

> Anciennement **inYou**. La marque est désormais **EnMoi**. Le code, les assets et la
> nomenclature portent encore l'ancien nom à de nombreux endroits : voir « Renommage » plus bas.

## Le produit

EnMoi est une solution de développement personnel fondée sur l'**inné** et l'**acquis**.

Le principe métier :

1. On part de la **date de naissance** d'une personne.
2. **7 formules mathématiques** (paramétrables en base) calculent 7 nombres compris entre 1 et 100.
3. Chacun de ces nombres désigne une **Force** (anciennement « aptitude ») parmi 100.
4. On génère un **PDF livrable : le PMI** (Potentiel Mental Inné), composé de pages
   d'introduction personnalisées puis de **2 pages par force** (14 pages de forces au total).

Chaque force correspond à un **rôle symbolique** fixe, déterminé par sa position dans les 7 :

| Position | Rôle symbolique |
|---|---|
| 1 | Ta colonne vertébrale |
| 2 | Ta boussole |
| 3 | Ta destination |
| 4 | Ton moteur |
| 5 | Ta vitrine |
| 6 | Ton énergie générationnelle |
| 7 | Ton inspiratrice |

### Changement majeur sur le PDF

Le contenu rédactionnel des forces **n'est plus saisi ni stocké en base**. Le client fournit
directement **2 pages PNG par force** (soit 200 fichiers). Le rôle du générateur PDF se réduit à
poser l'image PNG en fond de page et à **surimprimer deux valeurs**.

Toute la logique de mise en page textuelle des fiches (définition vivante, texte emblématique,
forces associées, zones de vigilance, mots-clés) est **obsolète** et doit disparaître.

#### Ce qui est déjà dans l'image (ne jamais le redessiner)

Constaté sur l'échantillon de 10 forces livré : les PNG sont **A4 exact à 300 DPI**
(2480 × 3508 px, ~350 Ko), déjà à la charte **enMOI**, et contiennent déjà le logo, le titre de
la force, tout le texte rédactionnel, la mention « Étape 1 » et le « (tsvp) ».

#### Ce qu'il faut surimprimer

Trois valeurs seulement, sur les zones laissées vides à dessein dans les gabarits :

| Page | Emplacement dans l'image | Valeur |
|---|---|---|
| A | bandeau turquoise, **en haut à gauche** (le logo est centré, « Étape 1 » à droite) | le **prénom** de la personne |
| B | `Force ___ /7 :` — blanc entre « Force » et « /7 » | la **position 1 à 7** |
| B | `Son rôle :` — blanc à droite du libellé | le **rôle symbolique** de la position |

⚠️ Le chiffre imprimé est la **position dans les 7**, jamais le numéro de la force (1-100). Ce
numéro n'apparaît nulle part sur les pages livrées ; il ne sert qu'à choisir le bon PNG.

Le prénom en page A est sur fond turquoise : texte **blanc**, comme les autres éléments du bandeau.

Toutes les coordonnées vivent dans **`lib/generate-pdf/overlayLayout.ts`**, et nulle part ailleurs.
Elles sont exprimées en **pixels du visuel source** (2480 × 3508), donc relevables directement dans
un éditeur d'image ; `pxToPt()` fait la conversion vers les points PDF. Quand le client réédite un
gabarit, on ajuste une constante de ce fichier et rien d'autre.

Chaque zone porte un `minFontSizePt` : la police est réduite automatiquement pour les prénoms longs
et les rôles qui déborderaient de l'espace prévu.

Méthode de calage : composer le texte sur le PNG source avec `sharp` aux mêmes coordonnées et
vérifier visuellement, plutôt que de deviner à l'aveugle.

## Périmètre et priorités

L'application finale regroupe trois parties. Ordre de travail décidé avec le client :

| Partie | Route | Priorité |
|---|---|---|
| **Back-office admin** | `/admin/*` | **Actuelle** — tout est à retravailler, fonctionnel et UI/UX |
| Accueil minimal | `/` | **Actuelle** — page simple + bouton « Se connecter / Créer un compte » en haut à droite, uniquement pour accéder au back-office |
| Site vitrine (présentation EnMoi…) | `/` et pages publiques | Plus tard |
| Espace utilisateur | `/account/*` | Plus tard |

L'accueil n'est **pas** le site vitrine pour l'instant : c'est une porte d'entrée sobre vers
l'authentification. Ne pas y investir de temps de design produit.

## Stack

- **Next.js 15.5** (App Router, Turbopack) — React 19
- **TypeScript**, alias `@/*` vers la racine
- **Tailwind CSS v4** (config CSS-first dans `app/globals.css`, pas de `tailwind.config`)
- **shadcn/ui** (`components.json`, style « new-york » — composants dans `components/ui/`)
- **Prisma 6** + **PostgreSQL sur Neon** (région `eu-central-1`, connexion pooler)
- **better-auth 1.3** (email/mot de passe, sessions en base, plugin `customSession` pour le rôle)
- **Resend** + `@react-email/components` pour les emails transactionnels
- **pdfkit** pour la génération du PMI
- **expr-eval** pour l'évaluation des formules mathématiques
- `react-hot-toast`, `lucide-react`

## Architecture

```
app/
  (web)/page.tsx              accueil public
  (admin)/admin/              back-office — layout garde le rôle ADMIN
    page.tsx                  tableau de bord (cartes de navigation)
    forces/                   médiathèque : dépôt des 2 visuels par force
    formules/                 édition des 7 formules
    pmi/                      génération du PDF
    settings/                 paramètres globaux
  (auth)/auth/                sign-in, forget-password, reset-password
  (customer)/account/         espace utilisateur (vide, plus tard)
  api/
    auth/[...all]/            handler better-auth
    forces/                   liste, upload, suppression, prévisualisation
    mathFunctions/, settings/, pdf/
lib/
  auth.ts, auth-client.ts     instances better-auth serveur / client
  auth-utils/                 getAuthSession, requireRole, errors
  api/apiError.ts             traduction centralisée erreur → statut HTTP
  computeFunctions/           évaluateur d'expressions + variables de naissance
  forces/forceAssets.ts       clés de stockage et validation des PNG
  storage/                    interface ObjectStorage + adaptateurs S3 / local
  generate-pdf/               assemblage du PMI
    overlayLayout.ts          ⭐ toutes les coordonnées de surimpression
  prisma.ts                   singleton PrismaClient
components/
  dataManipulation/           écrans CRUD du back-office
  navbar/                     NavbarDesktopAdmin, NavbarMobileAdmin
  ui/                         shadcn
prisma/
  schema.prisma, migrations/, seed.ts, seed-forces.ts, seed-mathfunctions.ts
public/
  fonts/                      AktivGrotesk (7 graisses), Philosopher-Bold, Rosalia
  pdf-design/                 gabarits PNG des pages d'introduction du PDF
  forces/                     échantillon de travail (10 forces) — non déployé
```

### Gestion des erreurs d'API

Toute route API enveloppe son corps dans un `try/catch` qui délègue à
`apiError(err, contexte)` (`lib/api/apiError.ts`). Cette fonction traduit :

| Erreur | Statut |
|---|---|
| `UnauthorizedError` (pas de session) | 401 |
| `ForbiddenError` (rôle insuffisant) | 403 |
| `ObjectNotFoundError` (clé absente du stockage) | 404 |
| `BusinessError` (message destiné à l'administrateur) | 400 par défaut |
| `ZodError` (entrée invalide) | 400 |
| tout le reste | 500, message générique, détail dans les logs serveur |

Ne jamais renvoyer un message d'erreur brut au client pour une exception
inattendue : il peut révéler des détails d'implémentation.

### Authentification et autorisation

Trois niveaux, à respecter systématiquement :

1. **`middleware.ts`** — vérifie uniquement la *présence* du cookie de session sur `/api/:path*`
   (hors `/api/auth`). Contrôle rapide, ne vérifie **pas** le rôle.
2. **Layouts serveur** — `app/(admin)/admin/layout.tsx` appelle `getAuthSession()` et redirige
   si `role !== "ADMIN"`.
3. **Routes API** — chaque handler appelle `getAuthSession()` puis `requireRole(session, ["ADMIN"])`.
   `getAuthSession()` **throw** si pas de session ; `requireRole` **throw** si le rôle ne colle pas.
   Toujours envelopper dans un `try/catch` qui renvoie un statut HTTP propre.

Rôles : `ADMIN` | `CUSTOMER` (enum Prisma, par défaut `CUSTOMER`).

### Formules mathématiques

Les 7 expressions sont stockées en base (`MathFunction.expression`) et éditables depuis
`/admin/formules`.

L'évaluation passe par un **évaluateur maison** (`lib/computeFunctions/evaluateExpression.ts`) :
tokenizer puis descente récursive sur une grammaire volontairement minimale (arithmétique,
parenthèses, `abs`, `round`, `floor`, `ceil`, `sqrt`, `min`, `max`). Il remplace `expr-eval`,
vulnérable à une prototype pollution sans correctif publié. Aucune propriété n'est lue
dynamiquement et aucun code n'est généré : `constructor` et `__proto__` sont rejetés comme des
variables inconnues.

Variables disponibles à partir de la date de naissance :

| Variable | Sens |
|---|---|
| `j3` / `m3` / `a5` | jour, mois, année complets |
| `j1`, `j2` | chiffres du jour (sur 2 positions) |
| `m1`, `m2` | chiffres du mois (sur 2 positions) |
| `a1`…`a4` | chiffres de l'année (sur 4 positions) |

Les variables sont passées dans une **portée explicite**, pas substituées textuellement. La
contrainte d'ordre de l'ancienne implémentation (remplacer `a5` avant `a1`) n'existe plus : on
peut ajouter une variable dont le nom préfixe une autre sans rien casser.

`evaluateForceNumber()` vérifie en plus que le résultat est un **entier de 1 à 100**. Hors de ces
bornes, la génération échoue avec un message nommant la formule fautive — jamais d'arrondi ou de
repli silencieux qui produirait un PMI faux.

⚠️ Les formules installées par `seed-mathfunctions.ts` sont des **placeholders** qui produisent
des décimaux hors bornes. Tant que le client n'a pas fourni ses vraies expressions, la génération
d'un PMI échoue volontairement.

## État actuel et dettes connues

La refonte est en cours sur la branche `refonte-enmoi`. Phases 0 à 3 terminées
(assainissement, modèle `Force`, stockage S3 + médiathèque, générateur PMI v2).

Dettes restantes :

- **Next 15.5.3 → 16.x** : une RCE critique est corrigée en 16. Migration majeure à traiter dans
  une passe dédiée avant la mise en production.
- **Prisma 6 → 7** : l'extension VS Code signale déjà que `url` dans `datasource` disparaît en v7
  au profit d'un `prisma.config.ts`. Le CLI v6 installé fonctionne : le message est un faux positif
  aujourd'hui, une vraie migration demain.
- Le modèle `BirthProfile` prévoit paiement, tokens, ambassadeurs, quiz — fonctionnalités de la
  phase utilisateur, pas encore branchées. Ne pas s'appuyer dessus pour le back-office.
- **`app/(customer)/account/`** n'est qu'une coquille exigeant une session. À construire avec la
  phase « espace utilisateur ».
- **Aucun test automatisé.** L'évaluateur de formules et le mapping position → force sont les
  premiers candidats.
- **`PASSWORD_ADMIN`** est un mot de passe réel en clair dans `.env` : à faire tourner avant la
  production.

⚠️ **Le temps de génération du PMI est à mesurer en production.** Localement : ~7 s pour 16 pages
et 7 Mo, sans les 14 lectures S3 qui s'y ajouteront. `maxDuration` est fixé à 60 s dans
`app/api/pdf/route.ts`. Si la marge se révèle trop courte, basculer sur une génération asynchrone.

## Modèle Force (migration faite)

Le modèle `Aptitude` a été remplacé par un modèle `Force` minimal, sur une base repartie de zéro.

> Les données de l'ancienne base sont archivées dans
> `02 - travaux/document/archive-db-avant-refonte-enmoi.json` (7 fiches réellement rédigées ;
> au-delà du n° 7 la table ne contenait que des placeholders de test). ⚠️ Les titres qui y figurent
> sont **l'ancienne nomenclature inYou** : le client a entièrement renommé ses 100 forces. Les
> titres de référence sont ceux des visuels livrés.

```prisma
model Force {
  id        String   @id @default(cuid())
  number    Int      @unique   // 1 à 100
  title     String
  pageAKey  String?            // clé de l'objet stocké (page A)
  pageBKey  String?            // clé de l'objet stocké (page B)
  updatedAt DateTime @updatedAt
}
```

Les clés sont **nullable** : une force peut exister sans ses visuels tant que le client ne les a
pas déposés. Le back-office doit rendre cet état visible (100 forces, N complètes).

Conséquences :

Le champ `title` ne sert **qu'au back-office et à la page de synthèse** : le titre de la force est
déjà gravé dans les visuels. `seed-forces.ts` crée les 100 lignes et connaît les 10 titres
confirmés par les visuels livrés ; les 90 autres portent un titre provisoire, à corriger au fur
et à mesure des livraisons du client.

Le back-office propose une **médiathèque des forces** (`/admin/forces`) en remplacement de
l'ancien « Gestion des aptitudes ».

### Assets des forces — upload par le back-office

Le client doit pouvoir **remplacer les visuels lui-même** ; ils sont amenés à changer. Les PNG ne
sont donc **pas versionnés dans le repo** : `public/forces/` ne contient qu'un échantillon de
travail (10 forces livrées par le client, dossiers `N-Nom/`, fichiers `Nom.png` / `Nom2.png`).

Contrainte d'hébergement : la production tourne sur **Vercel**, filesystem read-only. Écrire dans
`public/` à l'exécution est impossible — d'où un **stockage objet externe**. Volume attendu :
200 fichiers × ~350 Ko ≈ **70 Mo**, ce qui tient dans les paliers gratuits.

#### Stockage retenu : AWS S3

Décision : **AWS S3**, bucket dédié sur le **compte AWS de Sébastien** (et non celui du client —
prévoir la réversibilité en fin de contrat : le client devra récupérer ses assets).

Le coût n'a pas été le critère (à 70 Mo : ~0,01 $/mois sur S3, 0 $ sur R2, les deux négligeables).
Le SDK `@aws-sdk/client-s3` parle aussi bien à S3 qu'à tout service S3-compatible : la bascule vers
Cloudflare R2 ou autre ne demanderait que de changer l'endpoint et les credentials.

Configuration du bucket :

- Région **eu-west-3 (Paris)** — latence et RGPD.
- **Block Public Access activé** : les visuels ne sont pas publics. La prévisualisation dans le
  back-office passe par des **URLs présignées** à durée courte.
- **Versioning activé** : le client remplacera des visuels, un rollback sera utile un jour.
- IAM **dédié au seul bucket** (`s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`,
  `s3:ListBucket`) — jamais de clés admin dans l'application.

Le nommage d'origine du client est irrégulier (accents, espaces, casse, `2` en suffixe de page B,
au moins une coquille : `L'Intuituve optimiste`). **Ne jamais dériver la clé de stockage d'une
donnée que quelqu'un peut changer** — ni le nom du fichier uploadé, ni le numéro de force. La clé
est construite à partir de l'`id` (cuid immuable) et de la page : `forces/{id}/{a|b}.png`. Le nom
d'origine n'est conservé que pour l'affichage.

Le numéro est entré dans cette catégorie quand la médiathèque a permis de le réattribuer : s'il
figurait dans la clé, chaque renumérotation obligerait à déplacer des objets dans le stockage, une
opération qui ne peut pas être atomique avec l'écriture en base. Avec l'`id`, renuméroter n'est
qu'un `UPDATE`.

Les routes de lecture (prévisualisation, générateur PDF) lisent **la clé enregistrée en base**, sans
jamais la reconstruire : c'est la seule qui désigne à coup sûr l'objet déposé.

Accès derrière `lib/storage/` (interface + adaptateur), pour que le métier ne dépende pas du
fournisseur et que les tests puissent utiliser un adaptateur local.

En cas de PNG manquant, échouer explicitement avec un message nommant la force et la page — ne
jamais produire un PMI silencieusement incomplet.

### Assets du PDF lus sur disque

Les **polices** (`public/fonts/`) et les **gabarits des pages d'introduction**
(`public/pdf-design/`) sont lus avec `fs`, y compris en production. Les chemins étant construits
dynamiquement, l'analyse statique de Next ne les détecte pas : ils sont déclarés explicitement dans
`outputFileTracingIncludes` (`next.config.ts`) pour être inclus dans le bundle de la fonction.

C'est ce qui manquait à l'implémentation d'origine, laquelle contournait le problème en
**téléchargeant ses propres images en HTTP** depuis le site déployé — un aller-retour réseau par
image, et une génération de PDF dépendante de la disponibilité du site.

Seuls les **visuels de forces** viennent de S3, puisqu'ils sont déposés par le client.

## Renommage inYou → EnMoi

Périmètre retenu pour l'instant : **UI, metadata et assets uniquement**. On **ne renomme pas**
le dossier `inyou-app`, le champ `name` de `package.json`, le dépôt git ni la base Neon — ce sont
des chantiers annexes qui cassent les chemins locaux, à faire plus tard en une passe dédiée.

À traiter : textes visibles, `metadata` de `app/layout.tsx`, `app/robots.ts`, le logo
(`public/logo/logo-inyou.png`), `public/pictures/main-inyou.png`, et les libellés du back-office.

## Conventions de code

- Commentaires et libellés d'interface **en français** ; identifiants en anglais.
- Un commentaire d'en-tête d'une ligne en tête de fichier décrivant son rôle (convention déjà
  en place dans le projet).
- Server Components par défaut ; `"use client"` seulement pour les écrans interactifs
  (`components/dataManipulation/*`).
- Les composants `components/ui/` viennent de shadcn — les régénérer via le CLI plutôt que les
  écrire à la main.
- Couleur de marque actuelle : `#28939f` (variables `--primary`, `--foreground` dans `globals.css`).
  À revalider lors de la refonte UI/UX EnMoi.

## Commandes

```bash
npm run dev                  # dev server (Turbopack)
npm run build
npm run lint

npx prisma studio            # inspecter la base
npx prisma generate
npx prisma migrate dev --name <description>   # migration en dev
npm run migrate:postgres                      # applique les migrations en prod (.env.production)

npx tsx prisma/seed-mathfunctions.ts          # seed des 7 formules (idempotent)
npx tsx prisma/seed.ts                        # seed du compte admin
```

## Environnement

`.env` (dev) et `.env.production` (prod) — **non versionnés**, et à garder ainsi.

| Variable | Rôle |
|---|---|
| `DATABASE_URL` | Neon PostgreSQL (pooler) |
| `BETTER_AUTH_SECRET` | secret de signature des sessions |
| `BETTER_AUTH_URL`, `NEXT_PUBLIC_BETTER_AUTH_URL` | URL de base de l'auth |
| `EMAIL_ADMIN`, `PASSWORD_ADMIN`, `FIRSTNAME_ADMIN`, `LASTNAME_ADMIN`, `ROLE_ADMIN`, `NEXT_PUBLIC_NAME_ADMIN` | compte admin créé par le seed |
| `RESEND_API_KEY`, `RESEND_MAIL_FROM` | envoi des emails |
| `S3_REGION`, `S3_BUCKET` | bucket des visuels de forces (`eu-west-3`) |
| `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY` | IAM dédié, restreint au seul bucket |
| `S3_ENDPOINT` | optionnel — vide pour AWS, renseigné pour un service S3-compatible |

⚠️ `PASSWORD_ADMIN` est un mot de passe réel en clair dans `.env`. Ne jamais le recopier dans du
code, un commentaire, un log ou un message. Prévoir sa rotation avant la mise en production.

Une **CSP stricte** est définie dans `next.config.ts` (plus `Referrer-Policy`, `X-Frame-Options`,
`nosniff`). Toute nouvelle origine externe (police, image, API) doit y être ajoutée explicitement,
sinon elle sera bloquée silencieusement en production.

---
> Source: [enmoidev/enmoi](https://github.com/enmoidev/enmoi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
