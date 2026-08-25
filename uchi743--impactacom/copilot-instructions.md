## impactacom

> > Ce fichier est lu automatiquement par Claude Code au démarrage.

# CLAUDE.md — Projet Impactacom

> Ce fichier est lu automatiquement par Claude Code au démarrage.
> Il définit les règles et le contexte du projet.

## Contexte

Site vitrine **Impactacom** pour **Corinne Barois**, coach en prise de parole en public basée à Paris. Le site présente ses formations (individuelles, collectives, training en ligne, programmes sur mesure), son approche pédagogique, des témoignages et un formulaire de contact.

- **URL de production** : https://impactacom.fr
- **Hébergement** : Cloudflare Pages (preview : impactacom.pages.dev)
- **Stack** : HTML statique + CSS + JS vanilla (aucun framework, aucun build)

## Stack & architecture

**Aucun build, aucune dépendance npm.** Site statique pur, déployé tel quel sur Cloudflare Pages.

```
impactacom/
├── index.html         # Toutes les pages (SPA avec .page)
├── _redirects         # Fallback SPA Cloudflare (toutes routes → index.html)
├── _headers           # Headers sécurité (CSP, HSTS, etc.)
├── css/style.css      # Tous les styles
├── js/main.js         # Router SPA + animations reveal + formulaire
├── assets/            # Images, médias (actuellement vide)
├── README.md
└── CLAUDE.md          # Ce fichier
```

### Pattern SPA

Toutes les "pages" sont des `<div class="page" id="page-XXX">` dans `index.html`.
La navigation passe par `showPage('nom-page')` qui :
- Cache toutes les pages (`.page.active` retiré)
- Affiche la page cible (`.active` ajoutée)
- Met à jour l'URL via `history.pushState` (ex: `/programmes`)
- Re-déclenche les animations reveal (`IntersectionObserver`)

Le bouton retour du navigateur fonctionne via `popstate`.

### Pages existantes (10)

- `home` — accueil
- `programmes` — présentation des 4 formations
- `formation-training` — training en ligne
- `formation-individuel` — accompagnement individuel
- `formation-collective` — formation collective (entreprises)
- `formation-custom` — programme sur mesure
- `apropos` — bio de Corinne
- `temoignages` — témoignages clients
- `contact` — formulaire + coordonnées
- `mentions` — mentions légales

## Design system

### Couleurs (CSS variables dans `:root`)
- `--violet: #2D1F4E` — couleur principale (texte titres, CTAs)
- `--violet-mid: #4A3270` — hover CTAs
- `--violet-soft: #6B4FA0` — liens secondaires
- `--plum: #7C3C6E` — accent
- `--lilac: #EDE8F7` — backgrounds doux
- `--white: #FFFFFF` / `--off-white: #F8F6FC`
- `--text: #1A1228` / `--muted: #7A6F8F`

### Typographie
- **Titres** : `Playfair Display` (serif, italique pour emphase)
- **Corps** : `Mulish` (sans-serif, 300-600)

### Règles de style
- Border-radius **petits** (3-8px, pas plus) — esthétique éditorial/corporate, pas moderne-startup
- Animations `data-r` (reveal) déclenchées par `IntersectionObserver` au scroll
- Pas d'emojis, pas d'icônes fantaisistes — registre premium/pro
- Séparateurs légers : `border: 1px solid var(--border)` = `rgba(45,31,78,0.10)`

## Contenu validé (NE PAS MODIFIER SANS DEMANDER)

