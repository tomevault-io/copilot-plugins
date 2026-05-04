## ingenieria-de-datos

> **IMPORTANTE**: Cuando se modifique `README.md` en la raíz del proyecto, DEBES actualizar también `docs/index.md` para mantenerlos sincronizados.

# Reglas de Cursor para este Repositorio

## 🔄 Sincronización README ↔ Landing Page

**IMPORTANTE**: Cuando se modifique `README.md` en la raíz del proyecto, DEBES actualizar también `docs/index.md` para mantenerlos sincronizados.

### Reglas de Sincronización:

1. **Título y descripción**: Mantener consistencia entre ambos archivos
2. **Sección "Cómo empezar"**: Debe ser idéntica en ambos archivos
3. **Sección "Start Here"**: Debe aparecer en ambos con los mismos enlaces
4. **Nombres de secciones**: 
   - "AI como copiloto" → "Inteligencia Artificial como copiloto" (en ambos)
   - Mantener consistencia en todos los nombres
5. **Estructura del índice**: La ruta de aprendizaje debe ser similar en ambos

### Cuando el usuario modifique README.md:

1. **Detecta los cambios** en README.md
2. **Aplica los mismos cambios** a `docs/index.md`:
   - Actualiza la sección correspondiente
   - Mantén los enlaces a GitHub (docs/index.md usa enlaces absolutos a GitHub)
   - Preserva la estructura de navegación
3. **Notifica al usuario** que también actualizaste docs/index.md

### Excepciones:

- Los badges del README no necesitan estar en docs/index.md
- La sección "Documentación Adicional" puede diferir ligeramente
- docs/index.md puede tener secciones adicionales específicas de Pages

### Ejemplo de sincronización:

Si en README.md cambias:
- "Cómo empezar" → Actualiza también en docs/index.md
- Título de sección → Actualiza también en docs/index.md
- Agregas/quitas secciones → Actualiza también en docs/index.md

---

## 📝 Estilo y Convenciones

- Usar español en todo el contenido
- Mantener emojis consistentes
- Enlaces a GitHub deben ser absolutos en docs/index.md
- Enlaces relativos en README.md

---

## 🎯 Contexto del Proyecto

Este es un repositorio educativo de **Ingeniería de Datos en Español**:
- Contenido desde principiante hasta avanzado
- Enfoque en fundamentos sólidos
- AI como copiloto, no reemplazo
- Todo el contenido está en español

---

## 📁 Organización de Archivos y Ejercicios

**IMPORTANTE**: Sigue estas normas para mantener la organización del repositorio:

### Estructura de Ejercicios

1. **Ubicación de ejercicios:**
   - Los ejercicios de cada herramienta/orquestador deben estar en: `{modulo}/ejercicios/{nombre-herramienta}/`
   - Ejemplo: `05_pipelines/ejercicios/prefect/`, `05_pipelines/ejercicios/dagster/`

2. **Ubicación de outputs:**
   - Todos los archivos generados (outputs) deben guardarse en: `{modulo}/data/output/`
   - Ejemplo: `05_pipelines/data/output/`, `03_python/data/output/`
   - **NO** guardar outputs en la carpeta de ejercicios

3. **Creación de archivos:**
   - **Prioridad 1**: Usar Cursor (crear archivos desde el explorador o con `Cmd+N`/`Ctrl+N`)
   - **Prioridad 2**: Usar terminal/bash como alternativa
   - Siempre mencionar ambas opciones en la documentación

4. **Rutas en código:**
   - Usar `pathlib.Path` para construir rutas de forma robusta
   - Calcular `BASE_DIR` desde `__file__` para rutas relativas al proyecto
   - Crear directorios automáticamente con `mkdir(parents=True, exist_ok=True)`

5. **Entorno virtual:**
   - **SIEMPRE** recordar activar el entorno virtual antes de ejecutar comandos:
     ```bash
     pyenv activate ingenieria-de-datos
     # O: pyenv shell ingenieria-de-datos
     ```
   - Incluir este recordatorio en todas las secciones de instalación y ejecución

6. **Ayuda con errores:**
   - Incluir nota sobre usar el chat de Cursor (`Cmd+L` o `Ctrl+L`) para obtener ayuda
   - Mencionar que pueden copiar y pegar errores para recibir ayuda

### Ejemplo de estructura correcta:

```
05_pipelines/
├── ejercicios/
│   ├── prefect/
│   │   ├── 01-primer-flow.py
│   │   ├── 02-pipeline-etl.py
│   │   └── ...
│   └── dagster/
│       ├── 01-primer-asset.py
│       └── ...
└── data/
    └── output/
        ├── ventas_processed.parquet
        └── ...
```

---

## ⚠️ Recordatorios Importantes

- **SIEMPRE** sincroniza README.md y docs/index.md cuando haya cambios
- Mantén consistencia en nombres (especialmente "Inteligencia Artificial como copiloto")
- Los enlaces en docs/index.md deben apuntar a GitHub (absolutos)
- Preserva la estructura de navegación en ambos archivos
- **SIEMPRE** sigue la estructura de ejercicios y outputs descrita arriba

---
> Source: [caroacostatovany/ingenieria-de-datos](https://github.com/caroacostatovany/ingenieria-de-datos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
