## claude-code-docs-spa

> - ✅ Layout src/ (creado con uv init --package)

# Claude Code - Python Standards Modernos

## REGLAS OBLIGATORIAS PARA ESTE PROYECTO

### ESTRUCTURA
- ✅ Layout src/ (creado con uv init --package)
- ✅ pyproject.toml OBLIGATORIO
- ❗ Type hints OBLIGATORIOS en todas las funciones
- ❗ Tests pytest OBLIGATORIOS (>80% cobertura)
- ❗ Python >= 3.11

### HERRAMIENTAS INSTALADAS
- ✅ Ruff (linting + formatting)
- ✅ MyPy (type checking)
- ✅ pytest + pytest-cov + pytest-asyncio (testing)
- ✅ pre-commit (hooks)
- ✅ uv (package manager)
- ✅ bandit (security scanning)

### **FLUJO DE DESARROLLO RECOMENDADO**

#### **1. ENFOQUE ESCALABLE PARA MÓDULOS**
- **Recomendado**: Empezar con `src/[proyecto]/main.py` como punto de entrada principal
- **Flexible**: Crear módulos adicionales cuando sea lógicamente apropiado
- **Estructura sugerida**:
```
src/mi_proyecto/
├── __init__.py          # Exporta funciones principales
├── main.py             # Entry point principal
├── config.py           # Configuración de la aplicación
├── utils/              # Funciones de utilidad reutilizables
├── services/           # Lógica de negocio
└── models/             # Modelos de datos
```

#### **2. ESTRUCTURA DE main.py RECOMENDADA**
```python
"""Main module for [FUNCIONALIDAD] - brief description."""

from __future__ import annotations

import asyncio
from typing import Any
# Imports de módulos internos según necesidad

async def main() -> dict[str, Any]:
  """Main application entry point.

  Returns:
      Dictionary with application results
  """
  # IMPLEMENTAR FUNCIONALIDAD AQUÍ
  result = await tu_funcion_principal()
  return {"status": "success", "result": result}

if __name__ == "__main__":
  result = asyncio.run(main())
  print(result)
```

#### **3. CUÁNDO CREAR MÓDULOS ADICIONALES**
Crear nuevos módulos cuando:
- La funcionalidad pertenece a un dominio lógico diferente
- Necesitas reutilizar código en múltiples lugares
- Tienes clases complejas que merecen su propio módulo
- El código se vuelve difícil de mantener en un solo archivo

**Orden de creación:**
1. **Evaluar si el módulo es necesario** (evitar fragmentación prematura)
2. **Crear el módulo apropiado** (ej: `auth.py`, `database.py`, `utils/helpers.py`)
3. **Actualizar imports en archivos que lo necesiten**
4. **Exportar funciones públicas en `__init__.py`**
5. **Crear tests correspondientes** (`tests/test_module.py`)


### CÓDIGO OBLIGATORIO
1. **Type hints en TODAS las funciones públicas**
2. **Async/await para operaciones I/O**
3. **Pydantic para validación de datos**
4. **Excepciones específicas (NO Exception genérico)**
5. **Docstrings Google style OBLIGATORIOS**
6. **Context managers para recursos**

### TEMPLATE FUNCIÓN ESTÁNDAR
```python
from __future__ import annotations
from typing import Any

async def process_data(
  items: list[str],
  limit: int = 100,
  metadata: dict[str, Any] | None = None
) -> dict[str, Any]:
  """Process data items asynchronously.

  Args:
      items: List of items to process
      limit: Maximum items to process
      metadata: Optional metadata

  Returns:
      Dictionary with processing results

  Raises:
      ValueError: If limit is negative
  """
  if limit < 0:
      raise ValueError("Limit must be non-negative")

  return {"processed": len(items[:limit])}
```

### TESTING OBLIGATORIO
- Test para cada función pública
- Mocks para dependencias externas
- Tests parametrizados cuando aplique
- Tests async con pytest-asyncio

### **REGLAS PARA MODIFICACIONES: ENFOQUE EQUILIBRADO**

**IMPORTANTE: Este proyecto sigue un diseño escalable que empieza simple pero permite crecimiento organizado. Se prioriza la simplicidad inicial pero se permite una estructura modular cuando sea apropiado.**

**PROTOCOLO PARA NUEVAS FUNCIONALIDADES:**
1. ✅ **Evalúa el alcance** de la funcionalidad antes de decidir dónde implementarla
2. ✅ **Funcionalidades pequeñas**: Agregar a `main.py` o módulo existente relacionado
3. ✅ **Funcionalidades complejas o de dominio específico**: Crear módulo dedicado
4. ✅ **Siempre actualizar** `__init__.py` para exportar funciones públicas
5. ✅ **Crear tests apropiados** en el archivo de test correspondiente

**EJEMPLO PRÁCTICO:**
**Usuario pide:** "Crea una calculadora"

**ENFOQUE RECOMENDADO:**
- Si es solo operaciones básicas: agregar a `main.py`
- Si es una calculadora completa con múltiples operaciones: crear `calculator.py`
- Si las operaciones son reutilizables: crear `utils/calculator.py`
- Actualizar `__init__.py` para exportar funciones principales
- Crear `tests/test_calculator.py` o agregar a `tests/test_main.py` según corresponda

**EJEMPLOS DE CUÁNDO CREAR MÓDULOS:**
✅ **Crear módulos cuando:**
- Configuración de aplicación: `config.py`
- Autenticación: `auth.py` o `services/auth.py`
- Base de datos: `database.py` o `models/`
- Utilidades reutilizables: `utils/helpers.py`
- API endpoints: `api/endpoints.py`

❌ **Evitar crear módulos cuando:**
- Son solo 2-3 funciones pequeñas
- El código no se reutiliza en otros lugares
- La funcionalidad es muy específica y no crecerá

**VERIFICACIÓN ANTES DE APLICAR CUALQUIER CAMBIO:**
- Lee este archivo `.claude.md` COMPLETO antes de proceder
- Usa herramientas como `read_file` para ver el contenido actual de los archivos antes de editar
- Después de cambios, valida: `uv run ruff check . && uv run pytest --cov=src`

### SEGURIDAD CRÍTICA
- Validar TODAS las entradas externas
- NO hardcodear secretos (usar variables de entorno)
- Logging estructurado con structlog
- Bandit security scan OBLIGATORIO

### VERIFICACIÓN OBLIGATORIA
Ejecutar antes de cada commit:
```bash
uv run ruff check .
uv run ruff format .
uv run mypy src/
uv run pytest --cov=src --cov-fail-under=80
```

### COMANDOS UV ÚTILES
```bash
uv add package-name          # Agregar dependencia
uv add --dev package-name    # Agregar dependencia de desarrollo
uv run comando               # Ejecutar en el venv
uv sync                      # Sincronizar dependencias
uv lock                      # Actualizar lockfile
```


**Claude: Sigue estas reglas como guía flexible. Prioriza la calidad del código y la escalabilidad del proyecto.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JhonMA82)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JhonMA82)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
