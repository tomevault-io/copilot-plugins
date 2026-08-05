## llm-creator-studio

> Un **curso-repositorio** para aprender a construir un LLM desde cero programando en

# CLAUDE.md — instrucciones para sesiones de Claude Code en este repo

## Qué es esto

Un **curso-repositorio** para aprender a construir un LLM desde cero programando en
PyTorch. No es una librería ni un tutorial para leer: el usuario abre VSCode, lee la
teoría, implementa funciones marcadas con `NotImplementedError` y ejecuta tests hasta que
pasan. El objetivo final es un GPT de **8.933.440 parámetros** entrenado por él sobre
TinyStories hasta que genere historias cortas coherentes en inglés.

## EL PUNTO MÁS IMPORTANTE: a quién le hablas

El usuario es **ingeniero de software con experiencia** (Python, git, CLI, arquitectura)
pero su **base en LLMs y machine learning es baja**, y el curso está pensado para gente en
esa misma situación. Separa las dos cosas o te equivocarás en una de las dos direcciones:

- **No le expliques programación.** Qué es un bucle, qué es una clase, qué hace `dict.get`.
  Eso le hace perder el tiempo.
- **Sí explícale todo lo de ML desde cero.** Qué es un logit, por qué se normaliza, qué
  significa que un gradiente "se vaya a cero". Nada se da por sabido.

La primera versión de este repo se escribió demasiado técnica y hubo que reescribirla. El
brief original pedía teoría "densa, sin analogías, 400-900 palabras"; esa instrucción quedó
**derogada** el 2026-07-30 tras ver el material. Lo que manda es lo de abajo.

### Estructura obligatoria de cada explicación: intuición → ejemplo → fórmula

Cada concepto de ML entra **tres veces y en este orden**:

1. **Qué problema resuelve**, en lenguaje llano. Sin fórmulas, sin jerga sin definir.
2. **Un ejemplo con números pequeños** que el lector pueda seguir a mano. Matrices de 2×3,
   tres palabras, cuatro conteos. Números concretos, no símbolos.
3. **La fórmula formal**, conectada explícitamente con el ejemplo anterior.

La matemática **no se elimina ni se esconde**: deja de ser lo primero que se lee. Un
`TEORIA.md` que abre con `C_token ≈ 6N + 12·n_layers·T·d_model` está mal escrito aunque la
fórmula sea correcta.

### Estructura obligatoria de cada docstring de ejercicio

Es la misma idea aplicada al código, y se estableció el 2026-07-31 después de que el usuario
dijera "yo leo esto y no sé qué tengo que hacer". El diagnóstico: los docstrings explicaban
**qué** era la función y **por qué** hacía falta, pero nunca **qué teclear**.

El orden es fijo, y **`QUÉ TIENES QUE ESCRIBIR` va primero, antes que cualquier teoría**:

```
<Una frase: qué hace la función>

QUÉ TIENES QUE ESCRIBIR
-----------------------
<Pasos numerados con el código concreto a teclear, indentado. No pseudocódigo:
 las líneas reales, con los nombres reales de las variables.>

QUÉ TIENE QUE SALIR / COMPRUÉBALO CON...
----------------------------------------
<Números concretos que el lector pueda verificar a mano>

POR QUÉ / DE DÓNDE SALE ESA FÓRMULA
-----------------------------------
<La intuición, ahora que ya sabe qué está escribiendo>

CUIDADO CON... / LOS ERRORES QUE HAY QUE EVITAR
-----------------------------------------------
<Las trampas, y sobre todo las silenciosas: las que no dan error y degradan el resultado>

Args: / Returns: / Raises:
```

Los 62 docstrings siguen este formato. Al añadir uno nuevo, el barrido de comprobación es:

```python
# recorre CURRICULUM y verifica que cada símbolo tiene la sección
"QUÉ TIENES QUE ESCRIBIR" in ast.get_docstring(nodo)
```

Dos reglas que salieron de escribirlos:

- **El código de los pasos tiene que compilar con lo que el alumno tiene importado.** Si tu
  paso usa `math.sqrt` y `ejercicios.py` no importa `math`, el paso está mal. Pasó de verdad
  en el módulo 12 (se cambió a `** 0.5`) y en el 01 (faltaba `import time` en el fichero).
- **Los números de ejemplo se miden, no se estiman.** Los del módulo 17 se inventaron
  plausibles y estaban mal por un factor de 5; hay que ejecutar la referencia y copiar.

## Reglas de escritura, innegociables

- **Toda la prosa en español**: teoría, docstrings, comentarios, mensajes de la CLI,
  nombres de los tests. **Los identificadores del código en inglés**: `causal_mask`,
  `n_heads`, `class MultiHeadAttention`. Esta mezcla es deliberada.
