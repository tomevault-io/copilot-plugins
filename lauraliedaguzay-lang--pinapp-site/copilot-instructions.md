## pinapp-site

> ﻿# PINAPP INC. — RÈGLES PERMANENTES AGENT

﻿# PINAPP INC. — RÈGLES PERMANENTES AGENT
# Portée : tout le dépôt pinapp-site
# Cerveau stratégique : Claude | Générateur : Cursor | Déploiement : Hostinger SFTP

## ENTITÉ LÉGALE
- Pinapp Inc. = maison mère
- Pinapp Studio (pinapp.fr) = agence B2B premium — seul revenu actuel
- Auralis RH = SaaS IA RH pré-commercial
- Mémoire & Présence (memoireetpresence.fr) = digitalisation secteur funéraire
  · Vocabulaire M&P : hommage · transmission · présence · lien · mémoire
  · JAMAIS : mort · deuil · décès · funérailles dans les textes publics

## ASSOCIÉS — ÉGALITÉ TOTALE
- Lauralie : PDG Pinapp Inc. · technique · IA · automatisation
  · Zéro photo publique
- Michaël Bouilhac (Micha) : co-associé · imagerie · photo · vidéo · DA · Adobe · IA générative
  · Contact : micha@memoireetpresence.fr · WhatsApp uniquement
  · Zéro photo publique sans validation
- Traitement visuel égal partout — même espace · même hiérarchie

## STACK TECHNIQUE
- Claude = cerveau stratégique. Zéro code direct.
- Cursor = générateur de tout le code HTML/CSS/JS
- Hostinger SFTP (extension Cursor — Natizyskunk)
- n8n self-hosted Hostinger ~3€/mois
- Tally (formulaires · webhook natif)
- YouSign (signature électronique)
- Stripe (paiement)
- Notion (CRM · suivi · comptabilité)
- Gmail via n8n (emails automatiques)
- Buffer (social media)
- Claude API (IA · résumés · copy)
- WhatsApp Business (notifications Micha M&P)
- Budget max : 100€/mois · Claude Pro priorité (20€/mois)
- Netlify : SUPPRIMÉ définitivement

## NOTIFICATIONS N8N
- 🔷 Pinapp Studio — toutes notifications Pinapp
- 🌿 Mémoire & Présence — toutes notifications M&P → WhatsApp Micha
- Comptabilité séparée Notion : Pinapp vs M&P

## PALETTE — NUIT (défaut)
--bg:          #080d18
--bg-card:     rgba(255,255,255,0.04)
--text:        #f0f8ff
--text-muted:  rgba(240,248,255,0.55)
--accent:      #00e5b0
--accent-2:    #b388ff
--accent-3:    #7fffea
--accent-4:    #e040fb
--nacre:       #e8f4f8
--card-border: rgba(179,136,255,0.08)
--separator:   rgba(0,229,176,0.06)
--btn-cta-bg:  rgba(91,60,180,0.85)
--btn-cta-bd:  rgba(179,136,255,0.30)

## PALETTE — JOUR (eau Avatar 2)
--bg:          #0a2a2e
--bg-card:     rgba(10,60,65,0.60)
--text:        #e8fffd
--text-muted:  rgba(232,255,253,0.55)
--accent:      #7fffea
--accent-2:    #c4b5fd
--accent-3:    #ffffff
--accent-4:    #a78bfa
--nacre:       #e8f4f8
--btn-cta-bg:  rgba(109,40,217,0.80)
--btn-cta-bd:  rgba(196,181,253,0.30)

## TYPOGRAPHIE
- Titres : Clash Display 700 (Bunny Fonts)
- Corps  : Inter 300/400/500 (Bunny Fonts)
- Hero   : clamp(48px,8vw,104px) · letter-spacing -0.04em
- H2     : clamp(32px,5vw,56px) · letter-spacing -0.03em
- Label  : 11px · uppercase · letter-spacing 0.14em
- Shimmer : cyan→violet→nacré sur 1 mot-clé hero
- text-wrap: balance sur tous les titres

