## ux-good-patterns

> Créer une version améliorée du jeu "Complicité" de VERBAM (https://app.verbam.net/#/pre-play/COMPLICITE) pour proposer des améliorations UX constructives au créateur Florent.

# Projet Complicity - Contexte

## Objectif

Créer une version améliorée du jeu "Complicité" de VERBAM (https://app.verbam.net/#/pre-play/COMPLICITE) pour proposer des améliorations UX constructives au créateur Florent.

**Contrainte importante** : Garder la charte graphique VERBAM pour ne pas brusquer le créateur.

---

## Charte graphique VERBAM

| Élément | Couleur/Style |
|---------|---------------|
| Fond | Bleu foncé navy (~#1e3a5f) |
| Cartes/conteneurs | Jaune/lime vif (~#d4e040) |
| Boutons CTA principaux | Rose/magenta (~#e91e63) |
| Tags joueurs/équipes | Contour corail/rose, fond transparent ou rempli selon sélection |
| Titre "Complicité" | Police playful avec lettres de couleurs variées |
| Logo VERBAM | Lettres colorées avec confettis |
| Icônes | Emojis (⏰ réveil, 🎯 cible, 🏆 trophée, 👥 joueurs) |

---

## Flow des écrans VERBAM original

### 1. Écran pré-jeu
- Titre "Complicité" stylisé
- Description : "Devinez le mot de votre complice avant tout le monde"
- Infos : Joueurs (4, 5 ou 6) | Tour (2 min)
- Bouton "Je joue" (rose)
- Lien "Règles" (ne semble pas fonctionner)

### 2. Choix du mode de jeu
- "Découverte" : mode solo avec joueurs fictifs pour tester
- "À plusieurs" : mode multijoueur réel
- Lien "Revoir les règles"

### 3. Configuration joueurs (mode À plusieurs)
- Header "Joueurs (n)"
- Input "Entrer un autre joueur"
- Tags des joueurs avec ✕ pour supprimer
- Bouton "Faire les équipes"

### 4. Configuration équipes
- Header "Équipes (n)"
- Dropdown "Sélectionner des joueurs" pour chaque équipe
- Bouton ✕ pour supprimer une équipe
- Message "Veuillez créer au moins 2 équipes"
- Bouton "Nouvelle équipe"

### 5. Récapitulatif du simulacre (mode Découverte)
- Modal : "2 équipes jouent : Tintin & Milou contre Batman et Robin"
- Bouton "Bien compris"

### 6. Écran de tour
- "Pack : Complicité" (header)
- Compteur circulaire "X cartes restantes"
- "Au tour de **[Joueur]**"
- Bouton "Je suis bien **[Joueur]**" (confirmation identité)

### 7. Révélation de carte
- Header avec tour et timer "Plus que X min et Y sec"
- "Dévoilez la carte en cliquant dessus"
- Carte face cachée (logo VERBAM)
- Après clic : mot révélé (ex: "La La Land")
- Lien "Choisir un autre mot"
- Bouton "Trouvé ? (ou abandon)"
- Petit ✕ sur la carte pour la recacher

### 8. Sélection du gagnant
- Header "✨ GAGNANT ✨"
- "Sélectionnez le gagnant ou pas"
- Boutons pour chaque équipe + "😢 Personne"
- Bouton "Suivant" après sélection

### 9. Classement
- Header "✨ Classement ✨"
- Liste des équipes avec scores (🏆 pour le leader)
- Bouton "Personne suivante"
- Lien "Quitter"
- Section promo "Découvrez les autres jeux" (en bas)

---

## Plan de refonte UX (AVANT → APRÈS)

Le flow complet comporte **7 écrans** (suppression de l'écran "Sélection du mode"). Les étapes 3→6 bouclent tant qu'il reste des manches (12 manches au total).

| # | Écran | Description | Statut refonte |
|---|-------|-------------|----------------|
| 1 | **HOME** | Écran d'accueil avec titre + bouton | ✅ Implémenté |
| 2 | **Ajout joueurs & équipes** | Configuration des équipes | ✅ Maquette prête |
| 3 | **Annonce du joueur** | "Au tour de [Joueur]" + confirmation | ❌ À designer |
| 4 | **Manche de jeu** | Carte face cachée → révélation mot | ❌ À designer |
| 5 | **Sélection Gagnant** | Choix équipe gagnante de la manche | ❌ À designer |
| 6 | **Affichage Classement** | Scores en cours de partie | ❌ À designer |
| 7 | **Classement Final** | Résultats après toutes les manches | ❌ À designer |

### Maquette écran 2 : Ajout joueurs & équipes

**Amélioration clé** : Remplacer le système dropdown confus par des emplacements visuels directs.

```
┌─────────────────────────────────┐
│          Complicité             │
├─────────────────────────────────┤
│ Equipe 1                        │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │ajouter   │ │ajouter   │ │ajouter   │ │
│ │joueur    │ │joueur    │ │joueur    │ │
│ └──────────┘ └──────────┘ └──────────┘ │
├─────────────────────────────────┤
│ Equipe 2                        │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │ajouter   │ │ajouter   │ │ajouter   │ │
│ │joueur    │ │joueur    │ │joueur    │ │
│ └──────────┘ └──────────┘ └──────────┘ │
└─────────────────────────────────┘
```

- Chaque équipe affiche des emplacements "ajouter joueur"
- Clic sur un emplacement → saisie du nom du joueur
- Plus intuitif que le dropdown original

---

## Points d'amélioration UX identifiés

1. **Règles inaccessibles** : Le lien "Règles" ne semble pas ouvrir de contenu visible
2. **Création d'équipes confuse** : Le système dropdown pour assigner les joueurs aux équipes n'est pas intuitif
3. **Feedback visuel limité** : Peu de retour visuel lors de la sélection du gagnant
4. **Timer peu visible** : Pourrait être plus prominent pendant le jeu
5. **Distractions** : L'écran de classement montre des promos pour d'autres jeux
6. **Navigation limitée** : Pas d'option claire pour revenir en arrière sur certains écrans
7. **Confirmation identité** : L'étape "Je suis bien X" pourrait être optimisée

---

## Structure technique actuelle

```
src/easter-eggs/complicity/
├── index.ts              # Export principal
├── App.tsx               # State machine des écrans
├── types/
│   └── index.ts          # Types Screen, GameConfig, GameState
└── components/
    └── screens/
        └── HomeScreen.tsx  # Écran d'accueil (seul implémenté)
```

### Screens prévus
- `home` : Écran d'accueil ✅
- `setup` : Configuration (placeholder)
- `ready` : Prêt à jouer
- `play` : Jeu en cours
- `roundEnd` : Fin de manche
- `gameEnd` : Fin de partie

### Types définis
```typescript
type Screen = "home" | "setup" | "ready" | "play" | "roundEnd" | "gameEnd";

interface GameConfig {
  teams: string[];
  categories: string[];
  turnDuration: number; // in seconds
}

interface GameState {
  screen: Screen;
  config: GameConfig | null;
}
```

---

## Mots de test

Le jeu utilise des mots/expressions variés comme "La La Land" (films, culture pop, etc.)

Un fichier de mots existe : `public/mots_complet.js` (à vérifier le format)

---
> Source: [clementbouly/ux-good-patterns](https://github.com/clementbouly/ux-good-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
