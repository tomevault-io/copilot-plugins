## linkart

> > **Date**: 2025-12-03

# Module : Frontend Navigation

> **Version**: v1.0
> **Date**: 2025-12-03
> **Parent**: `.cursor/rules/roles/frontend-developer.mdc`
> **Usage**: Chargé dynamiquement pour la navigation et le routing.

---

## 🧭 Navigation & Routing (OBLIGATOIRE)

**Règle** : Séparer Wrappers (Logique Nav) et Screens (UI).

### Architecture
- **Wrappers** (`src/screens/`) : Connectés à React Navigation (`useNavigation`, `useRoute`).
- **Screens UI** (`src/features/*/screens/`) : Reçoivent actions via props.

### Patterns
- **Deep Linking** : Configuration centralisée pour URLs.
- **Guards** : Middleware d'authentification (Redirection si non connecté).
- **Persistence** : Persister l'état de navigation si nécessaire.
- **Typage** : Routes typées (NativeStackScreenProps).

**Exemple Wrapper** :
```typescript
export function PaymentScreen() {
  const nav = useNavigation();
  const route = useRoute();
  return <PaymentScreenFigma onBack={nav.goBack} data={route.params} />;
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LordThiouk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
