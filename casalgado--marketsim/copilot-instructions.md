## marketsim

> Classroom stock market simulation for ~20 students. Teacher acts as broker.

# CLAUDE.md — MarketSim

## Project Overview

Classroom stock market simulation for ~20 students. Teacher acts as broker.
Students view portfolios and leaderboard read-only. Built with Vue 3 + Firebase (no Cloud Functions).
All UI text is in English. Dates and currency use Colombian locale (es-CO / COP).

## Commands

```bash
npm run dev           # Start emulators + frontend (always use this)
npm run seed:dev      # Seed emulators once (then stop dev, data auto-saves on exit)
npm run deploy        # Build + deploy hosting + firestore rules/indexes
npm run deploy:rules  # Deploy firestore rules only
```

## Architecture

### Stack
- **Frontend**: Vue 3 (Composition API, script setup), Vite, Pinia, Vue Router, Tailwind CSS v4
- **Database**: Firestore (no Cloud Functions)
- **Auth**: Firebase Auth (email/password), role stored in Firestore users/{uid}
- **Icons**: @phosphor-icons/vue
- **CSV**: PapaParse (trade imports + Google Sheets price fetching)

### Store-First Architecture
**CRITICAL: Vue components must NEVER call Firestore services directly.**

#### Rules
1. **NEVER** import services into Vue components
2. **ALWAYS** use Pinia stores for all data access
3. **ALWAYS** use `computed()` for reactive store data in components (never `ref()` for store data)
4. Stores own: Firestore subscriptions, caching, cleanup, single source of truth

#### Data Flow
```
Component → Store → Service → Firestore
```

#### Pattern Example

❌ **WRONG:**
```javascript
// In a Vue component
import { stockService } from '@/services/stockService'
const stocks = ref([])
onMounted(() => stockService.getStocks().then(data => stocks.value = data))
```

✅ **CORRECT:**
```javascript
// In a Vue component
import { useStocksStore } from '@/stores/stocks'
const stocksStore = useStocksStore()
const stocks = computed(() => stocksStore.stocks)
onMounted(() => stocksStore.subscribe())
```

### Store Pattern
All Pinia stores follow this structure:
```javascript
export const useXStore = defineStore('x', () => {
  const items = ref([])
  const loading = ref(false)
  const error = ref(null)

  const fetchItems = async () => {
    loading.value = true
    error.value = null
    try {
      // service call
    } catch (err) {
      error.value = err.message
      throw err
    } finally {
      loading.value = false
    }
  }

  return { items, loading, error, fetchItems }
})
```

### Roles
- `admin` (teacher): full read/write via Firestore rules
- `student`: read-only, enforced by Firestore rules
- Role stored as `role` field in `users/{uid}` — checked via `get()` in security rules

### Models
All Firestore data flows through model classes. Never use plain objects from Firestore directly.
- `BaseModel` — fromFirestore(), toFirestore(), validate(), dateFields
- `User`, `Stock`, `Trade`, `Portfolio` extend BaseModel
- Import from `@/models`

## Directory Structure

```
frontend/src/
├── assets/styles/     # CSS design system (palette, base, layout, cards, components, forms, utilities)
├── components/        # Reusable components (never call services)
├── composables/       # useAuth, useSidebar, usePortfolio
├── firebase/          # Firebase init (auth, db exports)
├── layouts/           # AppLayout, BaseAuthLayout, AppHeader, AppSidebar
├── models/            # BaseModel, User, Stock, Trade, Portfolio
├── router/            # Routes + guards
├── services/          # Firestore services (called only by stores)
├── stores/            # Pinia stores (auth, stocks, portfolios)
└── views/             # Page components + admin/
```

## Vue File Convention
- Order: `<script setup>` → `<template>` → `<style scoped>`
- Always use `<script setup>`
- Always use `<style scoped>`

## UI / Code Rules

### Do Not
- Use icons or emojis in code unless explicitly requested
- Use `cursor-not-allowed` — use `cursor-default` instead
- Import services in Vue components
- Use `ref()` for data that comes from a store — use `computed()`
- Commit without explicit user permission
- Suggest or run `npm run lint` or `npm run build` unprompted

### Do
- Follow store-first architecture strictly
- Use model classes for all Firestore data
- Use `try/catch/finally` in all async store actions
- Keep components thin — logic belongs in stores and composables

## Development Notes
- Assume the dev server is always running
- Emulators run on: Auth :9099, Firestore :8080, Hosting :5000
- `VITE_USE_EMULATORS=true` in `.env.local` connects to emulators
- Firestore security rules: authenticated users read everything, only admin writes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/casalgado)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/casalgado)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
