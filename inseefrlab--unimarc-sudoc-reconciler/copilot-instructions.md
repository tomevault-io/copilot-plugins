## unimarc-sudoc-reconciler

> Ce fichier oriente toute personne (ou assistant IA) qui reprend le code. Il

# CLAUDE.md — Guide de travail sur ce dépôt

Ce fichier oriente toute personne (ou assistant IA) qui reprend le code. Il
décrit l'architecture, les invariants métier à respecter, et ce qui a déjà été
nettoyé.

## Contexte

Outil interne du Centre de ressources documentaires de l'Insee. Objectif :
**fiabiliser les notices du SIGB Syracuse en s'appuyant sur le Sudoc (ABES)**.
C'est un outil de **revue avec l'humain dans la boucle** : la machine détecte les
écarts et propose, mais c'est le documentaliste qui tranche (notamment le choix du
bon PPN pour les notices sans identifiant Sudoc).

## Architecture

- Front **React 19 + Vite + Tailwind** (`src/`), une seule page à 4 vues.
- Back **Express** (`server/`), exécuté par `tsx` (pas d'étape de compilation TS
  côté serveur). En dev, Vite est monté en middleware ; en prod, Express sert
  `dist/`.
- **Aucune base de données.** L'état vit dans une `Map` en mémoire
  (`server/sessions.ts`), indexée par un UUID de session. Volontaire : « un
  fichier en entrée, un fichier en sortie ».
- **Aucun secret dans le code.** Le LLM (endpoint + clé) se configure via l'UI.

## Carte des modules (backend)

| Module | Responsabilité |
| --- | --- |
| `index.ts` | Couche HTTP uniquement : routes + démarrage. Pas de logique métier. |
| `sessions.ts` | Type `Session` + `Map` en mémoire. |
| `syracuse.ts` | Lecture de l'XML Syracuse, catégorisation A/B, tables `SYRACUSE_MAPPING` et `SYRACUSE_TO_UNIMARC`, `UNMODIFIABLE_FIELDS`. |
| `marc.ts` | Accès Sudoc (recherche SRU, récupération par PPN, gestion des PPN fusionnés) et parsing UNIMARC (→ objet structuré et → texte WINIBW). |
| `compare.ts` | Comparaison champ à champ (seuils de similarité, règles ISBN/EAN/année/titre/éditeur) et `compareNoticeWithSudoc`. |
| `verify.ts` | Traitement d'une notice (A ou B) → objet résultat. |
| `exports.ts` | `buildXmlExport`, `buildCsvExport`, `buildTxtExport`. |
| `llm.ts` | Utilitaire d'URL de listing des modèles LLM. |

## Flux de données

`upload` (parse XML → session) → `verify` (boucle sur les notices, appelle
`verifyNotice`, met à jour `progress`) → le front interroge `/api/status` (polling)
→ `DASHBOARD` (résultats + résumé) → `DETAIL` (actions sur les écarts, rattachement,
RAMEAU) → `export` (XML/CSV/TXT construits depuis la session).

## Invariants métier — À NE PAS casser

Ces règles encodent du savoir bibliothéconomique et sont reprises telles quelles
du POC. Les modifier sans validation d'un·e documentaliste = risque de dégrader
les notices.

- **Catégorie A vs B** (`detectCategory`) : A si filtre « SUDOC » ou identifiant
  de la forme PPN (`^[0-9]{8}[0-9X]$`), sinon B.
- **Zones UNIMARC** mappées dans `parseSudocRecord` (200 titre, 010 ISBN, 214/210
  éditeur, 215 collation, 225 collection, 330 résumé, 327 TdM, 700/701/702
  auteurs, 606/607/608 vedettes RAMEAU, etc.).
- **Décodage de la zone 100** (dates de publication) dans `unimarcXmlToText` :
  conversion UNIMARC standard → format Sudoc, très spécifique, à laisser tel quel.
- **Suffixe `rameau`** ajouté aux vedettes : convention Syracuse attendue.
- **Seuils de comparaison** (`compare.ts`) : > 0.9 identique/mineure selon le champ,
  règles particulières ISBN (sans tirets), EAN (sans espaces), année (4 chiffres).
- **Pause de 500 ms** entre deux notices dans la boucle de `verify` : politesse
  envers les serveurs de l'ABES, ne pas retirer.
- **Endpoints ABES** : SRU `https://www.sudoc.abes.fr/cbs/sru/…` ; notice
  `https://www.sudoc.fr/{PPN}.xml` ; fusion `…/services/merged/{PPN}`.


## Limites connues et pistes d'amélioration

- `src/App.tsx` est resté **d'un seul tenant** (repris du POC, dense). Un découpage
  en composants par vue (`UploadView`, `DashboardView`, `DetailView`, …) et
  l'extraction des constantes partagées améliorerait la lisibilité. Report
  volontaire : le faire sans recette navigateur risquait d'introduire des
  régressions.
- Traitement **séquentiel** ; pour de gros volumes, envisager un parallélisme
  borné et une reprise sur erreur.
- Pas de tests automatisés. Prioriser des tests unitaires sur `compare.ts` et
  `marc.ts` (parsing UNIMARC) avec des notices Sudoc réelles figées.

## Recette recommandée avant mise en production

1. `npm run typecheck && npm run build` (doivent passer sans erreur).
2. `npm run start`, puis déposer un **vrai** export Syracuse contenant des notices
   des deux catégories.
3. Vérifier : détection A/B, écarts cohérents, rattachement d'un PPN de catégorie
   B, et les **trois exports** (rouvrir le XML corrigé dans Syracuse, contrôler la
   notation UNIMARC du .txt).

## Déploiement SSP Cloud

Cible : conteneur unique servant le front compilé + l'API, déployé sur Onyxia via
**ArgoCD** (GitOps), sans base de données. Le dossier `deploy/` contient les
manifestes Kubernetes bruts (pas de Helm chart : un seul environnement, pas besoin
de templating).

Flux : `git push` → GitHub Actions construit l'image → ghcr.io → ArgoCD détecte
le changement dans `deploy/` → applique les manifestes au cluster.

Points validés : réseau sortant vers `*.sudoc.*` (ABES) ✓, endpoint LLM interne
`llm.lab.sspcloud.fr` (accessible depuis le cluster) ✓.

## Corrections appliquées après le POC initial (session de déploiement)

- **`base: './'`** dans `vite.config.ts` : indispensable pour que les assets
  (JS, CSS) se chargent correctement derrière un proxy (SSP Cloud, VSCode port
  forwarding). Sans ça → page blanche.
- **Chemins API relatifs** dans `src/App.tsx` : `api/upload` au lieu de
  `/api/upload`. Même raison : un chemin absolu `/api/…` ne passe pas si l'appli
  est servie derrière un sous-chemin de proxy. Correction appliquée sur tous les
  `fetch()`.
- **`allowedHosts: ['.lab.sspcloud.fr']`** dans `vite.config.ts` : autorise le
  serveur de développement Vite à répondre aux requêtes venant des URL du SSP
  Cloud. Sans effet en production.
- **`temperature: 0`** sur l'appel LLM RAMEAU (dans `server/index.ts`) : maximise
  la reproductibilité de la seule brique non-déterministe.

---
> Source: [InseeFrLab/unimarc-sudoc-reconciler](https://github.com/InseeFrLab/unimarc-sudoc-reconciler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
