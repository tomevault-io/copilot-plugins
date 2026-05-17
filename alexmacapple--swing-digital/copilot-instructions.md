## swing-digital

> **Projet** : swing-digital

# Swing Digital - Mémoire Projet

**Projet** : swing-digital
**Type** : Site vitrine statique multi-pages (HTML/CSS/JS)
**Description** : Site Swing Digital - espaces augmentés, expériences immersives
**Date début** : 2026-02-20
**Auteur** : Alex
**Dépôt** : git@github.com:Alexmacapple/swing-digital.git (SSH)

---

## Contexte

Site vitrine pour Swing Digital, entreprise spécialisée dans les expériences immersives et espaces augmentés. Site découpé en 24 pages HTML avec navigation 3 niveaux.

**Architecture** : 4 niveaux de pages
1. Accueil, Espaces augmentés, Reservations
2. Experiences Series (rubrique 8 projets)
3. Pages projet (Monroe, Voyage, Dessine, Ni vues, Marilyn, Toulouse-Lautrec, Charlotte Henschel, XR Corporate)
4. Sous-pages Monroe (Piece, Roman Graphique, Installation, Photographie, Composition, Podcasts, Interviews, Experiences, Quiz)

**Pages utilitaires** : 404, Plan du site, Mentions légales

---

## Stack Technique

- HTML5 sémantique (24 pages)
- CSS3 responsive (variables, BEM, mobile-first, 6 breakpoints)
- JavaScript vanilla (navigation, videos, animations)
- Pas de framework ni bundler

## Polices

Brandon (blk, reg, light-it), Fragen (Bold), Raleway Bold

---

## Structure du site

```
src/
├── index.html                    Accueil (pages 1,2,3,4,8,62)
├── espaces-augmentes.html        Espaces augmentes (pages 5-7)
├── experiences-series.html       Rubrique projets (pages 9-10)
├── experience-monroe.html        Monroe entree (pages 11-13,24)
├── monroe-piece.html             Piece My Story (pages 14-19)
├── monroe-roman-graphique.html   Roman Graphique (pages 20-22)
├── monroe-installation.html      Installation (page 23)
├── monroe-photographie.html      Photographie (pages 25-26)
├── monroe-composition.html       Composition (pages 27-32)
├── monroe-podcasts.html          Podcasts (page 33)
├── monroe-interviews.html        Interviews (pages 34-36)
├── monroe-experiences.html       Experiences interactives (pages 37-39)
├── monroe-quiz.html              Quiz Marilyn (pages 40-41)
├── voyage-autour-de-moi.html     Voyage (pages 42-44)
├── dessine-moi-le-vent.html      Dessine-moi le vent (pages 45-47)
├── ni-vues-ni-connues.html       Ni vues ni connues (pages 48-49)
├── marilyn.html                  Marilyn (pages 50-52)
├── toulouse-lautrec.html         Toulouse-Lautrec (pages 53-54)
├── charlotte-henschel.html       Charlotte Henschel (pages 55-56)
├── xr-corporate.html             XR Corporate (page 57)
├── reservations.html             Reservations (pages 58-61)
├── 404.html                      Page introuvable
├── plan-du-site.html             Plan du site
├── mentions-legales.html         Mentions legales
├── sitemap.xml                   Sitemap (23 pages)
├── robots.txt                    Robots
├── css/style.css                 Styles (158 KB)
├── js/main.js                    Scripts (22 KB)
├── img/                          Images par page
├── fonts/                        Polices web
└── video/                        hero.mp4, contact.mp4
```

---

## Composants communs (dupliqués dans chaque page)

- **Header** : fixe, fond noir, logo texte, menu 4 sections, dropdown 8 projets, sous-menu Monroe (11 liens), hamburger mobile
- **Fil d'Ariane** : sticky sous le header, fond noir, texte blanc
- **Footer** : fond noir, copyright, liens (Équipe, Contact, Plan du site, Mentions légales)
- **Skip link** : en dur dans le HTML, premier élément du body

---

## Accessibilité - WCAG 2.2 AA / RGAA 4.1

- 0 violation axe-core sur 23/24 pages
- Navigation clavier complète (Tab, Escape, Arrow)
- Disclosure pattern sur dropdown et hamburger
- aria-current="page" sur lien actif et breadcrumb
- h1 sur chaque page (sr-only si pas de titre visible)
- Textes en casse normale, majuscules via CSS (RGAA 10.2)
- prefers-reduced-motion respecte (CSS + JS)
- Contraste conforme AA (blanc sur noir 21:1)
- Intitules de liens explicites (RGAA 6.1)
- Boutons video avec intitule identifiant le contenu

---

## Regles CSS strictes

- Zero couleur codee en dur hors :root
- 38+ variables couleur dans :root
- BEM naming sur tous les composants
- text-transform: uppercase pour les majuscules visuelles (pas dans le HTML)
- Breakpoints : 480px, 600px, 768px, 1024px, 1200px, 1441px

## Branche de travail

**Branche active** : main
**Tags** : v1, v2, v3, v4 (découpé multi-pages)
**Branches mergées** : cosmétique, image, decoupage

---

## Decisions

1. Découpé multi-pages (24 HTML) au lieu de single-page
2. Header/footer fond noir (cohérence, contraste 21:1)
3. Navigation 3 niveaux (menu > dropdown > sous-menu Monroe)
4. Breadcrumb sticky sous le header
5. Pas de framework JS (vanilla uniquement)
6. Chargement conditionnel Vimeo API (guard iframe)
7. CTA Reservation en button aria-disabled (billetterie non définie)
8. Video contact en pause par défaut (play au clic)

---

## Qualite responsive et code (audit Codex 2026-04-06)

**Score : 9/10** — Le 10/10 sera atteint quand le vrai domaine remplacera `DOMAINE` dans les 23 pages + sitemap.xml + robots.txt.

Corrections appliquees (9 commits) :
- Fix scroll horizontal iPhone logos partenaires (bug WebKit grid + aspect-ratio + flex)
- Safe-area complet sur tous les elements absolus (hero, back-to-top, bouton play, credits)
- Scrollbar masquee, menus scrollables, seuil tactile 44px
- Flip automatique sous-menu niveau 3 si debordement viewport
- Promesse video.play() geree pour iOS
- Nettoyage CSS mort (-189 lignes), variables inutilisees, doublons
- Cache-buster CSS sur les 24 pages
- Iframes Vimeo avec & echappes en &amp;

---

**Derniere mise a jour** : 2026-04-06
**Version** : 4.0.0

---
> Source: [Alexmacapple/swing-digital](https://github.com/Alexmacapple/swing-digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