- **La prosa de los `.py` va sin tildes ni `ñ`**: `tamanyo`, `ensenyar`, `pequenyo`. Dos
  excepciones, y solo dos: los **títulos de sección en mayúsculas** de los docstrings sí las
  llevan (`QUÉ TIENES QUE ESCRIBIR`, `POR QUÉ ESA FÓRMULA`) porque son los puntos de anclaje
  al leer en diagonal, y los emoji de la CLI, que viven en `progress.py`. Los `.md` llevan
  tildes normales en todas partes.
- **Cada `TEORIA.md` abre con `## Por qué importa este módulo`**, ANTES de cualquier
  concepto: qué problema resuelve, qué sabrás al terminar, y cuánto cuesta. Hay un test que
  lo verifica. Alguien que no sabe de LLMs no puede juzgar si merece la pena leer cuatro
  horas sobre atención si no le dices primero que es LA pieza que separa un modelo mediocre
  de ChatGPT.
- **Mínimo 900 palabras por `TEORIA.md`, sin techo.** El límite superior se quitó el
  2026-07-31: cada concepto se explica lo que haga falta, con ejemplos concretos.
- **Cada `SOLUCION.md` termina con `## El código completo`**: la implementación entera de
  todos los ejercicios del módulo, copiable. Hay un test que lo verifica. El brief original
  decía "no el código pelado"; esa instrucción quedó derogada el 2026-07-31. Quien se
  atasca necesita ver el código, no leer sobre él.
- **El mismo registro en los docstrings de `ejercicios.py`, en `SOLUCION.md`, en las pistas
  de `hints.py` y en los mensajes de la CLI.**
- **Todo docstring de ejercicio abre con `QUÉ TIENES QUE ESCRIBIR`**, y después vienen el
  ejemplo, el porqué y las trampas. Ver la sección de abajo: es innegociable y hay un
  barrido que lo comprueba.
- **Todo término de ML que aparezca debe estar en `GLOSARIO.md`**, y cada `TEORIA.md`
  enlaza allí al final.
- Analogías: permitidas si son **mecánicas y verificables** (la ruleta para el muestreo, el
  reparto de una recta [0,1]). Prohibidas las místicas ("es como un cerebro", "entiende").
- **Honestidad intelectual obligatoria.** Donde hay debate abierto (pre-norm vs post-norm,
  si las leyes de escala se sostienen, qué construye realmente un modelo por dentro) se dice
  que lo hay, en una sección `## Dónde está el debate` al final de cada `TEORIA.md`. Se
  citan los papers originales con enlace, en la tupla `references` de `curriculum.py` y en
  el `TEORIA.md`.
- **Ejecuta lo que escribas.** Cada `demo.py` y cada test tiene que correr de verdad antes
  de darlo por hecho. Nada de código que "debería funcionar".
- **Valida los tests contra la referencia** con `make test-reference` antes de dar un módulo
  por terminado. Si un test falla en ese modo, el test está mal escrito o la referencia está
  mal: en ambos casos es un bug del curso, no del alumno.

## Arquitectura: tres capas que no hay que confundir

```
modulos/NN_*/ejercicios.py   ← lo que ESCRIBE EL USUARIO (plantillas con NotImplementedError)
llmfs/reference/             ← implementaciones canónicas, completas y correctas
llmfs/{model,train,infer}/   ← el código que de verdad entrena, construido sobre el bridge
```

`llmfs/bridge.py` es la pieza clave. Cuando el código de producción necesita
`MultiHeadAttention`, llama a `bridge.resolve("05_atencion", "MultiHeadAttention")`, que:

1. carga `modulos/05_atencion/ejercicios.py`,
2. comprueba con AST si el símbolo sigue siendo la plantilla (`raise NotImplementedError`
   o `pass` como único cuerpo) — ver `bridge.looks_unimplemented`,
3. ejecuta el probe de `llmfs/probes.py` si hay uno registrado,
4. devuelve la implementación del usuario si pasa, y si no la de `llmfs.reference`,
   avisando **una vez** por stderr.

Consecuencia: **el usuario nunca se queda bloqueado**, y cuando su ejercicio está bien, el
modelo final entrena con SU código. El aviso por stderr es deliberado y no se debe silenciar.

Variables de entorno útiles: `LLMFS_FORCE_REFERENCE=1` (ignora los ejercicios),
`LLMFS_BRIDGE_VERBOSE=1` (avisa también cuando usa el código del usuario),
`LLMFS_DEVICE=cpu|cuda|mps`, `LLMFS_AMP=0|1`, `LLMFS_ROOT`.

## Hardware objetivo — respétalo en cada decisión

| | |
|---|---|
| PC principal | Intel i7 7700K, 16 GB RAM, **RTX 2060 6 GB (Turing, sm_75)** |
| Portátil | MacBook Pro M5, 16 GB RAM, backend **MPS** |

