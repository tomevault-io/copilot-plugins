## shareapp

> Application web de partage d'écran et audio en temps réel construite avec:

# Cursor Rules pour Screen Share App

## Contexte du Projet
Application web de partage d'écran et audio en temps réel construite avec:
- **Frontend**: React 18 + TypeScript + Tailwind CSS
- **Build Tool**: Vite
- **Communication**: WebRTC (P2P) + WebSocket (signalisation via Socket.io)
- **State Management**: Zustand
- **Hosting**: Firebase Hosting (pour HTTPS et déploiement)
- **Backend**: Serveur Node.js séparé pour la signalisation WebRTC

## Structure du Projet
```
src/
├── frontend/          # Code React/TypeScript
│   ├── components/    # Composants UI réutilisables
│   ├── hooks/         # Hooks React personnalisés
│   ├── stores/        # Stores Zustand
│   └── utils/         # Utilitaires
└── shared/            # Types partagés
signaling-server/      # Serveur de signalisation WebSocket
```

## Règles de Code

### TypeScript/React
- Utiliser TypeScript strict mode
- Préférer les fonctionnalités React (hooks) aux classes
- Utiliser `const` pour les composants fonctionnels
- Nommer les composants en PascalCase
- Utiliser des interfaces pour les types (pas de `type` sauf pour unions)
- Préférer `interface` pour les props de composants
- Utiliser des imports absolus avec les alias `@/` et `@/shared/`
- Grouper les imports: React, puis bibliothèques externes, puis imports locaux

### Styling
- Utiliser Tailwind CSS avec les classes utilitaires
- Utiliser la fonction `cn()` pour combiner les classes conditionnellement
- Suivre le système de design avec les variables CSS (--primary, --background, etc.)
- Préférer les classes Tailwind aux styles inline
- Utiliser les composants shadcn/ui comme base (quand disponibles)

### WebRTC
- Gérer proprement la fermeture des connexions RTCPeerConnection
- Nettoyer les MediaStream tracks lors de la déconnexion
- Utiliser les serveurs STUN/TURN appropriés
- Implémenter la reconnexion automatique en cas d'échec
- Logger les erreurs WebRTC pour le débogage

### Gestion d'État (Zustand)
- Créer des stores séparés par domaine fonctionnel
- Utiliser des actions nommées clairement (setXxx, addXxx, removeXxx)
- Éviter les mutations directes de l'état
- Préférer les sélecteurs pour accéder à l'état

## Conventions de Nommage

### Fichiers
- Composants React: `PascalCase.tsx` (ex: `ScreenShare.tsx`)
- Hooks: `camelCase.ts` avec préfixe `use` (ex: `useWebRTC.ts`)
- Utilitaires: `camelCase.ts` (ex: `cn.ts`, `webrtc.ts`)
- Types: `camelCase.ts` ou dans `types/index.ts`

### Variables/Fonctions
- TypeScript: `camelCase` pour variables/fonctions, `PascalCase` pour types/interfaces

## Bonnes Pratiques

### Performance
- Utiliser `React.memo()` pour les composants lourds qui se re-rendent souvent
- Éviter les re-renders inutiles avec des sélecteurs Zustand appropriés
- Utiliser `useCallback` et `useMemo` quand nécessaire
- Optimiser les captures d'écran (frame rate adaptatif)
- Nettoyer les MediaStream tracks lors du démontage des composants
- Utiliser `useRef` pour stocker les streams et éviter les re-créations
- Optimiser le build Vite pour la production (minification, tree-shaking)

### Sécurité
- Ne jamais exposer de clés API ou secrets dans le code
- Valider toutes les entrées utilisateur
- Utiliser HTTPS/WSS pour les connexions réseau
- Implémenter l'authentification pour les salons privés
- Ne pas commit les tokens Firebase ou clés d'API

### Gestion d'Erreurs
- Toujours gérer les erreurs avec try/catch pour les opérations async
- Afficher des messages d'erreur utilisateur-friendly avec des explications claires
- Logger les erreurs pour le débogage (console.error avec détails)
- Implémenter la reconnexion automatique
- Gérer les erreurs de permissions pour `getDisplayMedia`

