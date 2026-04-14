## poc-codeforge

> Développer un moteur de rendu robuste et extensible capable de transformer une structure déclarative JSON en une page web fonctionnelle (HTML/CSS/JS). Ce projet sert de base pour un builder low-code/no-code.

# POC - Moteur de Rendu JSON → HTML

## 🎯 Objectif
Développer un moteur de rendu robuste et extensible capable de transformer une structure déclarative JSON en une page web fonctionnelle (HTML/CSS/JS). Ce projet sert de base pour un builder low-code/no-code.

## 🏛️ Principes de Développement

### SOLID
- **S**ingle Responsibility: Chaque composant et utilitaire n'a qu'une seule raison de changer.
- **O**pen/Closed: Le système doit être ouvert à l'extension (nouveaux composants) mais fermé à la modification de son cœur (renderer).
- **L**iskov Substitution: Les composants doivent être interchangeables via une interface commune.
- **I**nterface Segregation: Les props des composants sont spécifiques à leurs besoins.
- **D**ependency Inversion: Le renderer dépend d'abstractions (Registry/Component types) et non d'implémentations concrètes.

### KISS (Keep It Simple, Stupid)
- Favoriser la clarté et la lisibilité du code.
- Éviter les abstractions prématurées.
- Structure de fichiers intuitive et plate.

## ♿ Accessibilité (A11y) - Priorité Haute
Le moteur doit garantir que le HTML généré respecte les normes WCAG :
- Utilisation systématique des balises sémantiques.
- Attributs `aria-*` requis pour les composants interactifs.
- Gestion du focus et navigation au clavier.
- Contrastes de couleurs conformes (via Tailwind).

## 🎨 Design & Style
- **Framework:** Tailwind CSS.
- **Esthétique:** Simple, moderne et dynamique.
- **Approche:** Utilisation de classes utilitaires directement dans les templates ou via des props de style prédéfinies.

## 🧪 Stratégie de Test (Vitest / JSDOM)
Les tests sont le garant de la stabilité. Chaque composant doit être testé sur 4 axes :

1.  **Structure:** Validation du DOM généré (hiérarchie des balises).
2.  **Interaction:** Simulation d'événements (clics, saisies) et vérification des comportements attendus.
3.  **Accessibilité:** Audit automatique (axe-core) et vérification manuelle des rôles/attributs.
4.  **Style:** Vérification de la présence des classes Tailwind critiques pour le layout et le responsive.

## ⚠️ Stabilité & E2E
- **Attention :** Le fichier `data/site.json` sert de base de référence pour les tests End-to-End (Playwright). Toute modification de sa structure, de ses IDs ou de ses textes (appName, titres) peut casser la CI. Veillez à mettre à jour les fichiers `.spec.ts` dans le dossier `e2e/` en cas de changement majeur.

## 🛠️ Stack Technique
- **Langage:** TypeScript (Strongly Typed).
- **Runtime:** Node.js (ESM).
- **CSS:** Tailwind CSS.
- **Tests:** Vitest + JSDOM + Testing Library.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Q300Z)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Q300Z)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
