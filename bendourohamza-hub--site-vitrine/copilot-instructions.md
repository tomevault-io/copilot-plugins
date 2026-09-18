## site-vitrine

> > ⚠️ **Exercice de formation.** Ce dépôt est le starter d'un atelier Claude Code.

# Site vitrine ORTIS MAROC — instructions du projet

> ⚠️ **Exercice de formation.** Ce dépôt est le starter d'un atelier Claude Code.
> Il ne remplace pas `ortis-maroc.com` et ne part jamais en production. Les
> données ORTIS qu'il contient sont réelles mais déjà publiques, et servent
> uniquement à rendre l'exercice concret. Rien de ce qui est produit ici n'a de
> valeur pour l'entreprise.

Ce fichier est lu par Claude Code au début de **chaque** session.
C'est la mémoire du projet. Tout ce qui est écrit ici, tu n'auras pas à le répéter.

## Le projet

Un site vitrine d'une seule page, statique, pour présenter ORTIS MAROC SARL.
Il est publié sur GitHub Pages depuis la branche `main`, dossier racine.

**L'entreprise** — ORTIS MAROC SARL, BTP à Rabat (Agdal), fondée et dirigée par
Hamza Bendouro. Spécialité : **second œuvre et lots techniques**, quasi
exclusivement en **marchés publics**. Siège de 4 à 5 personnes (direction, RAF,
bureau d'études, achats, contrôle qualité), environ 80 ouvriers et 7 cadres sur
le terrain.

**Donneurs d'ordre réels** (les seuls citables) : ONDA, DGSN, RAM, SRRA,
CGI Management.

- Public visé : les **maîtres d'ouvrage publics et leurs BET / AMO** qui
  vérifient une entreprise avant ou pendant une consultation, plus les
  **mandataires de groupement** qui cherchent un partenaire pour un lot
  technique. Ce ne sont pas des particuliers : ils viennent chercher de la
  preuve, pas de la séduction.
- Ce que le visiteur doit faire en arrivant : **nous appeler ou nous écrire**
  pour une consultation, une demande de dossier administratif, ou une visite de
  chantier. Le téléphone et le mail sont visibles sans scroller.
- Ton de voix : **sobre et institutionnel**. Phrases courtes, vérifiables, à la
  première personne du pluriel côté entreprise. Pas de superlatif, pas de
  vocabulaire commercial ("leader", "excellence", "sur-mesure", "clé en main").

## Coordonnées — valeurs exactes, ne jamais les réécrire de mémoire

- Année de création : **2005**
- Adresse : N°5, Immeuble 9, Rue Al Khatawat, Agdal, Rabat — Maroc
- Téléphone fixe : `+212 5 30 12 02 13`
- Mobile / WhatsApp : `+212 6 62 50 10 44`
- E-mail : `contact@ortis-maroc.com`
- Site principal de l'entreprise : `https://ortis-maroc.com`

Toute autre mention légale (ICE, RC, patente, capital) est **inconnue de ce
projet** : ne jamais écrire un numéro vraisemblable.

L'ICE affiché sur la page est `000000000000000` — quinze zéros, posés par Hamza
comme gabarit visible. Ce n'est **pas** l'ICE d'ORTIS. Le remplacer par le vrai
numéro, ou le laisser tel quel, mais ne jamais le troquer contre autre chose.

**Les lots techniques listés sur la page sont une proposition, pas un fait
vérifié** — électricité courants forts et faibles, plomberie et sanitaire,
climatisation et ventilation, protection incendie. Écrits sur demande de Hamza
le 2026-09-05, en attente de sa validation ligne à ligne.

## Design system

Charte ORTIS officielle, reprise de `ortis-maroc.com`. Elle prime sur toute
préférence de couleur : mon design system passe avant tes choix, et mon prompt
passe avant les deux.

- Colors: primary `#2378AC`, deep `#1A3A52`, accent `#F3C63E`,
  surface `#EDF4F9`, text `#1A3A52`, muted `#5A7387`
- Typography: Inter for body, Inter for headings
- Spacing: 8px scale, 12px border radius
- Style: **anguleux et aéré, plat**. Angles peu arrondis, beaucoup de blanc,
  aucune ombre marquée, aucune illustration décorative. La page doit ressembler
  à un dossier d'entreprise, pas à une landing page de startup.

Pour la version détaillée (composants, états, tons secondaires) : @design-system.md

## Règles de fabrication

- Un seul fichier `index.html`, CSS inclus dans une balise `<style>`. Pas de framework, pas de build.
- Pas de dépendance externe, sauf Google Fonts si le design system demande une police.
- Le site doit être lisible sur mobile : teste toujours en largeur 375px.
- Contraste minimum AA sur tous les textes.
- Les images vont dans `assets/`, jamais ailleurs.
- Chaque section du site a un `id` en minuscules, utilisable comme ancre.
- Le jaune `#F3C63E` ne porte **jamais** de texte sur fond blanc (contraste
  insuffisant) : il sert de fond, de filet ou de puce, avec du `#1A3A52` dessus.
- Le téléphone est un lien `tel:`, le mail un lien `mailto:` : sur mobile, un
  visiteur doit appeler en un geste.
- Page en français. Si une version arabe est demandée un jour, elle sera un
  fichier séparé, jamais un mélange sur la même page.

## Ce que je ne veux pas

- Pas de faux témoignages, pas de faux logos clients, pas de chiffres inventés.
  Si une information manque, laisse un placeholder visible en majuscules.
- **Aucun prix, aucun taux, aucune marge sur le site.** Ce sont des données de
  marché public : elles ne se publient pas.
- Ne cite aucun chantier, aucun montant de marché et aucun client autre que les
  cinq donneurs d'ordre listés plus haut. Un chantier nommé doit m'être demandé
  avant d'être écrit.
- Pas de bandeau cookies, pas de popup, pas de compte à rebours.
- Pas de lorem ipsum : écris du vrai texte à partir de ce que je t'ai dit,
  ou laisse un placeholder explicite.
- Pas de formulaire de contact : il faudrait un service d'envoi, et il n'y en a
  pas ici. Téléphone, WhatsApp et mail suffisent.

## Vérifier avant de dire que c'est fini

- La page s'ouvre sans erreur dans la console du navigateur.
- Les couleurs utilisées sont bien celles de la charte ORTIS ci-dessus.
- Aucun placeholder oublié dans le rendu final.
- Les coordonnées affichées sont identiques, caractère pour caractère, à la
  section « Coordonnées » de ce fichier.
- Aucun chiffre, prix ou nom de chantier que je n'ai pas validé.

---
> Source: [bendourohamza-hub/site_vitrine](https://github.com/bendourohamza-hub/site_vitrine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
