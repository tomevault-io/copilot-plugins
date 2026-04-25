## coana

> Sistema que procesa datos presupuestarios, de inventario y de nóminas para generar **unidades de coste** clasificadas por elemento de coste, centro de coste y actividad.

# CoAna — Contabilidad Analítica Universitaria

Sistema que procesa datos presupuestarios, de inventario y de nóminas para generar **unidades de coste** clasificadas por elemento de coste, centro de coste y actividad.

## Stack

| Componente    | Tecnología                                      |
| :------------ | :---------------------------------------------- |
| Lenguaje      | Python 3.14, gestionado con **uv**              |
| DataFrames    | **polars** (nunca pandas)                       |
| Modelos       | **Pydantic v2**                                 |
| UI            | tkinter (editor de árboles), streamlit          |
| CLI           | Typer (`@app.callback()` + `@app.command()`)    |
| Datos entrada | Excel `.xlsx` → caché automática en **Parquet** |
| Documentación | **Typst** (fichero único, compilado a PDF)      |

## Comandos principales

```bash
uv run coana                              # lanza la app Streamlit (incluye botón Ejecutar Fase 1)
uv run especificación                     # compila documentación/especificación.typ → .pdf
uv run coana editor-tree                  # abre el editor gráfico de ficheros .tree
uv pip install -e .                       # instala el paquete en modo editable
```

## Estructura del proyecto

```
coana/                          # paquete principal
  __init__.py                   # re-exports del paquete
  cli.py                        # CLI Typer: subcomandos coana version, editor-tree…
  util/
    __init__.py                 # re-exports: Euro, Árbol, NodoÁrbol, read_excel, UnidadDeCoste, OrigenUC
    euro.py                     # clase Euro (céntimos como int, formato europeo, Pydantic 2)
    arbol.py                    # NodoÁrbol + Árbol: lectura/escritura .tree, consultas, Pydantic 2
    excel_cache.py              # read_excel() con caché parquet transparente
    unidad_de_coste.py          # UnidadDeCoste + OrigenUC (Enum)
  apps/
    editor_tree.py              # editor gráfico tkinter de ficheros .tree
    gen_especificacion.py       # compila documentación/especificación.typ → .pdf
  fase1/
    __init__.py                 # ejecutar() + main() — orquestador de la Fase 1
    presupuesto/
      __init__.py               # re-exports: ContextoPresupuesto, TraductorPresupuesto
      contexto.py               # ContextoPresupuesto: carga todos los ficheros de referencia
      traductor.py              # TraductorPresupuesto: aplica filtro + reglas + genera salidas

data/
  entrada/
    estructuras/                # *.tree: actividades, centros de coste, elementos de coste, etc.
    presupuesto/                # *.xlsx + *.parquet: apuntes, centros, proyectos, líneas, etc.
  fase1/                        # salidas de la Fase 1
documentación/
  especificación.typ            # fuente de verdad: especificación en Typst
  especificación.pdf            # generado con uv run especificación
  img/                          # imágenes (drawio → PDF) incrustadas en la especificación
```

## Terminología clave

- **Unidad de coste (UC):** registro con `id`, `tipo` (elemento de coste), `centro_de_coste`, `actividad`, `importe`, `origen` (OrigenUC enum), `origen_id`, `origen_porción`.
- **OrigenUC:** `Enum(presupuesto, nómina, inventario, unidad)`.
- **Árbol / .tree:** estructura jerárquica de nodos con `código` (01.02.03…), `descripción` e `identificador` único (slug con guiones, p.ej. `estce`). La raíz representa la organización (UJI).
- Salidas presupuesto: `uc presupuesto.xlsx` y `presupuesto sin uc.xlsx`.

## Convenios de código

- Las tablas Excel se leen **siempre** a través de `from coana.util import read_excel`. Nunca directamente con openpyxl o polars. La caché parquet se refresca si el xlsx tiene mtime más reciente.
- Datos de entrada en `data/entrada/`, salida en `data/fase1/` (sin subdirectorio por año).
- Nuevos comandos CLI se añaden en `cli.py` o como sub-Typers en módulos aparte.
- `[tool.setuptools.packages.find] include = ["coana*"]` en pyproject.toml excluye `especificación/` del paquete.

---

## Especificación

La especificación es la fuente de verdad del comportamiento del sistema. Es un fichero Typst único en `documentación/especificación.typ`.

- **Formato:** Typst directo (no se genera desde Markdown).
- **Imágenes:** diagramas drawio exportados a PDF en `documentación/img/`.
- **Compilación:** `uv run especificación` ejecuta `typst compile documentación/especificación.typ`.

### Funciones auxiliares Typst

| Función | Uso |
|:--------|:----|
| `#val("valor")` | Valor literal (azul, monoespaciado) |
| `#etq("identificador")` | Etiqueta/identificador de árbol (rojo, monoespaciado) |
| `#cód("01.02")` | Código de nodo (tamaño reducido, entre paréntesis) |
| `#campo[nombre]` | Campo de entrada (teal) |
| `#lista-plana[...]` | Lista sin estilo itemize |

### Cómo modificar la especificación

1. Editar `documentación/especificación.typ` directamente.
2. Ejecutar `uv run especificación` para compilar el PDF.
3. Implementar el código Python correspondiente en `coana/fase1/`.

---

## Flujo de trabajo habitual

```
documentación/especificación.typ  →  uv run especificación  →  especificación.pdf
                                                                      ↓ (leer y entender)
data/entrada/                     →  Ejecutar Fase 1 (botón en la app)  →  data/fase1/
```

El ciclo típico al desarrollar una nueva regla:

1. Actualizar `documentación/especificación.typ`.
2. Ejecutar `uv run especificación` para compilar el PDF y verificar que se ve bien.
3. Implementar la regla en `coana/fase1/`.
4. Pulsar "Ejecutar Fase 1" en la app (`uv run coana`).
5. Revisar las salidas en `data/fase1/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amarzal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
