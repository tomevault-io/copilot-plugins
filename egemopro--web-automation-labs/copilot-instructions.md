## web-automation-labs

> > **VERSION:** Ce fichier est le point d'entrée automatique pour toute IA.

# 🤖 AGENTS.md - Point d'Entrée Central

> **VERSION:** Ce fichier est le point d'entrée automatique pour toute IA.
> Pour les détails complets, lire `MEGA_BRAIN.md`.

---

## 🎯 RÉSUMÉ EXÉCUTIF

| Élément | Valeur |
|---------|--------|
| **Type** | Orchestrateur d'automations web |
| **Méthode** | Chrome DevTools MCP + DOM Monitoring |
| **Skills** | Instagram, Facebook, Meta AI, Reddit, Chariow Store, Web (générique) |
| **Comportement** | Humain-like avec délais |

---

## 📁 STRUCTURE

```
labs/
├── AGENTS.md              ← CE FICHIER (alias)
├── MEGA_BRAIN.md          ← Documentation complète
├── start-chrome.sh        ← 🚀 Chrome DevTools MCP
├── instagram/SKILL.md     → Automatisation Instagram
├── facebook/              → Automatisation Facebook
├── meta-ai/              → Automatisation Meta AI
├── reddit/                → Automatisation Reddit ⭐
├── chariow-automation/    → Automatisation Chariow 🛒
└── web-automation-agent/ → Framework générique
```

---

## 🚀 DÉMARRAGE RAPIDE

```bash
# 1. Lancer Chrome (OBLIGATOIRE avant toute automation)
./start-chrome.sh start

# 2. Les skills utilisent automatiquement Chrome + MCP
```

## 🔗 ROUTAGE RAPIDE

| Plateforme | Action |
|------------|--------|
| **Démarrer Chrome** | `./start-chrome.sh start` |
| Instagram | → `instagram/SKILL.md` |
| Facebook | → `facebook/scripts/` |
| Meta AI | → `meta-ai/SKILL.md` |
| Reddit | → `reddit/SKILL.md` |
| Chariow Store | → `chariow-automation/SKILL.md` |
| Autre site | → `web-automation-agent/SKILL.md` |

---

## ⚡ COMMANDE DE DÉMARRAGE

```bash
# 1. Lancer Chrome (OBLIGATOIRE)
./start-chrome.sh start

# 2. Lire le contexte projet
memory_list(scope="project")

# 3. Vérifier le navigateur
list_pages()

# 4. Aller à la cible
navigate_page(url="https://...")

// 5. Snapshot initial
take_snapshot()

// 6. Injecter tracking
evaluate_script(`/* event_listener_monitor.js */`)

// 7. Vérifier
DOMMonitor.getStats()
```

---

## 📋 ACTIONS SUPPORTÉES

### Instagram
- ✅ Likes, Comments, Follows, DMs
- ✅ Stories, Reels, Explore
- ✅ Search, Profile visits

### Facebook
- ✅ Posts, Reactions, Comments
- ✅ Messenger (via scripts)
- ✅ Pages, Groups

### Meta AI
- ✅ Chat avec IA
- ✅ Génération d'images
- ✅ Génération de vidéos
- ✅ Vibes (mode créatif)

### Reddit
- ✅ Posts, Comments, Upvotes
- ✅ Subreddits, Search
- ✅ Messaging, Chat
- ✅ Profile, Notifications

### Chariow Store
- ✅ Products (CRUD)
- ✅ Sales, Customers
- ✅ Marketing (Discounts, Campaigns)
- ✅ Automations, Analytics
- ✅ Settings, Appearance

### Web (Générique)
- ✅ Navigation, Forms, Clicks
- ✅ E-commerce, Dashboards
- ✅ Tout site avec interface browser

---

## 🚨 RÈGLES DE SÉCURITÉ

| Règle | Limite |
|-------|--------|
| Likes Instagram | 200-400/jour |
| Follows Instagram | 50-100/jour |
| Comments | 20-40/jour |
| DMs | 20-50/jour |
| Délai minimum | 1-5s entre actions |

---

## 📝 COMPTES CONFIGURÉS

### Instagram
- **Compte:** @challengetechformation
- **URL:** https://www.instagram.com/
- **Followers:** 83

### Meta AI
- **Compte:** challengetechformation
- **URL:** https://www.meta.ai/

### Chariow Store
- **Store ID:** store_w3d7gm69kpai
- **URL:** https://ifbrssgd.mychariow.shop/
- **Admin:** https://app.chariow.com/stores/store_w3d7gm69kpai/

---

## 🔄 DOCUMENTATION

| Fichier | Usage |
|---------|-------|
| `MEGA_BRAIN.md` | Documentation complète |
| `AGENTS.md` | Ce fichier (alias) |
| `[skill]/SKILL.md` | Méthodologie détaillée |
| `[skill]/ui_inventory.md` | Éléments UI découverts |
| `[skill]/store_reference.md` | Structure/navigation |

---

## ✅ CHECKLIST OBLIGATOIRE

```
□ ./start-chrome.sh start    ← OBLIGATOIRE avant tout
□ Lire AGENTS.md ou MEGA_BRAIN.md
□ Charger le skill approprié
□ Initialiser DOM Monitor
□ Respecter les délais humains
□ Documenter chaque découverte
□ Mettre à jour la mémoire projet
□ Résumer pour l'utilisateur
```

---

**Pour détails complets → Voir `MEGA_BRAIN.md`**

---
> Source: [EgemoPro/web-automation-labs](https://github.com/EgemoPro/web-automation-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
