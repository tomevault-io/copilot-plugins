## qgis-plugin-generator

> Generador de plugins de QGIS: de una descripción en castellano a un `.zip`

# QGIS Plugin Generator — instrucciones para agentes

Generador de plugins de QGIS: de una descripción en castellano a un `.zip`
instalable y validado. Pensado para instalarse como skill en Claude Code o
Codex — no es una herramienta de validación de código escrito a mano.

## Antes de escribir una sola línea de código

Lee `skill/GUIA.md`. Es la guía operativa completa: reglas duras, flujo de
generación, selección de patrón y principios de código. Los documentos de
`skill/reference/` se leen **antes** de programar, no para comprobar
después.

Orden recomendado de lectura para una tarea nueva:

1. `skill/GUIA.md` — siempre
2. `skill/reference/qgis4-migration.md` — siempre (es lo que más rompe)
3. El documento de referencia del patrón que toque (Processing, UI,
   geometrías, señales…)

## Entorno (verificado, no asumir otra cosa)

- QGIS **4.0.2**, **PyQt6**, Python 3.12, instalado en `<QGIS>`
- Python de QGIS: `<QGIS>/bin/python-qgis.bat`
- Perfil de usuario: `%APPDATA%\QGIS\QGIS4\profiles\default`

**La trampa principal:** casi todo el PyQGIS que circula por internet (y lo
que un modelo genera de memoria) es de QGIS 3 + PyQt5 y **no funciona
aquí**. Enums sin cualificar (`Qgis.Info`), `QVariant` para tipos de campo,
`exec_()`, iconos compilados con `pyrcc`. Todo eso falla.

## Flujo

```
describir → elegir patrón → generar en output/ → validar → corregir → ZIP
```

Validar y empaquetar:

```bash
python validator/run_all.py output/<plugin>
```
```bash
python validator/build_zip.py output/<plugin>
```

El validador solo usa la biblioteca estándar de Python: funciona con
cualquier intérprete 3.9+, sin instalar nada. Devuelve
`ARCHIVO:LÍNEA: NIVEL: REGLA: mensaje` y código de salida 1 si hay errores.
Corrige y revalida, **máximo 3 iteraciones**; si sigue fallando, para y
explica el bloqueo en vez de seguir adivinando.

## Estructura

```
skill/GUIA.md         Guía operativa (única copia del conocimiento)
skill/reference/      9 documentos de API PyQGIS verificados
skill/templates/      5 patrones de plugin con marcadores {{...}}
skill/checklist.md    Revisión manual final
validator/            Validación estática, sin dependencias
examples/INDEX.md     Plugins de referencia clasificados por patrón
output/               Plugins generados + sus ZIP
```

## Reglas del proyecto

- Nunca importar `PyQt5`/`PyQt6` directamente: siempre `qgis.PyQt.*`.
- Enums scoped siempre (`Qgis.MessageLevel.Info`, no `Qgis.Info`).
- Campos con `QMetaType.Type.*`, no `QVariant.*`.
- Todo lo que añade `initGui()` lo quita `unload()`.
- Código en inglés; textos visibles en castellano dentro de `self.tr()`.
- Nada de `print()`: usar `QgsMessageLog` o `iface.messageBar()`.
- Antes de dar por bueno un plugin, pasarlo por el validador. Si no se ha
  probado dentro de QGIS real, **decírselo al usuario explícitamente**.
- En ejemplos y datos de prueba, usar geometrías y dominios genéricos
  (capas de líneas, puntos, parcelas). No usar dominios de negocio
  reconocibles.

## Probar de verdad sin abrir QGIS

```bash
<QGIS>/bin/python-qgis.bat script.py
```

Un algoritmo de Processing se puede instanciar y ejecutar en un script
independiente con `QgsApplication` + `Processing.initialize()`, añadiendo
`<QGIS>/apps/qgis\python\plugins` al `sys.path`. Es la forma más rápida de
comprobar que funciona de verdad y no solo que "parece correcto".

---
> Source: [BernabeHL/qgis-plugin-generator](https://github.com/BernabeHL/qgis-plugin-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
