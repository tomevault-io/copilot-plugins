## saas-ia

> Ce projet utilise la template **Sneat MUI Next.js Admin Template v3.0.0** (payée plusieurs centaines d'euros).

# 🎨 RÈGLE FRONTEND : Template Sneat MUI v3.0.0

## ⚠️ RÈGLE ABSOLUE - UTILISATION OBLIGATOIRE DE LA TEMPLATE PREMIUM

Ce projet utilise la template **Sneat MUI Next.js Admin Template v3.0.0** (payée plusieurs centaines d'euros).

### 📍 Localisation de la Template
- **Source** : `C:\Users\ibzpc\Git\SaaS-IA\sneat-mui-nextjs-admin-template-v3.0.0` (template originale complète)
- **Utilisation** : `mvp/frontend/` (copie adaptée pour le MVP)

### ✅ RÈGLES OBLIGATOIRES

1. **NE JAMAIS créer de composants UI from scratch**
   - Toujours chercher d'abord dans la template Sneat
   - Toujours utiliser les composants Material-UI de Sneat
   - Toujours réutiliser les layouts existants

2. **TOUJOURS adapter au lieu de recréer**
   - Pages d'auth (login, register) : adapter les pages Sneat existantes
   - Dashboard : utiliser AdminLayout de Sneat
   - Formulaires : utiliser TextField, Button, Card de Sneat
   - Tables : utiliser Table, TableRow de Sneat
   - Modals : utiliser Dialog de Sneat

3. **AVANT de coder un nouveau composant frontend**
   - ❓ Est-ce que Sneat a déjà ce composant ?
   - ❓ Est-ce que Material-UI (inclus dans Sneat) a ce composant ?
   - ❓ Puis-je adapter un composant existant ?
   - ✅ Si OUI à l'une de ces questions : RÉUTILISER !
   - ❌ Si NON à toutes : Demander confirmation avant de créer

4. **CHEMIN D'IMPORT OBLIGATOIRE**
```typescript
// ✅ BON : Import depuis Sneat
import { Button } from '@mui/material'
import AdminLayout from '@/layouts/AdminLayout'

// ❌ MAUVAIS : Créer un nouveau composant
import Button from '@/components/custom-button'
```

5. **RESSOURCES DISPONIBLES DANS SNEAT**
   - 🎨 100+ pages déjà designées
   - 🧩 500+ composants Material-UI configurés
   - 📱 Responsive design professionnel
   - 🌓 Dark mode intégré
   - ✅ Production-ready

### 🚫 INTERDICTIONS

- ❌ Créer des composants UI custom si Sneat les a
- ❌ Réinventer les layouts (sidebar, header, footer)
- ❌ Recoder les pages d'authentification
- ❌ Utiliser d'autres bibliothèques UI (Bootstrap, Ant Design, etc.)
- ❌ Modifier le theme Sneat sans validation

### 💡 EN CAS DE DOUTE

**Avant de coder quoi que ce soit en frontend, DEMANDER :**
> "Est-ce que ce composant existe déjà dans la template Sneat ?"

### 📚 DOCUMENTATION SNEAT

- **Dossier** : `C:\Users\ibzpc\Git\SaaS-IA\sneat-mui-nextjs-admin-template-v3.0.0`
- **Composants** : `sneat-mui-nextjs-admin-template-v3.0.0/src/components/`
- **Layouts** : `sneat-mui-nextjs-admin-template-v3.0.0/src/layouts/`
- **Pages exemples** : `sneat-mui-nextjs-admin-template-v3.0.0/src/pages/`
- **Config theme** : `sneat-mui-nextjs-admin-template-v3.0.0/src/configs/themeConfig.ts`

---

**Cette règle est PERMANENTE et s'applique à tous les développeurs et assistants IA travaillant sur ce projet.**

**Dernière mise à jour** : 2025-11-13
**Auteur** : @benziane

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benziane) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
