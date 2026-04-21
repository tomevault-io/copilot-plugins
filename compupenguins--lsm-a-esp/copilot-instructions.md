## lsm-a-esp

> Construir un sistema que traduzca **Lengua de Señas Mexicana (LSM)** a **lenguaje natural en español**, priorizando una representación basada en **landmarks** (en lugar de píxeles crudos de video).

# CLAUDE.md — Contexto operativo del proyecto LSM → Español

## 1) Objetivo del proyecto
Construir un sistema que traduzca **Lengua de Señas Mexicana (LSM)** a **lenguaje natural en español**, priorizando una representación basada en **landmarks** (en lugar de píxeles crudos de video).

### Motivación técnica
- Reducir costo de cómputo en entrenamiento/inferencia frente a video crudo.
- Disminuir sensibilidad a condiciones de cámara (color, brillo, contraste, fondo).
- Estandarizar insumos como series de tiempo de keypoints para modelos secuenciales.

---

## 2) Resumen del estado actual
- Dataset base: Kaggle con **249 glosas** y ~**5 señadores por glosa** (aprox.).
- Extracción principal offline de keypoints con **RTMPose/RTMW (rtmlib)** por su mayor precisión en videos de baja calidad.
- Salida consolidada en **Parquet** bajo `corpus_LSM_esp/`.
- Formato por fila = 1 video, incluyendo metadatos y series temporales serializadas de landmarks.
- Para tiempo real/webcam se usa **MediaPipe Holistic** por mejor balance rendimiento/precisión en hardware común.
- Existe notebook de EDA con análisis de duración, detección, calidad por región y outliers de duración.

---

## 3) Arquitectura (alto nivel)

### 3.1 Pipeline offline (dataset)
1. **Descubrimiento de videos** en estructura `MSLwords1/{glosa}/{intento}/...`.
2. **Inferencia frame a frame** con RTMPose (133 keypoints COCO-WholeBody).
3. **Construcción de tensor temporal** `(T, 133, 3)` con `[x, y, score]`.
4. **Máscara de detección** `person_detected` de tamaño `(T,)`.
5. **Serialización** con `np.save` a bytes.
6. **Persistencia en Parquet** comprimido (`zstd`).

### 3.2 Pipeline online (tiempo real)
1. Captura de webcam con OpenCV.
2. Estimación con MediaPipe Holistic.
3. Visualización en vivo de pose/manos.
4. (Pendiente) conexión de esta salida con un clasificador de glosa en tiempo real.

---

## 4) Estructura actual del repositorio (funcional)

- `build_lsm_dataset.py`  
  Script principal para construir parquet completo desde `MSLwords1/`.
  - Soporta `--resume`, `--device`, `--mode`, `--batch-size`.
  - Esquema con columnas de metadatos + blobs (`keypoints`, `person_detected`).

- `video_to_landmarks.py`  
  Extracción de landmarks para uno o varios videos con RTMPose y guardado en parquet.

- `camera.py`  
  Prototipo de captura en vivo con MediaPipe Holistic.

- `interpolate_landmarks.py` y `interpolate_gaps.py`  
  Postproceso de interpolación de gaps cortos (actualmente orientado a CSV/versión previa).

- `notebooks/eda_lsm_dataset.ipynb`  
  EDA del parquet: distribución, detección, calidad por regiones, outliers de duración, comparativas.

- `corpus_LSM_esp/`  
  Artefactos de dataset (incluyendo parquet versionado con DVC).

---

## 5) Especificación de datos clave

## 5.1 Esquema parquet (dataset consolidado)
Columnas relevantes observadas en el proyecto:
- `glosa` (str)
- `intento` (str)
- `video_id` (str)
- `video_path` (str, en algunos pipelines)
- `fps` (float32)
- `total_frames` (int32)
- `width` (int32)
- `height` (int32)
- `keypoints` (bytes): tensor `float32` shape `(T, 133, 3)`
- `person_detected` (bytes): vector `bool` shape `(T,)`

## 5.2 Convención de landmarks
- Formato COCO-WholeBody de 133 puntos.
- Cada punto: `[x, y, score]`.
- El `score` es crítico para enmascarar / filtrar ruido / imputar.

---

