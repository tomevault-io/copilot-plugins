## peso

> - Respecter strictement la structure monorepo : `backend/`, `frontend/`, `mobile-app/`, `infra/`

# Cursor Rules - Peso SaaS Monorepo
# Règles spécifiques par namespace technique pour maintenir la cohérence du projet

## 🏗️ ARCHITECTURE GÉNÉRALE

### Structure Monorepo
- Respecter strictement la structure monorepo : `backend/`, `frontend/`, `mobile-app/`, `infra/`
- Chaque module doit être autonome avec ses propres dépendances
- Utiliser des chemins relatifs pour les imports entre modules
- Maintenir la séparation claire des responsabilités

### Conventions de Nommage
- **Fichiers Python** : snake_case (ex: `user_service.py`, `weight_entry.py`)
- **Fichiers JavaScript/Vue** : camelCase ou PascalCase (ex: `userService.js`, `WeightEntry.vue`)
- **Dossiers** : kebab-case (ex: `mobile-app/`, `weight-entries/`)
- **Modales** : Toujours dans le dossier `components/Modal/` (ex: `EditWeightEntryModal.vue`)
- **Variables** : snake_case en Python, camelCase en JavaScript
- **Constantes** : UPPER_SNAKE_CASE
- **Classes** : PascalCase dans tous les langages

### Documentation
- Tous les commentaires en **anglais** (comme spécifié dans les règles utilisateur)
- Documenter toutes les fonctions publiques avec docstrings
- Maintenir les README.md à jour dans chaque dossier
- Utiliser des emojis dans la documentation pour la lisibilité

---

## 🐍 BACKEND (FastAPI/Python)

### Structure du Code
```python
# Imports dans cet ordre :
# 1. Standard library
import os
from typing import Optional

# 2. Third-party packages
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

# 3. Local imports
from app.core.database import get_db
from app.models.user import User
from app.schemas.user import UserCreate, UserResponse
```

### Modèles SQLAlchemy
```python
# Toujours hériter de Base
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.core.database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    # Toujours ajouter des index sur les champs de recherche fréquente
    # Toujours définir nullable=False pour les champs obligatoires
```

### Schémas Pydantic
```python
# Utiliser des schémas séparés pour Create, Update, Response
from pydantic import BaseModel, EmailStr
from datetime import datetime
from typing import Optional

class UserCreate(BaseModel):
    email: EmailStr
    password: str
    name: str

class UserUpdate(BaseModel):
    name: Optional[str] = None
    # Pour les updates, tous les champs doivent être Optional

class UserResponse(BaseModel):
    id: int
    email: str
    name: str
    created_at: datetime
    
    class Config:
        from_attributes = True  # Pour SQLAlchemy 2.0
```

### Services
```python
# Logique métier dans les services, pas dans les routes
class UserService:
    def __init__(self, db: Session):
        self.db = db
    
    def create_user(self, user_data: UserCreate) -> User:
        # Validation métier ici
        # Logique de création
        pass
    
    def get_user_by_email(self, email: str) -> Optional[User]:
        return self.db.query(User).filter(User.email == email).first()
```

### Routes API
```python
# Utiliser des préfixes cohérents
router = APIRouter(prefix="/api/v1/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=201)
async def create_user(
    user_data: UserCreate,
    db: Session = Depends(get_db)
):
    # Validation et création via service
    pass

@router.get("/me", response_model=UserResponse)
async def get_current_user(
    current_user: User = Depends(get_current_user)
):
    # Utiliser les dépendances pour l'authentification
    pass
```

### Gestion d'Erreurs
```python
# Toujours utiliser des exceptions HTTP appropriées
from fastapi import HTTPException

def get_user_or_404(user_id: int, db: Session) -> User:
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Tests
```python
# Structure des tests
import pytest
from fastapi.testclient import TestClient
from app.main import app

class TestUserAPI:
    def test_create_user_success(self, client: TestClient):
        # Arrange
        user_data = {"email": "test@example.com", "password": "password123"}
        
        # Act
        response = client.post("/api/v1/users/", json=user_data)
        
        # Assert
        assert response.status_code == 201
        assert response.json()["email"] == user_data["email"]
```

### Migrations Alembic
```python
# Toujours créer des migrations pour les changements de schéma
# Utiliser des noms descriptifs
# alembic revision --autogenerate -m "add_user_email_verification"
```

---

## 🎨 FRONTEND (Vue.js/JavaScript)

### Structure des Composants Vue
```vue
<template>
  <!-- Template en premier -->
  <div class="user-profile">
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
  </div>