### Tests
- Tester les composants critiques (WebRTC, capture d'écran)
- Tester avec différentes conditions réseau (lente, instable)
- Tester sur différents navigateurs (Chrome, Firefox, Safari)
- Tester sur HTTPS avant le déploiement (Firebase Hosting)

## Commandes Utiles

### Développement
- `npm run dev` - Lancer le serveur de développement Vite (port 1420 par défaut)
- `npm run build` - Build de production (génère le dossier `dist/`)
- `npm run preview` - Prévisualiser le build de production localement
- `cd signaling-server && npm start` - Lancer le serveur de signalisation (port 3001)

### Déploiement Firebase
- `firebase login` - Se connecter à Firebase
- `firebase init` - Initialiser Firebase (sélectionner Hosting uniquement)
- `firebase deploy --only hosting` - Déployer l'application sur Firebase Hosting
- `firebase serve` - Tester localement avec Firebase Hosting

## Fichiers Importants

- `vite.config.ts` - Configuration Vite (port, alias, plugins)
- `tailwind.config.js` - Configuration Tailwind CSS
- `package.json` - Dépendances Node.js frontend
- `signaling-server/package.json` - Dépendances serveur de signalisation
- `signaling-server/index.js` - Serveur WebSocket de signalisation (Socket.io)
- `firebase.json` - Configuration Firebase Hosting
- `.firebaserc` - Projet Firebase sélectionné (ne pas commit)
- `.env.example` - Template des variables d'environnement

## Notes Spécifiques au Projet

### WebRTC
- Le serveur de signalisation doit être lancé séparément (port 3001)
- Utilise Socket.io pour la communication en temps réel
- Serveur de signalisation déployé séparément (VPS, Railway, Render, etc.)
- Firebase Hosting ne peut pas héberger un serveur WebSocket directement

### Firebase
- **Hosting** : Utilisé pour héberger le frontend React (HTTPS automatique)
- **Configuration** : `firebase.json` pointe vers `dist/` (build Vite)
- **Déploiement** : `firebase deploy --only hosting` après `npm run build`
- **Important** : Firebase Hosting fournit HTTPS nécessaire pour `getDisplayMedia`
- Ne pas commit `.firebaserc` ou les fichiers de debug Firebase
- Utiliser des variables d'environnement pour les URLs de production

### Capture d'Écran
- Utilise l'API Web `getDisplayMedia` standard des navigateurs
- Fonctionne sur Chrome, Firefox, Edge et Safari (avec permissions HTTPS)
- L'utilisateur sélectionne l'écran/fenêtre à partager via l'interface native du navigateur
- **Nécessite HTTPS en production** : Firebase Hosting fournit HTTPS automatiquement
- Gérer les erreurs de permissions avec des messages utilisateur-friendly
- Nettoyer les MediaStream tracks lors de l'arrêt du partage

### Audio
- Utiliser Web Audio API pour le traitement audio
- Implémenter AEC (Acoustic Echo Cancellation) et noise reduction
- Codec Opus recommandé pour la qualité

## À Faire (TODOs)

- [ ] Intégrer le streaming vidéo complet via WebRTC
- [ ] Implémenter la capture audio système
- [ ] Ajouter la gestion d'erreurs et reconnexion automatique
- [ ] Implémenter la qualité adaptative selon la bande passante
- [ ] Ajouter les statistiques réseau en temps réel
- [ ] Implémenter l'authentification et les salons privés
- [x] Configurer Firebase Hosting pour HTTPS (fait)
- [ ] Déployer le serveur de signalisation sur un VPS/service cloud
- [ ] Configurer les variables d'environnement pour production
- [ ] Ajouter les tests E2E pour WebRTC

## Éviter

- ❌ Ne pas commit les fichiers sensibles (.env, clés API, tokens Firebase)
- ❌ Ne pas hardcoder les URLs de serveurs (utiliser des variables d'environnement)
- ❌ Ne pas oublier de nettoyer les ressources (streams, connexions WebRTC)
- ❌ Ne pas utiliser `any` en TypeScript (utiliser `unknown` si nécessaire)
- ❌ Ne pas ignorer les erreurs de permissions pour `getDisplayMedia`
- ❌ Ne pas oublier de tester sur HTTPS avant le déploiement
- ❌ Ne pas commit `.firebaserc` ou les fichiers de debug Firebase
- ❌ Ne pas déployer sans avoir testé le build local (`npm run build`)
- ❌ Ne pas oublier de configurer les variables d'environnement en production

## Ressources

- Documentation WebRTC: https://webrtc.org/
- Documentation React: https://react.dev/
- Documentation Zustand: https://zustand-demo.pmnd.rs/
- MDN getDisplayMedia: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia
- Firebase Hosting: https://firebase.google.com/docs/hosting
- Socket.io: https://socket.io/docs/v4/
- Vite: https://vitejs.dev/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AziziYoussef) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
