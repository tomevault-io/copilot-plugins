## powerlifting-cnn

> Este proyecto es una **tesis** cuyo objetivo es crear un **juez virtual para powerlifting** utilizando visión por computador y redes neuronales. Se analizan tres levantamientos compuestos:

# Proyecto de Tesis: Juez Virtual de Powerlifting

## Descripción General

Este proyecto es una **tesis** cuyo objetivo es crear un **juez virtual para powerlifting** utilizando visión por computador y redes neuronales. Se analizan tres levantamientos compuestos:

- **Sentadilla (sq)** — Squat
- **Press de Banca (bp)** — Bench Press
- **Peso Muerto (dl)** — Deadlift

Cada levantamiento se graba desde **3 vistas sincronizadas**: frontal (front), lateral izquierda (left) y lateral derecha (right).

### Pipeline completo (planeado)

1. **Detección de esqueletos** → YOLO pose / MediaPipe
2. **Extracción de keypoints** → Archivos `.npy`
3. **Clasificación de levantamientos** → Red neuronal que toma los keypoints y determina si un levantamiento es válido o nulo (y el tipo de falta)

Por ahora estamos en la **etapa 1** (esqueletos con YOLO/MediaPipe). Próximamente integraremos los keypoints con datos etiquetados en **CSV** que contienen: válido/nulo + tipo de falta.

---

## Estructura del Proyecto

```
MediapipePythonProjects/
├── script.py                          # Organizador de dataset (mueve archivos a front/left/right)
├── .gitignore                         # Ignora .venv, .idea, dataset/, display/pruebas
│
├── extract_keypoints/                 # Scripts de extracción de keypoints
│   ├── bp/                            # (vacío - pendiente)
│   ├── dl/                            # (vacío - pendiente)
│   └── sq/
│       ├── extract_keypoints_front_sq.py   # MediaPipe: 33 landmarks → (T, 22, 3) vista frontal
│       ├── extract_keypoints_left_sq.py    # MediaPipe: cadera/rodilla/tobillo izq → (T, 3, 3)
│       ├── extract_keypoints_right_sq.py   # MediaPipe: cadera/rodilla/tobillo der → (T, 3, 3)
│       └── extract_bar_features.py         # Tracking de barra con CSRT + velocidad/aceleración
│
├── keypoints/                         # Keypoints extraídos (.npy)
│   └── sq/
│       ├── front/   (316 archivos)
│       ├── left/    (245 archivos)
│       └── right/   (274 archivos)
│
├── display/                           # Visualización y demos
│   ├── bp/
│   │   ├── mediapipe_skeleton_bp_front.py      # Esqueleto MediaPipe BP frontal
│   │   ├── mediapipe_skeleton_bp_lateral.py     # Esqueleto MediaPipe BP lateral (smooth EMA)
│   │   ├── yolo_skeleton_bp_front.py           # YOLO pose BP frontal (6 keypoints)
│   │   ├── yolo_skelton_bp_side.py             # YOLO pose BP lateral (8 keypoints)
│   │   ├── yolo_seg_model.py                   # YOLO segmentación BP frontal
│   │   └── recording_processed_bp.py           # Grabación multi-view sincronizada con MediaPipe
│   ├── dl/
│   │   ├── mediapipe_skeleton_dl_front.py      # Esqueleto MediaPipe DL frontal (cuerpo completo)
│   │   ├── mediapipe_skeleton_dl_right.py      # MediaPipe DL derecha (hombro-cadera)
│   │   └── mediapipe_skeleton_left.py          # MediaPipe DL izquierda (hombro-cadera)
│   └── sq/
│       ├── mediapipe_skeleton_sq_front.py      # Esqueleto MediaPipe SQ frontal (33 landmarks)
│       ├── mediapipe_skeleton_sq_right.py      # MediaPipe SQ derecha (pierna der)
│       ├── mediapipe_skeleton_sq__left.py      # MediaPipe SQ izquierda (pierna izq)
│       ├── yolo_skeleton_sq_front.py           # YOLO pose SQ frontal (14 keypoints + EMA smooth)
│       ├── yolo_seg_sq_front.py                # YOLO segmentación SQ frontal
│       ├── testing.py                          # Máquina de estados para validar profundidad SQ
│       └── bar_grafic.py                       # Gráfico de movimiento vertical de barra
│
├── live/                               # Cámara en vivo
│   ├── live.py                         # Cámara índice 3
│   ├── live2.py                        # Cámara índice 1
│   └── live3.py                        # Cámara índice 2
│
├── models/                             # Modelos entrenados
│   ├── pose_landmarker_heavy.task       # MediaPipe Pose Landmarker
│   ├── bp_front_seg_v1.pt              # YOLO segmentación BP frontal
│   ├── bp_front_skeleton_v[3-6].pt     # YOLO pose BP frontal (varias versiones)
│   ├── bp_side_skeleton_v1.pt          # YOLO pose BP lateral
│   ├── bp_object_model.pt              # YOLO detección objetos BP
│   ├── sq_front_seg_v1.pt             # YOLO segmentación SQ frontal
│   └── sq_front_skeleton_v[1-3].pt    # YOLO pose SQ frontal (varias versiones)
│
└── runs/                               # Entrenamientos YOLO
    └── pose/
        ├── train/                      # Entrenamiento 1
        ├── train-2/                    # Entrenamiento 2
        └── train-3/                    # Entrenamiento 3
```

