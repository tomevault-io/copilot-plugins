## recyclique

> Standards de tests et assurance qualité


# Testing & Quality Assurance - Recyclic

## Structure des Tests

### Backend (API) - Python/pytest
- **Répertoire** : [api/tests/](mdc:api/tests/)
- **Configuration** : [pytest.ini](mdc:pytest.ini)
- **Fixtures** : [api/tests/conftest.py](mdc:api/tests/conftest.py)
- **Types de tests** :
  - `test_basic.py` : Tests de base
  - `test_infrastructure.py` : Tests d'infrastructure
  - `test_postgres_connectivity.py` : Tests de connectivité DB

### Frontend (React) - Vitest
- **Configuration** : [frontend/vitest.config.js](mdc:frontend/vitest.config.js)
- **Setup** : [frontend/src/test/setup.ts](mdc:frontend/src/test/setup.ts)
- **Utils** : [frontend/src/test/test-utils.tsx](mdc:frontend/src/test/test-utils.tsx)
- **Types** : [frontend/src/test/vitest.d.ts](mdc:frontend/src/test/vitest.d.ts)

## Standards de Couverture

### Exigences Minimales
- **Code métier** : >80% de couverture
- **Composants UI** : >60% de couverture
- **API endpoints** : >90% de couverture
- **Services critiques** : >95% de couverture

### Tests Obligatoires
- **Unit tests** : Toutes les fonctions publiques
- **Integration tests** : Workflows complets
- **API tests** : Tous les endpoints
- **Database tests** : Migrations et modèles

## Types de Tests

### Backend Tests
```python
# Structure recommandée
def test_function_name_should_expected_behavior():
    # Arrange
    # Act  
    # Assert
```

### Frontend Tests
```typescript
// Structure recommandée
describe('ComponentName', () => {
  it('should render correctly', () => {
    // Test implementation
  });
});
```

## Configuration des Tests

### Base de Données de Test
- **PostgreSQL** : Base séparée pour les tests
- **Scripts** : [api/test_postgres.bat](mdc:api/test_postgres.bat) et [api/test_postgres.sh](mdc:api/test_postgres.sh)
- **Variables** : [api/env.test](mdc:api/env.test) et [api/env.test.postgres](mdc:api/env.test.postgres)

### Fixtures et Mocks
- **Database** : Fixtures pour données de test
- **API calls** : Mocks pour services externes
- **Telegram** : Mocks pour bot interactions

## Commandes de Test

### Backend
```bash
# Tests complets
cd api && python -m pytest tests/

# Tests avec Docker
docker-compose exec api python -m pytest tests/

# Tests avec couverture
python -m pytest tests/ --cov=src/
```

### Frontend
```bash
# Tests unitaires
cd frontend && npm test

# Tests avec couverture
npm run test:coverage
```

## Quality Gates

### Pre-commit Hooks
- **Linting** : Black, flake8 (Python), ESLint (JS/TS)
- **Type checking** : mypy (Python), tsc (TypeScript)
- **Tests** : Exécution automatique des tests
- **Security** : Scan des dépendances

### CI/CD Pipeline
- **Tests automatiques** : Sur chaque PR
- **Coverage reports** : Génération et vérification
- **Security scans** : Dependabot, CodeQL
- **Performance tests** : Tests de charge pour API

## Documentation des Tests

### Test Documentation
- **README** : [api/TESTS_README.md](mdc:api/TESTS_README.md)
- **Test cases** : Documentation des cas de test complexes
- **Fixtures** : Documentation des données de test
- **Mocks** : Documentation des mocks et stubs

### Test Reports
- **Coverage** : Rapports de couverture générés
- **Performance** : Métriques de performance des tests
- **Flaky tests** : Identification et correction

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/La-Clique-qui-Recycle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