- **Expérience de Corinne** : **25 ans** d'expérience (pas 20, pas 30)
- **Taux de satisfaction** : **92%**
- **Adresse** : 108 rue Michel-Ange — 75016 Paris
- **Téléphone** : 06 13 54 70 05
- **Email** : `cobarois@gmail.com`
- **Mentions légales Qualiopi** : bloc complet en pied de page conforme (certification, formation professionnelle)
- **Crédits** : Photographe (email à confirmer) + Webdesign original : Al Di Girolamo (https://aldigirolamo.fr)

## Règles d'édition

1. **Edits chirurgicaux uniquement.** Léo préfère les modifications minimales et ciblées. Ne JAMAIS refondre une section sans demander explicitement.
2. **Pas de refactor spontané** du CSS ou du JS sans demande claire.
3. **Grilles CSS** : si on utilise `grid-template-rows` avec des valeurs en `vw` (ex: `25vw`), garder cette logique — elle a été debuggée plusieurs fois.
4. **Formulaire de contact** : actuellement côté client uniquement (pas de backend). Le submit affiche un message de succès. Ne pas ajouter de traitement serveur sans demander.
5. **Vidéos** : intégrations YouTube via `playVideo(containerId, videoId)` — iframe injectée à la demande (pas de chargement initial).
6. **Fonts** : déjà chargées via Google Fonts avec `preconnect`. Pas besoin d'en ajouter.
7. **Préserver l'historique SPA** : toujours utiliser `showPage()` + `history.pushState` pour les navigations internes, jamais de lien `<a href="/xxx">` direct.

## Google Analytics 4 — NE PAS CASSER

Propriété GA4 `impactacom.fr` · ID de mesure **`G-E4RYFL7HVR`** · ID de flux `5783088316`.
Compte propriétaire : `barois@baroisetassocies.com`.

Chargement **conditionné au consentement** (Consent Mode v2) : snippet inline dans `index.html`
(`gtag('consent','default', … denied)`), puis `loadGA()` dans `js/main.js` injecte `gtag.js`
uniquement si le visiteur a accepté le bandeau cookies (`localStorage.impactacom_consent`).

Trois pièges déjà rencontrés — deux ont causé **zéro donnée collectée pendant des mois** :

1. **`gtag('js', new Date())` est obligatoire** dans `loadGA()`, avant `gtag('config', …)`.
   Sans lui, `gtag.js` se charge et bootstrappe le conteneur mais **n'envoie aucun hit**
   et ne pose aucun cookie `_ga` — sans la moindre erreur en console.
2. **CSP `connect-src` doit autoriser `https://*.google-analytics.com`** (dans `_headers`).
   GA4 envoie le trafic européen sur `region1.google-analytics.com` : n'autoriser que
   l'hôte exact `www.google-analytics.com` bloque silencieusement toute la collecte.
3. Les `page_view` SPA sont envoyés manuellement depuis `showPage()` (`config` utilise
   `send_page_view: false`). Ils sont gardés par `window.__gaLoaded` : sans ce garde, le
   `page_view` du premier affichage part avant `config` et GA le jette.

Vérifier une modif GA en prod (console du navigateur, après un hard reload) :
`performance.getEntriesByType('resource').filter(e=>/g\/collect/.test(e.name)).length` > 0
et `/(^|;\s*)_ga=/.test(document.cookie) === true`.

## Déploiement

Cloudflare Pages déploie automatiquement la branche `main` :
- Pas de commande de build (site statique pur)
- `_redirects` configure le fallback SPA : `/* /index.html 200` pour que toutes les routes (`/programmes`, `/contact`, etc.) servent `index.html` au refresh
- `_headers` définit les en-têtes de sécurité (CSP, HSTS, X-Frame-Options, etc.)
- Preview : `impactacom.pages.dev` · Prod : `impactacom.fr`

Pour déployer : `git push origin main`, Cloudflare déploie en ~30s.

Preview locale (avec fallback SPA comme en prod) : `python .serve.py` puis http://localhost:8080

## À faire / TODO

- [x] Email de contact confirmé : `barois@impactacom.fr`
- [x] Email photographe confirmé : `contact@studio-michelange.com`
- [ ] Décider si le formulaire de contact doit avoir un vrai backend (Cloudflare Pages Functions ou Formspree)
- [ ] Ajouter les assets images dans `/assets/` si besoin (actuellement tout est en SVG inline ou via YouTube)

---
> Source: [Uchi743/Impactacom](https://github.com/Uchi743/Impactacom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