</template>

<script>
// Script en second, avec Composition API
import { ref, onMounted } from 'vue'
import { useUserStore } from '@/stores/user'

export default {
  name: 'UserProfile',
  setup() {
    const user = ref(null)
    const userStore = useUserStore()
    
    onMounted(async () => {
      user.value = await userStore.getCurrentUser()
    })
    
    return {
      user
    }
  }
}
</script>

<style scoped>
/* Styles en dernier, toujours scoped */
.user-profile {
  padding: 1rem;
}
</style>
```

### Stores Pinia
```javascript
// Utiliser Pinia pour la gestion d'état
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { userApi } from '@/services/api'

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref(null)
  const isLoading = ref(false)
  
  // Getters
  const isAuthenticated = computed(() => !!user.value)
  
  // Actions
  const login = async (credentials) => {
    isLoading.value = true
    try {
      const response = await userApi.login(credentials)
      user.value = response.data
    } finally {
      isLoading.value = false
    }
  }
  
  return {
    user,
    isLoading,
    isAuthenticated,
    login
  }
})
```

### Services API
```javascript
// Centraliser les appels API
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000
})

// Intercepteurs pour l'authentification
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

export const userApi = {
  async login(credentials) {
    return api.post('/api/v1/auth/login', credentials)
  },
  
  async getCurrentUser() {
    return api.get('/api/v1/users/me')
  }
}
```

### Router Vue
```javascript
// Configuration du router
import { createRouter, createWebHistory } from 'vue-router'
import { useUserStore } from '@/stores/user'

