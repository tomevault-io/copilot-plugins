## job-joy

> Après chaque modification de code, sans exception :

# CLAUDE.md — Job Joy

## RÈGLE ABSOLUE — Qualité non négociable

Après chaque modification de code, sans exception :
1. `npm run lint` → 0 erreur, 0 warning sur les fichiers modifiés
2. Tout e2eID renommé/supprimé → mettre à jour les steps BDD correspondants
3. Lancer `C:\dev\job-joy\.claude\craft-ctrl\craft-ctrl.exe` → traiter **toutes** les anomalies. Boy scout : laisser le rapport meilleur qu'à l'arrivée.

Pas de "dette pré-existante" : ClaudeCode est le seul développeur, toutes les non-conformités sont de sa responsabilité.

---

## Présentation

**Job Joy** — Application Electron + Node.js (TypeScript) : collecte offres d'emploi (IMAP, HTML), analyse IA (Claude/Mistral), base SQLite locale, interface React embarquée.
**Stack** : TypeScript · Jest · Playwright/Cucumber (Gherkin fr) · SQLite (`better-sqlite3`) · Electron · Express · React 19 + Tailwind v4 + shadcn/ui.

---

## Mode autonome

Projet personnel — opérer sans confirmation pour les actions locales (git, fichiers, npm). Confirmer uniquement si l'action est irréversible et à fort impact.

**Une branche locale par US** : `us/X-NN` créée au `GO US`, fusionnée à `US VALIDEE`.
- `git checkout -b us/X-NN` au départ · commits WIP libres sur la branche · `git merge && git branch -d` à la clôture
- **Avant tout geste destructif** : `git commit -m "wip: checkpoint avant [opération]"` — obligatoire

---

## Commandes de pilotage

Tu es le **Lead Dev** — orchestre le tunnel, fait les revues, maintient l'état des US en session.
Avant chaque délégation : résumer l'étape, proposer le plan, attendre validation (sauf en `GO FIN`).

### Tunnel

| Commande | Effet |
|----------|-------|
| **`GO US`** | Agent US : reformuler en US + CA, écrire le fichier MD, créer branche `us/X-NN`. Voir `.claude/commands/agent-us.md`. |
| **`GO NEXT`** | Avancer d'une étape : US → IMPACT → BDD → TDD-back → TDD-front → IMPACT-VERIFY → US REVIEW. |
| **`GO FIN`** | Enchaîner GO NEXT sans pause jusqu'à US CONTROL (back-end pur) ou US REVIEW (avec UI) — revues en autonome, s'interrompre si arbitrage nécessaire. |
| **`US REVIEW`** | Présenter les CA livrés, points notables. `@workflow: en review`. Attendre feedback utilisateur. **Protocole feedback → voir ci-dessous.** Quand l'utilisateur valide → US CONTROL. |
| **`US CONTROL`** | DoD complète (`.claude/Software craftsmanship/7.…`). Invoquer `/go-tests <US-X.NN>`. Vérifier lien US↔BDD, captures, guardrail CQRS:ES (`grep db.prepare.*INSERT\|UPDATE` hors `src/infrastructure/` ou `src/kernel/`). 0 anomalie craft-ctrl. Rapport pass/fail + **plan de test manuel** (voir ci-dessous). |
| **`US VALIDEE`** | Clôture. Prérequis : US CONTROL ✅. `git checkout release-windows && git merge us/X-NN && git branch -d us/X-NN`. Puis déplacer le fichier US : `git mv ".claude/sprints/Sprint XX/US-X.NN - Titre.md" "commun/docs/in-appli/documentation/product-management/Sprint XX/US-X.NN - Titre.md"` + commit. |

### Skills (invoquer avec `/nom`)

| Skill | Comportement |
|-------|--------------|
| `/go-tests <US-X.NN>` | Lint + Jest pattern US + BDD `@tag` — pour US CONTROL. `--all` = suite complète pour `/go-publish`. |
| `/go-publish [0-3]` | Version → tests → BDD → doc → metrics → bump → build → commit → tag → push. |
| `/go-doc` | Doc référence + changelog utilisateur. |
| `/go-ds <chemin>` | Audit AST + corrections design system (shadcn, G01–G05). |
| `/go-control-bdd` | Doublons steps + orphelins. Avant toute refacto steps. |
| `/go-controles` | Registre outils qualité. Obligatoire avant modif hook/script/skill. |
| `/go-audit-id-us` | Renommages US en attente → audit références → régénère plan sprints. |