## 6) Decisiones de diseño actuales
- **RTMPose offline**: más costoso pero robusto para videos con calidad variable.
- **MediaPipe en real-time**: latencia menor y suficiente para webcam.
- **Parquet + bytes serializados**: equilibrio entre portabilidad y tamaño de dataset.
- **Representación temporal explícita**: facilita modelos secuenciales (LSTM/Transformer/TCN).

---

## 7) Objetivos inmediatos (roadmap corto)

### A) Aumentación orientada a landmarks
Diseñar y evaluar augmentations temporal-espaciales que preserven semántica de la seña:
- **Temporal**: time-warp suave, jitter temporal, frame-drop controlado, speed perturbation.
- **Espacial geométrica**: traslación, escalado, rotación pequeña, ruido gaussiano leve en coordenadas.
- **Oclusión sintética**: masking parcial de manos/cara usando scores o dropout por región.
- **Score-aware augmentation**: degradaciones realistas condicionadas al confidence.

### B) Entrenamiento de reconocedor de glosas
Comparar arquitecturas:
- Baselines: MLP temporal simple, 1D-CNN.
- Secuenciales: **BiLSTM/GRU**.
- Atención: **Transformer Encoder** (máscara temporal y padding).
- (Opcional) GCN/ST-GCN con grafo corporal/manos.

### C) Limpieza profunda con DTW
- Detectar intentos atípicos intra-glosa comparando trayectorias temporales por señador.
- Medidas candidatas: DTW global y por subregión (manos, cuerpo, cara).
- Salida esperada: score de consistencia por muestra + flags para revisión/eliminación.

---

## 8) Métricas recomendadas

### Clasificación de glosa
- Top-1 accuracy
- Top-k accuracy (k=3,5)
- Macro-F1 (por desbalance entre glosas)
- Matriz de confusión y análisis de glosas vecinas

### Calidad de landmarks/dataset
- % frames sin persona detectada
- % landmarks bajo umbral de score por región
- Outliers de duración intra-glosa
- Distancia DTW intra-glosa (percentiles por clase)

---

## 9) Riesgos y deuda técnica actual
- `interpolate_gaps.py` y `interpolate_landmarks.py` parecen duplicados/legado CSV.
- `README.md` minimal: falta guía reproducible end-to-end.
- `main.py` todavía placeholder.
- Falta módulo de entrenamiento formal (`src/`, configs, experiment tracking).
- Falta protocolo claro de split (por señador/video) para evitar leakage.

---

## 10) Requisitos de prompts futuros (para mejor calidad)
Cuando pidas ayuda al asistente, incluye explícitamente:
1. **Objetivo puntual** (ej. “entrenar baseline Transformer para 249 glosas”).
2. **Input exacto** (ruta parquet, columnas disponibles, tamaño de muestra).
3. **Restricciones** (GPU/CPU, RAM, tiempo máximo de entrenamiento).
4. **Métrica principal** (macro-F1 o top-1, etc.).
5. **Criterio de aceptación** (ej. superar baseline LSTM por +3% macro-F1).

Plantilla sugerida:
- Tarea:
- Datos de entrada:
- Supuestos:
- Recursos disponibles:
- Métricas objetivo:
- Entregables esperados (scripts, notebook, reporte):

---

## 11) Próximos hitos recomendados (orden sugerido)
1. Estandarizar loader de parquet a tensores (`Dataset` + `collate_fn`).
2. Definir split robusto (idealmente por señador si metadata lo permite).
3. Implementar baseline LSTM reproducible.
4. Implementar paquete de augmentations para landmarks.
5. Correr benchmark LSTM vs Transformer.
6. Integrar módulo de limpieza DTW para curación de muestras atípicas.
7. Documentar pipeline de entrenamiento e inferencia en README técnico.

---

## 12) Definición operativa del “estado actual”
Proyecto en fase de **consolidación de datos + EDA avanzada**, antes de una fase formal de **modelado supervisado de reconocimiento de glosas**.

En otras palabras:
- La base de datos de landmarks ya existe y es utilizable.
- El siguiente salto de valor está en: **augmentación**, **entrenamiento**, **limpieza por DTW** y **estandarización de experimentos**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CompuPenguins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