Todo lo específico de hardware vive en `llmfs/device.py` y en ningún otro sitio. Lo que hay
que tener presente:

- **sm_75 no tiene bfloat16 en hardware.** Se usa `float16` + `GradScaler`. Y ojo:
  `torch.cuda.is_bf16_supported()` devuelve `True` en Turing contando emulación por
  software, así que **no se usa**; se mira la compute capability directamente.
- **Sin FlashAttention-2** (pide sm_80+). `F.scaled_dot_product_attention` cae solo al
  backend `memory_efficient`, que sí funciona en Turing.
- **`torch.compile` desactivado por defecto.** En Turing falla a compilar con frecuencia.
  Flag opcional (`compile: true` en el YAML), nunca por defecto.
- **MPS**: `PYTORCH_ENABLE_MPS_FALLBACK=1` lo fija `llmfs/__init__.py` *antes* de importar
  torch. fp32 por defecto, fp16 opcional. Algunos ops caen a CPU silenciosamente.
- El tensor de logits (`batch × ctx × 4096`) es el mayor consumidor de memoria de la
  tirada final, por encima de las activaciones. Si hay OOM en la 2060, ahí es donde mirar.

## Comandos

```bash
make install          # uv sync --extra compare
make test             # suite completa (tests/ + modulos/)
make test-fast        # sin los marcados @pytest.mark.slow
uv run pytest tests/  # solo la infraestructura del paquete

uv run python -m llmfs status        # tabla de progreso (corre los tests)
uv run python -m llmfs status --cached
uv run python -m llmfs next          # qué módulo toca y qué ejercicio
uv run python -m llmfs check 05      # tests del módulo 05 con pistas
uv run python -m llmfs demo 05       # experimento del módulo 05
uv run python -m llmfs hint 05 -e 2  # pista progresiva (repetir para más nivel)
uv run python -m llmfs device        # hardware detectado
```

El estado del currículo **no se declara en ningún sitio**: se calcula ejecutando los tests.
`.llmfs_progress.json` es solo caché (y está en `.gitignore`).

## Cómo añadir un módulo

1. Registrarlo en `llmfs/curriculum.py`: `Module(...)` con sus `Exercise(...)`,
   `est_minutes` honesto y `references` con enlaces a los papers.
2. Crear `modulos/NN_nombre/` con **los cinco ficheros**: `TEORIA.md`, `ejercicios.py`,
   `demo.py`, `test_NN.py`, `SOLUCION.md`. Hay un test que verifica que no falta ninguno.
   El `TEORIA.md` sigue la estructura intuición → ejemplo numérico → fórmula, cierra con
   `## Dónde está el debate` y enlaza al `GLOSARIO.md`.
3. Implementar las piezas en `llmfs/reference/` y reexportarlas en
   `llmfs/reference/__init__.py`. **Los nombres de ejercicio son únicos en todo el curso**
   (hay un test); el bridge resuelve por nombre plano.
4. Registrar probes en `llmfs/probes.py` para los ejercicios donde "escrito pero devuelve
   basura" sea un riesgo real (formas de tensores, sobre todo).
5. Escribir las tres pistas en `llmfs/hints.py`: conceptual → técnica → estructural.
   La tercera no da la solución escrita. Añadir los términos nuevos a `GLOSARIO.md`.
6. `ejercicios.py`: el docstring de módulo abre con **CÓMO SE HACE ESTE MÓDULO** (los 5
   pasos), **QUÉ VAS A CONSTRUIR** (un diagrama de cómo encajan los ejercicios) y
   **VOCABULARIO QUE VAS A NECESITAR** (cada término de ML que aparezca, definido en una
   línea). Los docstrings de cada ejercicio llevan formas de entrada/salida y la fórmula,
   cuerpo `raise NotImplementedError(...)`.
7. `test_NN.py`: valida contra `llmfs.reference` o contra el equivalente de PyTorch
   (`nn.MultiheadAttention`, `F.layer_norm`...) con `torch.allclose`. **No vale comprobar
   solo que no peta.** Los tests se importan con `from llmfs.bridge import exercises` y
   `ej = exercises(__file__)`, nunca con `sys.path`.
8. `demo.py`: experimento ejecutable que visualiza el concepto. Guarda figuras en
   `runs/figures/` vía `llmfs.paths.figures_dir()`. Tiene que correr en cuda, mps y cpu.
9. Regenerar el bloque de código de `SOLUCION.md` con
   `uv run python scripts/regenerar_soluciones.py` (lo extrae de `llmfs/reference/`, así que
   nunca diverge). Si el ejercicio necesita una función auxiliar o un import que no está en
   `ejercicios.py`, añadirlo a `AUXILIARES` o `IMPORTS` de ese script.