## FOND
- Nuit : #080d18 CSS pur · canvas particules 80pt 4 couleurs · liaisons <100px
- Jour : #0a2a2e CSS pur · 3 caustics blur(80px) mix-blend-mode:screen · rayons
- ZÉRO image de fond (bg-sombre/bg-clair supprimés définitivement)
- ZÉRO image Unsplash

## GLASSMORPHISM
- backdrop-filter: blur(20px)
- bg-card: rgba(255,255,255,0.04)
- border: 1px solid var(--card-border)
- border-radius: 20px
- pseudo ::before : dégradé cyan→violet→nacré opacity 0.03

## LAYOUT
- scroll-snap-type: y mandatory · sections 100dvh
- 1 idée / 1 section / 1 CTA maximum
- Nav latérale : points de section
- Mobile-first 390px iPhone Safari
- Vanilla JS uniquement · WCAG AA

## PRICING VALIDÉ
- Automatisation Messages   : 490€ HT
- Automatisation Facturation: 590€ HT
- Automatisation Devis      : 790€ HT
- Automatisation RDV        : 990€ HT
- IA sur-mesure             : 890€ HT
- Site Vitrine              : 1 290€ HT
- Système Complet           : dès 2 400€ HT · sur devis
- Pack Duo Lauralie+Micha   : dès 3 900€ HT · sur devis (~1600€ économisés)
- Formation 1               : 67€
- Formation 2               : 147€
- Formation 3               : 397€
- Pack Prompting            : 97€ (inclus dans 397€)

## AUTOMATISATIONS N8N (8 WORKFLOWS)
1. Diagnostic → Notion + email confirmation + notification 🔷
2. Devis → YouSign → Stripe → facture auto + notification 🔷
3. Paiement Stripe → accès formation + email bienvenue
4. Formation → séquence email J+1/J+3/J+7 + certificat + upsell J+14
5. Livraison → feedback Tally + avis Google J+7 + maintenance J+30
6. M&P contact → WhatsApp Micha 🌿 + email famille + Notion M&P
7. Veille hebdo → Claude API → Buffer → Newsletter Notion
8. Maintenance mensuel → rapport clients + relance impayés + export comptable

## RÈGLES ABSOLUES
INTERDIT :
- Frameworks JS (React, Vue, etc.)
- Images de fond (bg-sombre/bg-clair supprimés)
- Images Unsplash · photos de Lauralie ou Micha sans validation
- Champagne #C9A96E (remplacé par nacré #e8f4f8)
- Scroll horizontal · hover sur mobile
- box-shadow opacity > 0.12 · HTML > 500Ko
- CDN externes sauf Bunny Fonts
- Mention : mort · deuil · décès · funérailles (M&P)
- Prix TTC sans mention TVA

OBLIGATOIRE :
- Vanilla JS · Custom properties · WebP final · Lazy-load
- WCAG AA · Prix HT + TVA art. 293 B CGI
- HTTPS Hostinger (Let's Encrypt hPanel)
- Arnaud valide tout livrable visuel
- Thomas bloque toute livraison non conforme

## AGENTS ACTIFS
DA      : Arnaud (Apple/Tesla) · Théo · Raphaëlle · Nino · Viktor T.
Copy    : Élisa (StoryBrand) · Lucas (Brand)
Commer. : Alexis · Clara · Bruno · Mehdi · Chloé · Fiona · Omar
Formation: Samir · Rémi
Cognitif: Dr. Élise (TDAH) · Prof. Karim
Juridique: Lila · Maître Dubois · Maître Fontaine
Finance : Éric · Nathalie
Veille  : Nora · Axel
QA      : Thomas (arbitre · bloque si non conforme)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lauraliedaguzay-lang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