const routes = [
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Login.vue'),
    meta: { requiresGuest: true }
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/Dashboard.vue'),
    meta: { requiresAuth: true }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// Navigation guards
router.beforeEach((to, from, next) => {
  const userStore = useUserStore()
  
  if (to.meta.requiresAuth && !userStore.isAuthenticated) {
    next('/login')
  } else if (to.meta.requiresGuest && userStore.isAuthenticated) {
    next('/dashboard')
  } else {
    next()
  }
})
```

### Composants Réutilisables
```vue
<!-- Toujours créer des props avec validation -->
<script>
export default {
  name: 'BaseButton',
  props: {
    variant: {
      type: String,
      default: 'primary',
      validator: (value) => ['primary', 'secondary', 'danger'].includes(value)
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  emits: ['click']
}
</script>
```

### Modales
```vue
<!-- Toujours placer les modales dans components/Modal/ -->
<!-- Structure standard pour les modales -->
<template>
  <div v-if="isVisible" class="modal-overlay" @click="closeModal">
    <div class="modal-content" @click.stop>
      <div class="modal-header">
        <h3 class="modal-title">{{ title }}</h3>
        <button @click="closeModal" class="modal-close-btn">✕</button>
      </div>
      
      <div class="modal-body">
        <!-- Contenu de la modale -->
      </div>
      
      <div class="modal-actions">
        <button @click="closeModal" class="btn-secondary">Annuler</button>
        <button @click="submit" class="btn-primary">Confirmer</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ExampleModal',
  props: {
    isVisible: {
      type: Boolean,
      default: false
    }
  },
  emits: ['close', 'submit']
}
</script>
```

### Styles avec Tailwind CSS
```vue
<!-- Utiliser les classes utilitaires Tailwind -->
<template>
  <div class="bg-white rounded-lg shadow-md p-6">
    <h2 class="text-xl font-semibold text-gray-900 mb-4">
      {{ title }}
    </h2>
    <p class="text-gray-600">
      {{ description }}
    </p>
  </div>
</template>
```

---

## 📱 MOBILE (React Native/Ionic)

### Structure des Composants React Native
```javascript
// Utiliser des hooks personnalisés pour la logique
import React from 'react'
import { View, Text, StyleSheet } from 'react-native'
import { useUser } from '../hooks/useUser'

const UserProfile = () => {
  const { user, isLoading } = useUser()
  
  if (isLoading) {
    return <LoadingSpinner />
  }
  
  return (
    <View style={styles.container}>
      <Text style={styles.name}>{user?.name}</Text>
      <Text style={styles.email}>{user?.email}</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#fff'
  },
  name: {
    fontSize: 18,
    fontWeight: 'bold'
  },
  email: {
    fontSize: 14,
    color: '#666'
  }
})
```

### Hooks Personnalisés
```javascript
// Centraliser la logique dans des hooks
import { useState, useEffect } from 'react'
import { userApi } from '../services/api'

export const useUser = () => {
  const [user, setUser] = useState(null)
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState(null)
  
  useEffect(() => {
    loadUser()
  }, [])
  
  const loadUser = async () => {
    try {
      const response = await userApi.getCurrentUser()
      setUser(response.data)
    } catch (err) {
      setError(err.message)
    } finally {
      setIsLoading(false)
    }
  }
  
  return { user, isLoading, error, refetch: loadUser }
}
```

### Navigation React Navigation
```javascript
// Configuration de la navigation
import { NavigationContainer } from '@react-navigation/native'
import { createStackNavigator } from '@react-navigation/stack'

const Stack = createStackNavigator()

const AppNavigator = () => (
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen 
        name="Login" 
        component={LoginScreen}
        options={{ headerShown: false }}
      />
      <Stack.Screen 
        name="Dashboard" 
        component={DashboardScreen}
        options={{ title: 'Dashboard' }}
      />
    </Stack.Navigator>
  </NavigationContainer>
)
```

### Stockage Local
```javascript
// Utiliser AsyncStorage pour le stockage local
import AsyncStorage from '@react-native-async-storage/async-storage'

export const storage = {
  async setToken(token) {
    await AsyncStorage.setItem('token', token)
  },
  
  async getToken() {
    return await AsyncStorage.getItem('token')
  },
  
  async removeToken() {
    await AsyncStorage.removeItem('token')
  }
}
```

---

## 🐳 INFRASTRUCTURE (Docker/AWS)

### Dockerfiles
```dockerfile
# Multi-stage builds pour optimiser la taille
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose
```yaml
# Toujours définir des health checks
services:
  backend:
    build: ../backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/db
    depends_on:
      db:
        condition: service_healthy
```

### Variables d'Environnement
```bash
# Toujours utiliser des valeurs par défaut sécurisées
# Préfixer avec le nom du service
BACKEND_SECRET_KEY=your-secret-key-change-in-production
BACKEND_DATABASE_URL=postgresql://user:pass@localhost:5432/db
FRONTEND_API_URL=http://localhost:8000
```

### Terraform
```hcl
# Utiliser des modules réutilisables
module "vpc" {
  source = "./modules/vpc"
  
  environment = var.environment
  cidr_block  = var.vpc_cidr
}

# Toujours tagger les ressources
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = {
    Name        = "${var.environment}-app-server"
    Environment = var.environment
    Project     = "peso"
  }
}
```

---

## 🧪 TESTS

### Tests Backend (Pytest)
```python
# Utiliser des fixtures pour les données de test
import pytest
from fastapi.testclient import TestClient

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture
def test_user(db):
    user = User(email="test@example.com", name="Test User")
    db.add(user)
    db.commit()
    return user

def test_create_user(client, db):
    response = client.post("/api/v1/users/", json={
        "email": "new@example.com",
        "name": "New User"
    })
    assert response.status_code == 201
```

### Tests Frontend (Vitest)
```javascript
// Tests unitaires pour les composants
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import UserProfile from '@/components/UserProfile.vue'

describe('UserProfile', () => {
  it('displays user information', () => {
    const user = { name: 'John Doe', email: 'john@example.com' }
    const wrapper = mount(UserProfile, {
      props: { user }
    })
    
    expect(wrapper.text()).toContain('John Doe')
    expect(wrapper.text()).toContain('john@example.com')
  })
})
```

---

## 🔒 SÉCURITÉ

### Authentification
- Toujours utiliser JWT avec expiration
- Implémenter le refresh token
- Valider les tokens côté serveur
- Utiliser HTTPS en production

### Validation des Données
- Valider toutes les entrées utilisateur
- Utiliser Pydantic pour la validation backend
- Sanitiser les données avant stockage
- Éviter les injections SQL avec SQLAlchemy

### Gestion des Secrets
- Ne jamais commiter de secrets dans le code
- Utiliser des variables d'environnement
- Chiffrer les mots de passe avec bcrypt
- Utiliser des secrets managers en production

---

## 📝 COMMITS ET VERSIONING

### Messages de Commit
```
feat: add user authentication system
fix: resolve database connection timeout
docs: update API documentation
refactor: simplify user service logic
test: add unit tests for weight tracking
```

### Versioning
- Utiliser Semantic Versioning (MAJOR.MINOR.PATCH)
- Tagger les releases avec Git
- Maintenir un CHANGELOG.md
- Automatiser le déploiement avec GitHub Actions

---

## 🚀 DÉPLOIEMENT

### Environnements
- **Development** : Local avec Docker Compose
- **Staging** : Environnement de test sur AWS
- **Production** : Environnement de production sur AWS

### CI/CD
- Tests automatiques sur chaque PR
- Build et déploiement automatique
- Rollback automatique en cas d'échec
- Monitoring et alertes

---

## 📊 MONITORING

### Logs
- Centraliser les logs avec un service dédié
- Structurer les logs en JSON
- Inclure des métadonnées (user_id, request_id)
- Niveaux de log appropriés (DEBUG, INFO, WARN, ERROR)

### Métriques
- Métriques d'application (temps de réponse, taux d'erreur)
- Métriques d'infrastructure (CPU, mémoire, disque)
- Alertes automatiques sur les seuils critiques
- Dashboards de monitoring