10. Correr `make test` y `make test-soluciones` antes de dar la fase por terminada. El
   segundo pega cada solución sobre su `ejercicios.py` y corre los tests: es la única forma
   de garantizar que el código de las soluciones se puede copiar y funciona.

## Dependencias

`torch`, `numpy`, `datasets`, `matplotlib`, `pytest`, `tqdm`, `pyyaml`, `rich`, `regex`.

- `regex` (no el `re` de la stdlib) porque el pre-tokenizador estilo GPT-4 del módulo 02
  usa `\p{L}`.
- `tiktoken` está en el extra `[compare]` y **solo** se usa en la comparativa de
  compresión del módulo 02.
- **Nada de `transformers` ni de HuggingFace para el modelo.** `datasets` se usa
  únicamente para descargar TinyStories.

## Estado del currículo

**18 módulos, numerados 00-17, 62 ejercicios, ~42 h de trabajo estimado.**

El `00_que_es_un_llm` se añadió el 2026-07-30 (no estaba en el brief original) y renumeró
todo lo demás: lo que el brief llamaba módulo NN es ahora NN+1. Es un módulo conceptual sin
torch donde se construye un generador de texto por conteo, y sirve de ancla para todo lo
que viene: el bucle autorregresivo del módulo 14 es literalmente el mismo.

| Fase | Contenido | Estado |
|---|---|---|
| 1 | Esqueleto, `llmfs`, CLI, bridge, tests de infraestructura | ✅ hecho |
| 2 | Módulos 00-04 (fundamentos) | ✅ hecho |
| 3 | Módulos 05-10 (baselines + arquitectura) | ✅ hecho |
| 4 | Módulos 11-13 (entrenamiento) | ✅ hecho |
| 5 | Módulos 14-17 (uso y evaluación) | ✅ hecho |

Al retomar: `make test` y `make test-reference` tienen que estar en verde antes de tocar
nada. `llmfs status` es siempre la fuente de verdad de qué módulos existen.

**Las Partes 0, I y II están terminadas: módulos 00-10.** El modelo está montado y auditado;
`GPT(ModelConfig())` da 8.933.440 parámetros exactos y `expected_param_count` coincide con
`count_parameters`.

**EL CURSO ESTÁ COMPLETO: los 18 módulos, 62 ejercicios.** 98 tests de infraestructura y
518 del curso contra la referencia, todos en verde. 17 figuras generadas de ejecuciones
reales.

`uv run python -m llmfs train --config tiny_char` entrena de verdad (~70 s en MPS, pérdida
de 3,2 a 1,60, 112k tokens/s) y genera Shakespeare reconocible.

Lo que queda pendiente, si se retoma:
- El pipeline de TinyStories con BPE real (`llmfs/data/prepare.py` solo tiene
  `preparar_shakespeare`; `preparar()` lanza `NotImplementedError` para el resto).
- El comando `llmfs sample` sigue siendo un stub (`_LATER` en `llmfs/cli.py`).
- El servidor FastAPI del módulo 17 se describe en la teoría pero no se implementa.
- La tirada real de 500M tokens en la RTX 2060 no se ha ejecutado (no hay hardware CUDA
  aquí). Los tiempos del README para esa tirada están marcados como estimación.

Hallazgos que conviene no repetir:
- `named_parameters()` **deduplica por defecto** (`remove_duplicate=True`). La creencia común
  de que devuelve los pesos atados dos veces es falsa.
- Un test que pasa `modelo(idx, idx)` (targets sin desplazar) produce una fuga de información
  y una pérdida por debajo de `ln(V)`. El síntoma es idéntico a una máscara causal rota, así
  que ante ese síntoma hay que mirar primero cómo se monta el batch.
- `x in lista_de_tensores` revienta con "Boolean value of Tensor is ambiguous", porque `in`
  usa `==` y en tensores eso devuelve comparación elemento a elemento. Comparar por `id()`.
- El guardia `test_ningun_fichero_de_test_define_dos_veces_el_mismo_test` ha cazado ya
  **nueve** duplicados de `test_coincide_con_la_referencia`. Al escribir un `test_NN.py` con
  varios ejercicios, nombrar cada test con el ejercicio al que pertenece
  (`test_adamw_coincide_con_la_referencia`), nunca genérico.

Los directorios de `llmfs/` (`model/`, `train/`, `infer/`, `eval/`, `viz/`, `tokenizer/`,
`data/`) se crean en la fase que los llena, no antes: en este repo no existe nada que no
funcione. `llmfs status` es siempre la fuente de verdad de qué hay hecho.

Los subcomandos `llmfs train`, `llmfs sample` y `llmfs data` existen como stubs que
explican en qué módulo se construyen. Al implementarlos, quitar la entrada
correspondiente de `_LATER` en `llmfs/cli.py`.

---
> Source: [roberottt/llm-creator-studio](https://github.com/roberottt/llm-creator-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
