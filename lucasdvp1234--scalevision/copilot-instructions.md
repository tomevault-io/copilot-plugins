## scalevision

> Site de l'agence Scalevision — agence de création de contenu basée à Paris (UGC, micro-trottoir, influence). Le site accompagne aussi la vente de webinaires/conférences pour les créateurs de contenu.

# CLAUDE.md — Scalevision Site

## Projet
Site de l'agence Scalevision — agence de création de contenu basée à Paris (UGC, micro-trottoir, influence). Le site accompagne aussi la vente de webinaires/conférences pour les créateurs de contenu.

## Stack
- HTML/CSS/JS vanilla (pas de framework)
- Tailwind CSS (config dans `tailwind.config.js`)
- Assets images hébergées sur S3 : `https://ugcademie.s3.eu-north-1.amazonaws.com/`
- Formulaires d'inscription via **Tally** (embed iframe)

## Fichiers clés
| Fichier | Rôle |
|---|---|
| `index.html` | Landing page principale Scalevision |
| `webinar.html` | Landing webinaire **version A** (B-test) |
| `webinar-b.html` | Landing webinaire **version B** (B-test) |
| `felicitation.html` | Page de confirmation post-inscription |
| `cgv.html` / `cgv---academie.html` | CGV |
| `mentions-legales-ugcademie.html` | Mentions légales |
| `politique-de-confidentialite-ugcademie.html` | Politique de confidentialité |

## B-Test webinaire (session du 23 mars 2026)
Deux versions d'une landing pour le webinaire du **12 avril 2026 à 20H30** (live depuis l'agence Paris).

### Version A — `webinar.html`
- Design bleu roi `#251EAF` + rose `#FF3C8E` (palette brand originale)
- Fonts : Bebas Neue (display) + Playfair Display (serif)
- Layout : hero centré, timeline verticale, cards speakers, scale tiers, form Tally
- Formulaire Tally `vGBWP4` intégré côté droit avec panneau info à gauche

### Version B — `webinar-b.html`
- Design dark `#08080F` + rose + gold `#FFD700` + vert `#00FF7F`
- Fonts : Bebas Neue + Playfair Display + Inter
- Layout différent : plus éditorial, grid speakers différente, même Tally
- Formulaire Tally `vGBWP4` intégré (embed complet)

### Différences clés A vs B (pour le B-test)
- A = fond bleu (couleurs brand) / B = fond noir (plus premium/éditorial)
- A = form dans wrapper 2 colonnes (info gauche + Tally droite) / B = form fullwidth avec header
- Même contenu, même Tally, hiérarchie visuelle légèrement différente

## Contenu du webinaire 12 Avril
- **Objectif affiché :** 10 000€/mois (fourchette réelle : 7 500€ – 13 000€)
- **Format :** Live depuis l'agence Paris, pas de replay, places limitées
- **Giveaway :** DJI Osmo Pocket 3 tiré au sort en direct à la fin

### Intervenants
| Nom | Statut | Thème |
|---|---|---|
| **YesCyril** | Révélé | 5M+ abonnés cumulés, 40K€/mois — comment fait-il ? |
| Mélanie | Mystère (créatrice #1) | 10 000 abonnés en 6 mois |
| Romane Blucheau | Mystère (créatrice #2) | Signer des contrats avec 3K abonnés |
| Yolande / Famille ACLA | Mystère (créatrice #3) | 10 000€/mois réguliers |

> Les 3 créatrices ne sont PAS révélées sur la landing — uniquement leurs thèmes.

### Structure du live
1. Ouverture — Présentation Constantin + Scalevision + pourquoi ce live
2. Partie 1 — Les 8 piliers des 10K€/mois
3. Partie 2 — Interviews de 3 créatrices (contrats réels montrés en direct)
4. Partie 3 — YesCyril exclusif
5. Bonus + Q&A + Tirage Osmo Pocket

### Piliers enseignés
1. Contenu 2. Percer/Abonnés 3. Routine 4. Prospection 5. Vente 6. Closing 7. Récurrence 8. Scaling (10K → 50K€)

### Scale tiers (visual sur les deux pages)
- 300 abonnés → 3–4K€/mois
- 3–4K abonnés → 6–7K€/mois
- 4K+ abonnés → 10K€+ illimité

## Tally
- **Form ID :** `vGBWP4`
- **URL :** https://tally.so/r/vGBWP4
- Embed params recommandés : `alignLeft=1&hideTitle=1&transparentBackground=1&dynamicHeight=1`

## Design system (version A — brand)
```css
--bg: #251EAF
--bg-dark: #1a1580
--white: #FFF
--pink: #FF3C8E
--pink-light: #FF6BA8
--gold: #FFD700
--green: #00FF88
--font-display: 'Bebas Neue'
--font-serif: 'Playfair Display'
```

## Assets images (S3)
```
const.jpeg     → Constantin (CEO)
gaelle.jpg     → Lory.hts (ancienne intervenante, plus utilisée sur webinar)
remi.jpg / elea.jpg / sharleyne.jpg / yamine.jpg → créateurs génériques
osmo.png       → DJI Osmo Pocket 3 (dans assets/img/osmo.png)
```

## Tracking Meta (Pixel)
- **Pixel ID :** `2290938654583850`
- Présent dans le `<head>` de toutes les pages de conversion, avant le tag Google Fonts

| Page | Événements |
|---|---|
| `webinar.html` | `PageView` |
| `webinar-b.html` | `PageView` |
| `felicitation.html` | `PageView` + `LeadOptinWebi` + `Lead` |

## Notes importantes
- **Ne pas dévoiler** les 3 créatrices (Mélanie, Romane, Yolande/ACLA) sur les pages — uniquement afficher leurs thèmes
- La contrainte "3000 abonnés minimum" a été **supprimée** (conférence ouverte à tous)
- L'ancien co-animateur Lory.hts (290K abonnés) n'est **plus mentionné** dans le webinar 12 avril
- Copyright → 2026

---
> Source: [lucasDVP1234/scalevision](https://github.com/lucasDVP1234/scalevision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
