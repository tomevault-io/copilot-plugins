## inazuma-eleven-123-spanish

> - PT = puntos de técnica; PE = puntos de resistencia (versión castellana).

# Retoma del proyecto

## Terminología confirmada por el usuario

- PT = puntos de técnica; PE = puntos de resistencia (versión castellana).
  Usar estas siglas también en indicadores gráficos y textos de ayuda.

## BLOQUEO DE CAJA Y TIPOGRAFÍA — orden explícita del usuario

- El usuario confirma que la apariencia de los diálogos de v20 está muy bien.
  Es la referencia aprobada. No modificar caja, dimensiones, posición, tamaño
  de letra, glifos, espaciado, fuentes, codificación ni algoritmo de saltos.
- Mantener fullwidth, avance de cálculo 11, límite 220 y tres líneas por página,
  con las cinco fuentes exactas verificadas por `tools/dialogue_lock.py`.
- Las nuevas traducciones deben adaptarse a esta configuración. Un texto largo
  no autoriza a cambiar la caja, las fuentes o el motor para hacerlo caber.
- No eliminar, desactivar, actualizar hashes ni sortear el bloqueo para compilar.
  Solo una nueva petición explícita del usuario sobre la tipografía permite
  revisar esta protección. Una petición general de continuar o traducir no basta.
- Esta aprobación se refiere al aspecto observado; no significa que toda la ROM
  o todos los capítulos hayan superado el recorrido QA.

- Objetivo actual: traducir y depurar Inazuma Eleven 1 de la recopilación 3DS.
- Leer `CLAUDE.md` y `docs/PROTOCOLO_QA_IE1.md`. El protocolo es una instrucción
  explícita del usuario: cajas de diálogo sin bugs gráficos; recorrido hasta la
  primera pachanga con varios NPC; detenerse ante cada fallo, corregirlo y repetir
  exactamente su reproducción antes de continuar.
- Consultar `docs/SSD_REGISTROS_IE1.md` antes de tocar el texto. El reinsertor
  antiguo omite el tamaño de cada registro inline. Las conclusiones históricas de
  imposibilidad no son evidencia de que el formato correcto falle.
- Flujo acordado con el usuario el 2026-09-05: el usuario juega y prueba; el agente
  observa el emulador y sus logs sin enviar entradas de teclado/ratón durante la
  prueba. Preparar correcciones y volver al punto del fallo con el usuario.
- No subir ROMs, extracciones, logs ni datos oficiales recuperados. Mantener los
  originales y distinguir una candidata de una build verificada en juego.

## Espacio en disco: instrucción del usuario

- No acumular builds ni copias históricas en `work/`. Mantener solo la candidata
  actual y sus entradas necesarias. Tras verificar la nueva candidata y su copia
  instalada, borrar los binarios de la anterior, no archivarlos en otra carpeta.
- No generar una ROM completa adicional si basta actualizar el mod de Azahar.
- Eliminar intermediarios de extracción y empaquetado, cachés regenerables y logs
  duplicados al terminar. Conservar informes pequeños útiles para el diagnóstico.
- Nunca borrar originales, guardados, traducciones revisadas, manifiestos,
  recursos únicos o herramientas necesarias para reconstruir la candidata.
- Antes de borrar, comprobar las rutas absolutas, las dependencias y la candidata
  instalada. Si el usuario ha limpiado archivos manualmente, inventariar el estado
  real y no asumir que siguen existiendo builds citadas en notas históricas.

## Herramientas (ie123kit)

- El código Python vive en `tools/src/ie123kit` (instalar con `pip install -e tools[dev]`); en `tools/`
  solo hay shims generados con `python -m ie123kit.nucleo.compat.shims generar`. Mapa en `tools/README.md`.
- Los 5 congelados (`dialogue_typography.py`, `font_patch.py`, `dialogue_lock.py`, `build_ie1_probe.py`,
  `build_ui_revision.py`) siguen en `tools/` y no se tocan ni se copian.
- Tests: `python -X utf8 -m pytest tools/tests -m "not requiere_rom"` (y `-m requiere_rom` en local con
  `work/`). Guardias: `python -m ie123kit.nucleo.compat.guardia bloqueados`, `... guardia git` y
  `python -m ie123kit.nucleo.compat.shims comprobar`.
- No se desactiva ni se salta la CI `.github/workflows/toolkit.yml` (filtrada a `tools/**`) ni
  `.github/workflows/guardia.yml`, que corre **sin filtro de rutas** en todos los commits y es la que
  impide subir ROMs o datos extraídos en cualquier carpeta. No añadir `paths` a `guardia.yml`.

---
> Source: [luishidalgoa/inazuma-eleven-123-spanish](https://github.com/luishidalgoa/inazuma-eleven-123-spanish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