---

## 🐳 DÉVELOPPEMENT AVEC DOCKER

### Environnement de Développement
- **Toujours utiliser Docker** pour le développement local
- **Lancer l'environnement** : `./dev.sh` depuis la racine du projet
- **Services disponibles** :
  - Frontend: http://localhost:3000
  - Backend API: http://localhost:8000
  - Documentation API: http://localhost:8000/docs
  - Nginx: http://localhost:80
  - Mailpit: http://localhost:8025

### Commandes Docker Essentielles

#### Backend (Python/FastAPI)
```bash
# Tests dans le conteneur backend
docker-compose exec backend python -m pytest tests/ -v
docker-compose exec backend python -m pytest --cov=app --cov-report=html

# Linting et formatage
docker-compose exec backend black .
docker-compose exec backend isort .
docker-compose exec backend flake8 .
docker-compose exec backend mypy app/

# Sécurité
docker-compose exec backend bandit -r app/
docker-compose exec backend safety check

# Shell interactif
docker-compose exec backend python
docker-compose exec backend bash
```

#### Frontend (Vue.js)
```bash
# Tests dans le conteneur frontend
docker-compose exec frontend npm run test:unit
docker-compose exec frontend npm run test:e2e

# Linting et formatage
docker-compose exec frontend npm run lint
docker-compose exec frontend npx prettier --check src/

# Build
docker-compose exec frontend npm run build

# Shell interactif
docker-compose exec frontend bash
```

#### Base de Données
```bash
# Accès à PostgreSQL
docker-compose exec db psql -U peso -d peso_dev

# Migrations
docker-compose exec backend alembic upgrade head
docker-compose exec backend alembic revision --autogenerate -m "description"

# Reset de la base
docker-compose down -v && docker-compose up -d
```

#### Logs et Monitoring
```bash
# Voir les logs
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f db

# État des services
docker-compose ps
docker-compose top
```

### Tests et Qualité

#### Tests Backend
- **Tests unitaires** : `docker-compose exec backend pytest tests/unit/`
- **Tests d'intégration** : `docker-compose exec backend pytest tests/integration/`
- **Tests avec couverture** : `docker-compose exec backend pytest --cov=app --cov-report=html`
- **Tests de sécurité** : `docker-compose exec backend bandit -r app/`

#### Tests Frontend
- **Tests unitaires** : `docker-compose exec frontend npm run test:unit`
- **Tests E2E** : `docker-compose exec frontend npm run test:e2e`
- **Linting** : `docker-compose exec frontend npm run lint`

### Développement Rapide

#### Workflow Typique
1. **Démarrer l'environnement** : `./dev.sh`
2. **Développer** : Code dans les dossiers `backend/` et `frontend/`
3. **Tester** : Utiliser les commandes Docker ci-dessus
4. **Vérifier** : Les changements sont automatiquement rechargés
5. **Valider** : Tests et linting avant commit

#### Hot Reload
- **Backend** : Redémarrage automatique avec uvicorn
- **Frontend** : Hot reload avec Vite
- **Base de données** : Persistance des données entre redémarrages

## 🔄 WORKFLOW DE DÉVELOPPEMENT

### Branches
- `main` : Code de production
- `develop` : Branche de développement
- `feature/*` : Nouvelles fonctionnalités
- `hotfix/*` : Corrections urgentes

### Code Review
- Toujours faire une PR avant merge
- Tests obligatoires
- Review de code par au moins une personne
- Validation des tests d'intégration

### Documentation
- Maintenir la documentation à jour
- Documenter les changements d'API
- Guides de déploiement
- Troubleshooting commun

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nRudzy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
