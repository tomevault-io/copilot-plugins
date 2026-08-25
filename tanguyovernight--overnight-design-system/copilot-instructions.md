## overnight-design-system

> > Ce dépôt est travaillé par **plusieurs agents IA** + un humain (Tanguy). Lis ceci **avant**

# AGENTS.md — Guide de collaboration (Codex, Claude & autres agents)

> Ce dépôt est travaillé par **plusieurs agents IA** + un humain (Tanguy). Lis ceci **avant**
> toute modification. Objectif : avancer vite **sans jamais casser la démo jouable**.

## 1. Le projet
App mobile de **jeux à boire** (« Overnight »), 18+. Voir les docs de cadrage :
- `CAHIER-DES-CHARGES.md` — vision, modes de jeu, critères v1.
- `BRIEF-MAQUETTES.md` — **direction artistique + tous les écrans + tous les jeux** (la source de vérité UI).
- `JEUX.md` — règles complètes de chaque jeu.
- `RECHERCHE-TECHNIQUE.md` — stack cible (Expo / React Native / Supabase).
- `DIRECTION-ARTISTIQUE.md` — historique de la DA.

Prototype jouable (offline, « 1 téléphone qu'on se passe ») : **`demo/index.html`** — un seul fichier autonome (HTML/CSS/JS vanilla, zéro dépendance).
Démo en ligne : **https://s.chat3d.ai/overnight-jab** (sert un commit figé via CDN).

## 2. ⚠️ Règles de branches (ne PAS casser)
- **Ne JAMAIS pousser sur `main`.** `main` reste propre ; on y arrive **uniquement par Pull Request**.
- **Ne JAMAIS pousser sur la branche d'un autre agent.** Branches réservées :
  - Claude : `claude/*` (ex. `claude/drinking-games-app-spec-4Egty`)
  - Codex : utilise **`codex/<sujet>`** (ex. `codex/maquettes-profil`).
- **1 agent = 1 branche = 1 PR (draft).** Ouvre une **draft PR** vers `main`, l'humain review/merge.
- La démo en ligne pointe sur un **SHA figé**, donc tant que le merge n'est pas fait, **rien ne casse** en prod.
- En cas de doute → **propose un diff / des notes, ne force rien.**

> Recommandation pour Tanguy : activer la **branch protection** sur `main` (require PR review)
> pour rendre toute casse impossible. Tant que ce n'est pas fait, les agents s'auto-disciplinent
> via les règles ci-dessus.

## 3. Carte du repo
```
CAHIER-DES-CHARGES.md   RECHERCHE-TECHNIQUE.md   DIRECTION-ARTISTIQUE.md
BRIEF-MAQUETTES.md      JEUX.md                  AGENTS.md (ce fichier)
content/
  verite-ou-bois.ts     # questions (rounds) — typé
  questions.ts          # ancien brouillon de questions (à fusionner/retirer)
demo/
  index.html            # LE prototype jouable (source de vérité du gameplay)
mockups/                # maquettes statiques (HTML + PNG), itérations de style
```

## 4. Démo : conventions de code (`demo/index.html`)
- **Vanilla JS, un seul fichier, aucune dépendance** (polices via Google Fonts CDN, fallback OK).
- Pattern : un objet `S` (state) + `SCREENS[nom]()` qui renvoie du HTML + `render()` qui injecte dans `#app`. Les boutons appellent des fonctions globales (`onclick="..."`).
- Chaque jeu = `startX()` / `SCREENS.x` / handlers. Ajout d'un jeu : entrée dans `GAMES`, route dans `startGame`, et ses fonctions.
- **Design tokens** : voir les variables CSS `:root` (mêmes valeurs que `BRIEF-MAQUETTES.md` §2). Respecte-les (flat, violet/jaune, Plus Jakarta Sans + Inter, **pas de glow, pas d'emoji UI**, avatars = initiales).
- Strings avec apostrophes : préfère les **backticks** ou les guillemets `"…"` (éviter `\'` mal échappés).

### Tester la démo (obligatoire avant de proposer un changement)
Headless via Playwright (Chromium) :
```bash
node demo/test.mjs        # ouvre la démo, clique dans chaque jeu, échoue s'il y a une erreur JS
```
Le test doit afficher `ERREURS JS: AUCUNE`. (Voir `demo/test.mjs`.)

## 5. Contenu (questions)
- **Mécaniques de jeu = libres de droit** → OK de les réimplémenter.
- **Textes** : viser de l'**original**. Ne pas recopier mot pour mot la prose éditoriale d'apps tierces (risque juridique, cf. `CAHIER-DES-CHARGES.md` §3.3).

## 6. Pour Codex spécifiquement (maquettes)
- La spec UI complète est dans **`BRIEF-MAQUETTES.md`** : palette, typo, espacements, composants, **chaque écran état par état**, les 9 jeux.
- Livre les maquettes **nommées par écran/état** (ex. `palmier-equilibre`, `verite-duree`).
- Si tu touches au code : branche `codex/*` + draft PR, et fais tourner `demo/test.mjs`.
- Si tu ne fais que des visuels/notes : dépose-les (PR ou fichiers dans `mockups/codex/`), l'humain les relaie à Claude qui implémente.

## 7. État actuel (jeux jouables dans la démo)
Le Bus · Palmier (+ mini-jeu d'adresse) · Fuck the Dealer · PMU · Purple · L'Échelle ·
Tape la carte · Vérité ou Bois (144 questions, 10 rounds, choix de durée) · Je n'ai jamais · Tu préfères.
À faire : La Pyramide (bluff, multi-device), synchro multi-téléphones (Supabase), portage Expo.

---
> Source: [TanguyOvernight/overnight-design-system](https://github.com/TanguyOvernight/overnight-design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