---

## Dataset

| Ejercicio | Vista Front | Vista Left | Vista Right | Total |
|-----------|-------------|------------|-------------|-------|
| BP (Press Banca) | 492 | 487 | 481 | 1,460 |
| DL (Peso Muerto) | 259 | 150 | 150 | 559 |
| SQ (Sentadilla) | 375 | 245 | 274 | 894 |
| **Total** | **1,126** | **882** | **905** | **~2,913** |

Los videos están en `dataset/<ejercicio>/<vista>/` y están ignorados por git.

---

## Dependencias Principales

| Paquete | Versión | Uso |
|---------|---------|-----|
| mediapipe | 0.10.31 | Detección de pose landmarks |
| ultralytics | 8.4.51 | YOLOv8/26 para pose y segmentación |
| torch | 2.13.0.dev+cu130 | PyTorch con CUDA 13 |
| opencv-python | 4.13.0 | Procesamiento de video |
| opencv-contrib-python | 4.13.0 | CSRT tracker (barra) |
| numpy | 2.2.6 | Arrays numéricos y .npy |
| matplotlib | 3.10.8 | Gráficos |
| scipy | 1.15.3 | Gradientes numéricos |
| roboflow | 1.3.8 | Gestión de datasets |
| tqdm | 4.67.3 | Barras de progreso |

---

## Estado Actual

### ✅ Completado
- [x] Dataset de ~2,913 videos en 3 ejercicios × 3 vistas
- [x] Extracción de keypoints con MediaPipe para **sentadilla** (front, left, right)
- [x] Tracking de barra con CSRT (sentadilla)
- [x] Modelos YOLO entrenados para **pose** (BP frontal v3-v6, BP lateral v1, SQ frontal v1-v3)
- [x] Modelos YOLO entrenados para **segmentación** (BP frontal v1, SQ frontal v1)
- [x] Visualizaciones de esqueleto con MediaPipe y YOLO para los 3 ejercicios
- [x] Herramienta de grabación multi-view sincronizada para BP
- [x] Máquina de estados para validación de profundidad en sentadilla

### 🔄 En Progreso
- [ ] Esqueletos con YOLO para los 3 levantamientos (sentadilla y press de banca avanzan bien)

### ❌ Pendiente
- [ ] Extracción de keypoints para **press de banca** (bp)
- [ ] Extracción de keypoints para **peso muerto** (dl)
- [ ] Integración con **CSV etiquetados** (válido/nulo + tipo de falta)
- [ ] Red neuronal clasificadora de levantamientos
- [ ] Evaluación de modelos (métricas)
- [ ] README / documentación
- [ ] requirements.txt o pyproject.toml

---

## Convenciones del Código

- **Idioma**: Comentarios y variables en **español**
- **Estilo**: Sin tipado explícito, Python simple con numpy/opencv/mediapipe
- **Keypoints guardados como**: `.npy` con forma `(T, N, 3)` donde:
  - `T` = frames
  - `N` = landmarks seleccionados
  - `3` = (x, y, visibility)
- **Rutas de modelos**: Relativas desde el script (`../../models/...`)
- **Parámetros de entrenamiento YOLO**: img=960, batch=8, AdamW(lr=0.001), pose_weight=12.0

---

## Próximos Pasos (Hoja de Ruta)

1. Terminar extracción de keypoints con MediaPipe para BP y DL
2. Entrenar/mejorar modelos YOLO pose para los 3 levantamientos
3. Obtener/aumentar dataset etiquetado en CSV con decisiones de jueces
4. Alinear secuencias de keypoints con etiquetas CSV
5. Diseñar y entrenar red neuronal clasificadora (LSTM, Transformer, o MLP sobre ventanas temporales)
6. Evaluar y documentar resultados para tesis

---
> Source: [bvstt1/powerlifting-cnn](https://github.com/bvstt1/powerlifting-cnn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