> Menu.ps1 pour process bloquants : serveurs dev (1-2), publication Electron (3-6), BDD site (7), publication site (8).

---

## Le tunnel

| Livrable | Étapes |
|----------|--------|
| Domaine + CLI | US → IMPACT → TDD-back → IMPACT-VERIFY → US CONTROL → US VALIDEE |
| Domaine + UI | US → IMPACT → BDD → TDD-back → TDD-front → IMPACT-VERIFY → US REVIEW → US CONTROL → US VALIDEE |

**US REVIEW existe uniquement pour les US avec UI.** Pour les US back-end pures (pas de TDD-front, `@BDD: non applicable` ou `@BC: Infra`) : IMPACT-VERIFY → US CONTROL directement, sans pause. Alain n'a pas de compétence TypeScript pour faire une revue de code — la REVIEW n'a de valeur que quand il y a une UI à valider fonctionnellement.

Rôles détaillés : `.claude/commands/agent-us.md` · `agent-impact.md` · `agent-bdd-appli.md` · `agent-tdd-back.md` · `agent-tdd-front.md`

---

## Protocole US REVIEW

La phase REVIEW n'est pas une zone libre. Chaque retour utilisateur est classifié avant d'agir.

### Classifier le feedback

| Type | Critère | Traitement |
|------|---------|------------|
| **Nouveau comportement** | L'utilisateur veut quelque chose qui n'était pas dans les CA | Ajouter/modifier le CA dans le fichier US → mini-cycle TDD (RED→GREEN) → BDD si observable → retour REVIEW |
| **Mauvais arbitrage** | Un CA existant décrit mal ce qui est attendu | Corriger le CA dans le fichier US → adapter l'implémentation en TDD → retour REVIEW |
| **Refactoring** | Même comportement, implémentation à retravailler | Refactorer avec couverture de tests maintenue → commit → retour REVIEW |
| **Cosmétique** | Libellé, couleur, alignement sans impact comportemental | Corriger directement → lint → retour REVIEW |

### Règles pendant REVIEW

- **Les CA dans le fichier US reflètent toujours ce qui est réellement implémenté** — si le code diverge d'un CA, l'un des deux est faux. Corriger les deux.
- **Tout ajout de comportement passe par TDD** : RED d'abord, même pour un "petit fix de review".
- **Tout nouveau comportement observable** → step BDD correspondant avant de retourner en REVIEW.
- **Commit après chaque mini-cycle GREEN** — pas de cumul de changements non commités.
- **`npm run lint` + craft-ctrl** après chaque série de corrections, avant de redemander la validation.

### Pré-condition pour passer à US CONTROL

Avant de présenter US CONTROL à l'utilisateur, vérifier :
1. Tous les CA du fichier US reflètent ce qui est implémenté (ni plus, ni moins)
2. Tout CA `@BDD: tester` a son scénario dans la feature
3. `npm run lint` = 0 erreur, 0 warning
4. `craft-ctrl` = 0 anomalie
5. Couverture ≥ 80% sur les fichiers modifiés (y compris ceux modifiés pendant REVIEW)

---

## Plan de test manuel — US CONTROL

À produire systématiquement lors de US CONTROL, **après le rapport pass/fail**.
Objectif : indiquer à Alain où cliquer dans l'UI pour vérifier que rien n'est cassé.

### Format attendu

```
### Chemins à tester

**Chemin 1 — [Nom de la fonctionnalité impactée]**
`Page > Section > Action`
1. [étape concrète avec libellé UI exact]
2. [ce qu'on observe si c'est OK]

**Chemin 2 — ...**
```

### Règles

- **Curatif** : couvrir chaque CA livré — où l'utilisateur voit le résultat concret de l'US.
- **Préventif** : couvrir les zones adjacentes qui auraient pu régresser (imports recâblés, projections partagées, routes API modifiées).
- Nommer les labels UI exacts (texte des boutons, titres de sections, libellés de champs).
- Ne pas lister plus de 5 chemins — se concentrer sur l'essentiel.
- Pour les US `@BDD: non applicable` (refactoring back-end) : les chemins préventifs sont les seuls à lister.

