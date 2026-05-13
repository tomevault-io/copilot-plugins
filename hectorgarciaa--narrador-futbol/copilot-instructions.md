## narrador-futbol

> Guía para agentes que trabajen en este repositorio (`narrador-futbol`).

# AGENTS.md

Guía para agentes que trabajen en este repositorio (`narrador-futbol`).

## 1) Objetivo y alcance real

Este proyecto busca construir un narrador automático de fútbol con IA.

Estado actual del código:
- Implementado de forma funcional: detección (YOLO), tracking de objetos/jugadores, asignación de equipo por color de camiseta, visualización en video y métricas de tracking.
- Parcial o pendiente: detección de acciones, integración LLM y TTS (están descritos en `README.me`, pero no forman un pipeline completo en código productivo hoy).

Cuando recibas tareas, prioriza este alcance real y evita asumir que todo el pipeline de narración ya existe.

## 2) Stack y dependencias

Lenguaje principal: Python.
- Versión objetivo del proyecto: `Python 3.13.7`.
- Comando habitual del proyecto: `python` (debe resolver a `3.13.7` en este repo).
- Si se usa otro intérprete temporalmente, indícalo explícitamente al usuario antes de entregar cambios.

Dependencias usadas por el código:
- `ultralytics`
- `supervision`
- `opencv-python` (`cv2`)
- `numpy`
- `scikit-learn`
- `matplotlib`
- `pandas`
- `plotly`
- `seaborn`
- `roboflow` (solo para script de descarga de dataset)

Notas:
- `requirements.txt` es la fuente de dependencias del proyecto y debe mantenerse actualizado.
- No asumir que el entorno tiene librerías instaladas; verificar antes de ejecutar scripts pesados.

## 3) Estructura clave del repo

Rutas relevantes:
- `src/detection/`: scripts de detección y fine-tuning con YOLO.
- `src/tracking/`: pipeline de tracking, detectores auxiliares, tracker custom y evaluación.
- `data/partidoPrueba/`: videos de prueba.
- `data/detection/FootBall-Detection-2/`: dataset YOLO.
- `models/`: pesos base y fine-tuned.
- `output/`: salidas de tracking y JSON.
- `README.me`: documentación general (algunas secciones son aspiracionales).

## 4) Cómo ejecutar sin romper rutas

Importante: varios scripts usan rutas relativas pensadas para ejecutarse desde su propia carpeta.

Ejemplos correctos:
- Detección base:
  - `cd src/detection && python pruebaDeteccionYolo.py`
- Detección con modelo fine-tuned:
  - `cd src/detection && python pruebaFinetuning.py`
- Fine-tuning:
  - `cd src/detection && python finetuning.py`
- Tracking principal:
  - `cd src/tracking && python main.py`

Si quieres ejecutar desde raíz, adapta rutas o refactoriza a `Path(__file__)`.

## 5) Convenciones técnicas del proyecto

- Tracking custom:
  - `src/tracking/trackers/byteTracker.py` es una variante local de ByteTrack con lógica de equipo (`equipo`) y penalizaciones de matching.
  - No reemplazarla por implementación upstream sin validar comportamiento.
- Formato de tracks esperado:
  - Diccionario por clase (`player`, `goalkeeper`, `referee`, `ball`) con lista por frame y luego `{track_id: data}`.
  - `data` suele incluir: `bbox`, `confidence`, `team`, `distances`, `shirt_color`, `bbox_size`.
- Límites de objetos:
  - Los límites (`player=22`, `ball=1`, `referee=3`) se validan sobre `tracks` (salida de tracking), no sobre el conteo crudo de detección YOLO por frame.
- Dibujo/anotación:
  - `Drawer` usa etiquetas de equipos concretas (`Real Madrid`, `Wolfsburgo`) al mostrar distancias. Si cambias nombres de equipos, ajusta esta parte.
- Ejecución con UI:
  - `SHOW_OUTPUT=True` en tracking abre ventana OpenCV. En entornos headless usar `False`.

## 6) Seguridad y secretos

- No introducir nuevas credenciales en código.
- En `src/detection/descargarDataSetDeteccion.py` hay API keys hardcodeadas (deuda técnica existente). Si tocas ese script, migra a variables de entorno y no expongas valores en commits.

## 7) Higiene de repositorio

Este repo tiene archivos grandes (videos, pesos `.pt`, salidas). Evita ensuciar el historial:
- No agregar nuevos binarios pesados salvo petición explícita.
- Evitar commitear salidas generadas (`output/`, `runs/`) y archivos de sistema (`.DS_Store`).
- Mantener cambios acotados a lo pedido por el usuario.

## 8) Validación mínima al cambiar código

No hay suite de tests formal. Cuando hagas cambios:
- Validar sintaxis de Python en archivos modificados.
- Si el cambio toca tracking/detección, ejecutar un smoke test corto y reportar si no fue posible por dependencias o tiempo.
- Antes de responder al usuario, ejecutar el comando principal afectado y comprobar que no falla al inicio (al menos arranque sin excepción).
- Explicar claramente cualquier supuesto (modelo, video, rutas, parámetros de confianza).

## 9) Idioma y estilo de colaboración

- Por defecto, responder en español (salvo que el usuario pida otro idioma).
- Ser concreto: qué cambiaste, por qué, cómo ejecutarlo y limitaciones.

## 10) Documentación obligatoria

- Siempre que un cambio modifique comportamiento o configuración, actualizar `README.me` (y otros README relevantes si aplica) en la misma entrega.

---
> Source: [hectorgarciaa/narrador-futbol](https://github.com/hectorgarciaa/narrador-futbol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
