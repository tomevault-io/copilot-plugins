## sdd-dev

> Este archivo proporciona contexto e instrucciones para los agentes de IA que trabajan en este ambiente de desarrollo.

# AGENTS.md - Ambiente de Desarrollo con Agentes de IA

Este archivo proporciona contexto e instrucciones para los agentes de IA que trabajan en este ambiente de desarrollo.

## Descripción del Proyecto

Este es un ambiente de desarrollo basado en Docker para crear aplicaciones web con Python, utilizando **Reflex** para el frontend y **FastAPI** para el backend. El proyecto sigue la metodología **Spec-Driven Development (SDD)**, donde las especificaciones guían la implementación.

## Configuración del Ambiente

### Iniciar el Ambiente de Desarrollo

```bash
# Iniciar todos los servicios
./scripts/start-environment.sh

# O manualmente con Docker Compose
docker compose up -d vibekanban
```

### Crear un Nuevo Proyecto

```bash
./scripts/create-project.sh nombre-del-proyecto
```

### Ejecutar una Aplicación

```bash
# Navegar al proyecto
cd workspace/nombre-del-proyecto

# Instalar dependencias
pip install -r requirements.txt

# Inicializar Reflex (primera vez)
reflex init

# Ejecutar la aplicación
reflex run
```

### Puertos del Sistema

| Servicio | Puerto | Descripción |
| :--- | :--- | :--- |
| VibeKanban | 3000 | Orquestador de tareas |
| Reflex Frontend | 3001 | Frontend de la aplicación |
| Reflex Backend | 8001 | Backend de la aplicación |
| PostgreSQL | 5432 | Base de datos (opcional) |

## Arquitectura de Proyectos

### Estructura de Directorios

Cada proyecto creado con `create-project.sh` sigue esta estructura:

```
nombre_proyecto/
├── nombre_proyecto/      # Módulo principal de Reflex
│   ├── components/       # Componentes de UI reutilizables
│   ├── pages/            # Páginas de la aplicación
│   ├── state/            # Gestión de estado
│   ├── utils/            # Utilidades
│   └── nombre_proyecto.py # Punto de entrada de la app
├── backend/              # Backend FastAPI (opcional)
│   ├── api/endpoints/    # Endpoints de la API
│   ├── domain/models/    # Modelos de dominio
│   ├── application/      # Casos de uso y schemas
│   └── infrastructure/   # Configuración y DB
├── assets/               # Archivos estáticos
├── tests/                # Tests
├── openspec/             # Especificaciones
│   ├── specs/            # Specs aprobadas
│   └── changes/          # Propuestas de cambio
├── rxconfig.py           # Configuración de Reflex
└── requirements.txt      # Dependencias
```

### Principios de Diseño

El código debe seguir estos principios:

1.  **Clean Architecture**: Separar dominio, aplicación e infraestructura.
2.  **Single Responsibility**: Cada módulo tiene un único propósito.
3.  **Dependency Injection**: Usar `Depends()` de FastAPI.
4.  **Type Safety**: Usar type hints en todo el código.

## Estilo de Código

### Estándares de Python

| Herramienta | Configuración |
| :--- | :--- |
| Formatter | `black` (line length 88) |
| Import sorting | `isort` (compatible con black) |
| Linter | `ruff` |
| Type checker | `mypy` (modo estricto) |

### Convenciones de Nombres

| Tipo | Convención | Ejemplo |
| :--- | :--- | :--- |
| Variables/Funciones | snake_case | `user_name`, `get_user()` |
| Clases | PascalCase | `UserService` |
| Constantes | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Privado | _leading_underscore | `_internal_method()` |

### Documentación

Todas las funciones públicas deben tener docstrings en estilo Google. Los comentarios y README deben estar en español.

```python
def get_user(user_id: int) -> User:
    """Obtiene un usuario por su ID.

    Args:
        user_id: El ID del usuario a buscar.

    Returns:
        El objeto User correspondiente.

    Raises:
        UserNotFoundError: Si el usuario no existe.
    """
    ...
```

## Reglas de Seguridad

**CRÍTICO - Siempre seguir estas reglas:**

1.  **Nunca** commitear archivos `.env` o secretos.
2.  Usar variables de entorno para configuración sensible.
3.  Todos los endpoints de API requieren autenticación (excepto `/health`, `/docs`).
4.  Sanitizar todas las entradas de usuario.
5.  Usar consultas parametrizadas (SQLModel lo maneja automáticamente).
6.  Validar uploads de archivos.
7.  Configurar headers de seguridad (CSP, CORS).
8.  Registrar eventos de seguridad en logs.

## Requisitos de Testing

| Requisito | Valor |
| :--- | :--- |
| Cobertura mínima | 80% |
| Endpoints | Todos deben tener tests |
| Fixtures | Usar pytest fixtures para setup |
| Mocking | Mockear servicios externos |
| Casos de error | Testear explícitamente |

## Flujo de Git

### Mensajes de Commit

Seguir Conventional Commits en español:

| Prefijo | Uso |
| :--- | :--- |
| `feat:` | Nueva funcionalidad |
| `fix:` | Corrección de bug |
| `docs:` | Documentación |
| `style:` | Formato (sin cambios de código) |
| `refactor:` | Refactorización |
| `test:` | Tests |
| `chore:` | Mantenimiento |

### Nombres de Branches

| Tipo | Formato |
| :--- | :--- |
| Feature | `feature/descripcion-corta` |
| Fix | `fix/descripcion-corta` |
| Refactor | `refactor/descripcion-corta` |

## Habilidades Disponibles (Skills)

Usar estas habilidades para tareas específicas:

| Skill | Cuándo Usar |
| :--- | :--- |
| `fastapi-endpoint` | Crear nuevos endpoints de API |
| `reflex-component` | Construir componentes de UI |
| `pydantic-model` | Definir modelos de datos |
| `sqlmodel-crud` | Operaciones de base de datos |
| `pytest-test` | Escribir tests |
| `security-headers` | Revisión de seguridad antes de merge |

## Flujo de Trabajo con OpenSpec

El desarrollo sigue el flujo de Spec-Driven Development:

1.  **Crear Spec**: Antes de codificar, crear una especificación en `openspec/changes/`.
2.  **Revisar**: Iterar sobre la propuesta hasta que esté aprobada.
3.  **Implementar**: Escribir el código siguiendo la especificación.
4.  **Testear**: Crear tests que validen la implementación.
5.  **Archivar**: Mover la spec a `openspec/specs/` cuando esté completa.

> **Importante**: Nunca escribir código sin una especificación aprobada.

## Coordinación de Agentes

Este proyecto utiliza múltiples agentes especializados:

| Agente | Responsabilidad |
| :--- | :--- |
| **Product Owner** | Revisar specs y criterios de aceptación |
| **Frontend Dev** | Componentes y páginas de Reflex |
| **Backend Dev** | FastAPI y lógica de negocio |
| **DevSecOps** | Seguridad y despliegue |
| **QA Engineer** | Testing y validación |

Cuando haya dudas, cambiar al agente de Plan (tecla Tab) para analizar antes de implementar.

---
> Source: [vtomasv/sdd-dev](https://github.com/vtomasv/sdd-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