---

## Architecture

```
src/kernel/         ← CQRS:ES : EventBus, EventStore, wiring, types partagés
src/infrastructure/ ← SQLite, HTTP, emails
src/parametrage/    ← BC1  src/collecte/ ← BC2  src/enrichissement/ ← BC3
src/analyse/        ← BC4  src/arbitrage/ ← BC5  src/candidature/ ← BC6
src/suivi/          ← BC7  src/contacts/ ← BC8  src/campagnes/ ← BC9
app/server.ts       ← Adaptateur HTTP (Express + routes API)
app/renderer/       ← Adaptateur UI (React 19 + Vite)
app/globals.css     ← Tokens Tailwind v4 (@theme) — source unique CSS
components/ui/      ← shadcn/ui — NE PAS MODIFIER
tests/bdd-appli/    ← Features app + step definitions (organisation par BC)
data/               ← events.sqlite (EventStore CQRS:ES — source de vérité)
```

Tout code métier → `src/[bc]/`. Jamais dans `utils/` (supprimé) ni `app/` (réservé aux adaptateurs).

---

## Conventions

- **Sprints** : `.claude/sprints/Sprint X — Titre/` · sprint courant = dernier dossier modifié
- **US** : format et annotations `@` → `.claude/craft-ctrl/craft-ctrl-convention.md`
- **Features BDD appli** : `appli/tests/bdd-appli/features/[page].feature` + steps dans `bdd-appli/steps/[page].steps.ts`
- **Organisation BDD** : 1 fichier steps par BC, annotation `// @BC: NomBC` ligne 1, steps partagés dans `shared.steps.ts`
- **Steps partagés** : `shared.steps.ts` — uniquement les steps utilisés par ≥ 3 BCs, sans map e2eid générique
- **Features BDD site** : `site/tests/bdd-site/` — scope Cucumber distinct, ne jamais croiser avec appli
- **docs/** : contenu public uniquement · **temps/** : documents de travail internes

---

## Qualité

**TDD** — RED → GREEN → REFACTOR. 1 commit = 1 test GREEN + implémentation minimum. Couverture 80% sur le code modifié. Pas de code sans test. Ne pas relancer la suite complète pendant le tunnel.

**BDD** — `BDD_IN_MEMORY_STORE=1`, jamais de données réelles. Steps non dupliqués (1 step = 1 fichier). Garde-fou refactoring : `npm run test` + `npm run test:bdd:appli` en vert *avant* de commencer tout refactoring.

**ESLint** — `npm run lint` = 0 erreur, 0 warning. Seuils dans le fichier ratchet (`.claude/Software craftsmanship/4.…`).

**Clean Code** — Noms en français pour le métier. Registres centraux `as const`. YAGNI.

**CQRS:ES** — Toute mutation passe par `bus.publish()`. Jamais INSERT/UPDATE SQL direct. Lire depuis les projections mémoire. Voir `project-regles-absolues.md` (MEMORY).

**Outils qualité** — Registre : `.claude/Software craftsmanship/6.…`. `/go-controles` obligatoire avant toute modif hook/script/skill.

---

## Git

**Format** : `type(scope): message en français`
Préfixes : `feat` `fix` `refactor` `chore` `test` `docs` `style` `revert`
**`wip`** : sans scope, sans quality gate — checkpoints rollback sur branches `us/X-NN`.

Scopes : `domaine` · `api` · `front` · `bdd` · `infra` · `doc`

**Hooks Husky** : `pre-commit` → lint-staged (0 warning) · `commit-msg` → commitlint · `pre-push` → typecheck.
`wip:` passe commitlint sans scope. Snapshot (Menu.ps1 option 2) : force-push vers `snapshot`, pas de hooks.

Ne pas pusher automatiquement sauf demande explicite.

---

## État

> Voir `project-etat-courant.md` (MEMORY) — version, sprints actifs, métriques.
> Vision produit : `.claude/sprints/00 - Vision.md`.

---
> Source: [AlainMeunierFr/job-joy](https://github.com/AlainMeunierFr/job-joy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
