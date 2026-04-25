## ugc-wintermate

> - **INTERDIT** : Utilisation de React Context pour la gestion d'état

## 1. Gestion d'état avec Zustand uniquement
- **INTERDIT** : Utilisation de React Context pour la gestion d'état
- **OBLIGATOIRE** : Utiliser Zustand pour tous les stores
- Structure recommandée pour les stores :
```ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthStore {
  user: User | null;
  isAuthenticated: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set, get) => ({
      user: null,
      isAuthenticated: false,
      login: async (credentials) => {
        // Logique d'authentification
      },
      logout: () => set({ user: null, isAuthenticated: false }),
    }),
    { name: 'auth-store' }
  )
);
```

## 2. Composants UI avec ShadCN exclusivement
- **OBLIGATOIRE** : Utiliser les composants ShadCN pour tous les éléments UI
- **INTERDIT** : Créer des composants UI custom sans base ShadCN
- Structure existante : [components/ui] (mdc:front/src/components/ui)
- Exemple d'utilisation dans [HomePage](mdc:front/src/pages/home.tsx)

## 3. Typage strict des réponses API
- **OBLIGATOIRE** : Typer toutes les réponses API avec les types de [shared/types](mdc:shared/types)
- **INTERDIT** : Utiliser `any` ou types non définis
- Exemple de typage correct dans [users.ts](mdc:front/src/api/users.ts)

## 4. Middlewares d'authentification
- **OBLIGATOIRE** : Utiliser des middlewares pour protéger les routes
- Structure recommandée :
```ts
import { useAuthStore } from '@/stores/auth';
import { Navigate } from 'react-router-dom';

export const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const isAuthenticated = useAuthStore((state) => state.isAuthenticated);
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return <>{children}</>;
};
```

## 5. Architecture des pages
- **OBLIGATOIRE** : Respecter le flow Screen → Hook → React Query défini dans [api-call.mdc](mdc:.cursor/rules/front/api-call.mdc)
- Structure type d'une page :
```tsx
export const ExamplePage = () => {
  // 1. Hooks de données (React Query)
  const { data, isLoading, error } = useGetData();
  
  // 2. Hooks de stores (Zustand)
  const { user, actions } = useStore();
  
  // 3. États locaux (React useState)
  const [localState, setLocalState] = useState();
  
  // 4. Logique métier
  const handleAction = async () => {
    // Logique
  };
  
  // 5. Rendu avec composants ShadCN
  return (
    <div className="p-4">
      <Button onClick={handleAction}>Action</Button>
    </div>
  );
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mathysFor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
