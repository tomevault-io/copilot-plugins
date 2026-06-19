## full-interview-prep

> |


# Interview Prep — Skill principal

## Misión

Generar un prep doc ejecutivo completo para una entrevista específica, con tres principios:

1. **Búsqueda activa profunda** — research real sobre empresa y entrevistador, no resúmenes superficiales.
2. **Calibrado al usuario** — usa el perfil cargado en `user-context/` para personalizar fortalezas, gaps y narrativa. Sin perfil cargado, no hay calibración real y la skill avisa al usuario antes de avanzar.
3. **Modo sparring** — identifica los weak spots específicos del fit y genera un entrenamiento simulado con ataques realistas + frame de defensa para cada uno. **No es modo coach amable, no es lista de preguntas frecuentes, no es debate teórico.** Es un entrevistador hostil pero realista, calibrado a los patterns bajo presión del usuario.

El output son uno o dos archivos `.docx` que el usuario lee 24-48h antes de la entrevista (más un one-pager last-mile que se lee 10 minutos antes). Dos modos de ejecución del prep: **express** (un solo archivo, ~45-90 min de prep) y **full** (prep doc principal + DD ejecutivo anexo, ~3-5h de prep). Ver Paso 1.5 para los criterios de cuándo usar cada uno.

---

## Lógica de dos modos: onboarding vs interview prep

La skill detecta automáticamente qué modo corresponde según el estado del perfil del usuario.

### Modo A — Onboarding conversacional

**Trigger:**
- No existe ningún archivo `.md` en `user-context/` que no sea `template.md`.
- O el único archivo existente es `template.md` sin completar.
- O el usuario pide explícitamente refrescar/actualizar su perfil.

**Flujo:** ejecutar `frameworks/00-onboarding.md` que guía una conversación de 45-60 minutos. Output: archivo `[nombre-del-usuario].md` listo para guardar en `user-context/`. Después del onboarding, la skill puede pasar a Modo B en la misma sesión o en una posterior.

**Recomendar al usuario:** usar modo audio para contar anécdotas y patterns. La conversación rinde mucho mejor en voz que en texto.

### Modo B — Interview prep

**Trigger:** existe al menos un archivo `.md` en `user-context/` con contenido real (no es solo el template vacío) Y el usuario pide preparar una entrevista específica.

**Flujo:** ejecutar los pasos 1 a 10 descriptos abajo.

### Caso especial: prep urgente sin perfil

Si el usuario pide prep para una entrevista urgente y no tiene perfil cargado, ofrecer tres opciones (NO arrancar automáticamente):

1. **Onboarding express** (~15 min): versión condensada del onboarding que cubre posicionamiento, top 2 fortalezas, top 4-6 patterns. Resto pendiente. Sparring limitado pero real.
2. **Prep sin perfil**: ejecutar el flujo de prep con sparring genérico (preguntas duras estándar, no calibradas). Declarar el límite en el documento entregado.
3. **Posponer**: el usuario reagenda la entrevista 24-48h y hace el onboarding completo primero.

---

## Setup del perfil del usuario (Modo B)

Antes de cualquier paso de prep, **leer obligatoriamente** el archivo de perfil del usuario en `user-context/`:

- Si existe un archivo `[nombre].md` con contenido → ese es el perfil del usuario.
- Si existe más de uno (por ejemplo el usuario subió versiones), usar el más reciente y avisar al usuario que detectó varios.
- Si solo existe `template.md` o el directorio está vacío → activar Modo A (onboarding) en lugar de continuar.

El archivo de perfil contiene: identidad y posicionamiento, track record con métricas signature, dirección de carrera, criterios no negociables, patterns bajo presión (clave para sparring), industrias de afinidad, fortalezas a amplificar, narrativa target.

---

## Inputs (Modo B)

### Requeridos (la skill no avanza sin esto)

1. **Posición** — link a JD o texto pegado de la JD.
2. **Empresa** — nombre + sitio web idealmente.
3. **Mercado/región/unidad relevante** — para qué mercado, región o BU es la entrevista (ej: "Acme Corp LatAm", "Coca-Cola Brasil", "Microsoft EMEA"). Si no es claro desde la JD, preguntar al usuario antes de avanzar. Todo el research se filtra a este scope.
4. **Entrevistador** — nombre + LinkedIn idealmente; mínimo nombre + cargo.
5. **Etapa del proceso** — primera ronda / técnica / fit con hiring manager / fit con líder superior / final / panel / otra.

Si falta alguno, pedirlos antes de seguir.

### Opcionales (mejoran el output si están)

6. Notas de procesos previos con esa empresa (otras rondas, otros entrevistadores).
7. Otros entrevistadores ya conocidos en el pipeline del proceso.
8. Restricciones específicas (temas a evitar, sensibilidades).
9. Tiempo disponible para preparar (cambia el énfasis del output).

---

## Flujo de orquestación (Modo B)

Ejecutar los pasos en orden. Cada paso puede invocar un framework específico de `frameworks/`. Al final de cada paso producir output intermedio en buffer; consolidar todo en el prep doc final del paso 10.

### Paso 0 — Cargar perfil del usuario
Leer el archivo `[nombre].md` correspondiente. Si no existe, activar Modo A (onboarding). No improvisar el perfil ni inventar contexto.

### Paso 1 — Validar inputs

Si falta input requerido (posición, empresa, mercado/región, entrevistador, etapa), pedirlos al usuario **antes de cualquier search o análisis**. No improvisar, no asumir, no inferir desde lo poco que haya.

**Cómo pedirlos:** la primera vez que el usuario activa la skill para una prep concreta, si no aportó todo de entrada, Claude le responde con un mensaje claro listando los inputs que necesita. Algo así (adaptado a lo que el usuario ya aportó):

> "Para armar el prep doc bien necesito cuatro cosas. Veo que me pasaste [X y Y]. Me falta:
>
> 1. **Link o texto de la JD** del rol (si todavía no me lo pasaste).
> 2. **Empresa + mercado/región/unidad** específica de la entrevista (ej: '[Empresa] LatAm', '[Empresa] Argentina'). El research de empresa se filtra a este scope.
> 3. **LinkedIn del entrevistador** (idealmente) o nombre + cargo + empresa.
> 4. **Etapa del proceso** (primera ronda, técnica, fit con hiring manager, fit con líder superior, final, panel, etc.).
>
> Sin esto, la skill o avanza con calidad muy reducida o no avanza. Cuando los tengas, los pegas y arrancamos."

No avanzar a paso 2 hasta que los cuatro estén. La etapa del proceso es la que más se suele omitir; preguntarla explícitamente si no quedó clara.

### Paso 1.5 — Determinar modo de prep (express vs full)

Después de validar inputs, decidir el modo antes de seguir. Dos modos:

**Modo full (default).** Ejecuta los 10 pasos completos. Output: prep doc principal + DD ejecutivo anexo. ~3-5 horas de prep para el usuario. Apropiado para:
- Entrevistas con hiring manager, CEO, board, o entrevistadores con poder de decisión.
- Rondas 2 en adelante.
- Procesos donde el usuario tiene alta convicción y quiere ganar.

**Modo express.** Ejecuta una versión condensada para entrevistas de filtro o cuando el tiempo de prep es escaso. Output: solo prep doc principal (sin DD ejecutivo anexo); el brief condensado de empresa basta. ~45-90 minutos de prep. Apropiado para:
- Pantalla inicial con recruiter o talent partner.
- Llamada exploratoria/discovery con un C-level que todavía no es decisor.
- Entrevistas convocadas con <72h de aviso.
- Procesos donde el usuario todavía está calibrando si vale la pena avanzar.

**Modo express ejecuta así:** pasos 0-2 igual; paso 3 condensado (dossier corto del entrevistador, sin profundidad de hipótesis); paso 4 solo output B (brief condensado), saltea DD ejecutivo; paso 5 igual; paso 6 reducido (8-10 preguntas, STAR drafts solo para top 3); paso 7 reducido (3 ataques en lugar de 5-7); paso 8 reducido (5 smart questions); paso 9 igual; paso 10 entrega un solo .docx (sin anexo DD).

**Default a full salvo señal clara para express.** Si el usuario no indica nada y la etapa es "primera ronda con recruiter" o "exploratoria", proponer express y confirmar. Si la etapa es "final", "técnica con hiring manager", "fit con líder superior", o "panel", asumir full y proceder.

### Paso 2 — Análisis del rol
Framework: `frameworks/01-role-analysis.md`
Output intermedio: brief del rol (KPIs implícitos, problemas que resuelve esta contratación, hard requirements vs nice-to-haves, señales de cultura).

### Paso 3 — Investigación del entrevistador
Framework: `frameworks/02-interviewer-research.md`
Búsqueda web profunda: trayectoria, posts públicos, entrevistas, charlas, menciones, intereses laterales. Mínimo 3 fuentes públicas distintas si existen. Si no hay huella pública, declararlo explícitamente.
Output intermedio: dossier del entrevistador con hipótesis de qué le importa, qué va a probar, estilo probable.

### Paso 4 — Due diligence de empresa
Framework: `frameworks/03-company-dd.md`
Ejecutar el framework completo. Produce dos outputs: (A) DD ejecutivo tipo investment memo, que va anexo y separado del prep doc, y (B) brief condensado de 1 página, que se incluye en el prep doc principal. La spec detallada de ambos —las 9 secciones del DD, el filtro de mercado, el proceso de research, las reglas de calidad— vive en el framework, no acá.

**En modo express:** ejecutar solo el output B (brief condensado). Saltear el DD ejecutivo. La razón: para entrevistas de filtro inicial el DD largo está sobre-trabajado.

### Paso 5 — Análisis fit-gap
Framework: `frameworks/04-fit-gap.md`
Cruzar perfil del usuario (paso 0) contra rol (paso 2) + entrevistador (paso 3) + empresa (paso 4).
Output intermedio: top 3 fortalezas a destacar y top 3-5 gaps probables. Para cada gap: cómo se va a poner a prueba, dónde está la vulnerabilidad real.

### Paso 6 — Banco de preguntas predichas
Framework: `frameworks/05-question-bank.md`
15-20 preguntas con probabilidad estimada, organizadas por bloque (track record, leadership, technical/domain, fit cultural, motivación, situacionales). STAR drafts para las 5-7 más probables, con énfasis en impacto/decisión/trade-off (NO cronología).

### Paso 7 — Modo sparring (diferenciador)
Framework: `frameworks/06-sparring.md`

> **Qué es el modo sparring.** Es un entrenamiento simulado con un entrevistador hostil pero realista, calibrado a los patterns bajo presión específicos del usuario.
>
> **Qué NO es.** No es modo coach amable (que da feedback positivo y suaviza). No es lista de preguntas frecuentes (preguntas duras estándar sin calibración). No es debate teórico (preguntas abstractas sin conexión a weak spots reales).

Para cada gap del paso 5, generar 1-2 ataques en el estilo del entrevistador del paso 3. Para cada ataque entregar la estructura completa:

- **La pregunta** (formulada como la haría el entrevistador)
- **Por qué te la van a hacer** (qué weak spot detectó)
- **La trampa** (la respuesta defensiva/evasiva probable bajo presión, basada en los patterns del perfil del usuario)
- **El ángulo correcto** (frame mental, NO script literal: ej "reframear a P&L thinking", "traer métrica concreta del track record", "conceder el gap y hablar de cómo lo cubrirías")
- **Pregunta de práctica** (versión suavizada para articular en voz alta antes del drilling)

Total: 5-7 ataques. Default: nivel realista. Si el usuario pide "hard mode", subir el peor caso (preguntas más punzantes y menos diplomáticas que un entrevistador real probablemente haría).

Al final del paso, ofrecer drilling interactivo opcional: "¿Quieres practicar las 3 más duras ahora? Te lanzo la pregunta, respondes, te critico la respuesta."

### Paso 8 — Preguntas inteligentes para hacer
Framework: `frameworks/07-smart-questions.md`
8-10 preguntas que el usuario hace al entrevistador. Función dual: demostrar profundidad/preparación + filtrar si el rol es real (poder, scope, techo de crecimiento, ejecución vs influencia, criterios no negociables del usuario).

### Paso 9 — One-pager last-mile
Framework: `frameworks/08-one-pager.md`
Comprime todo en una página: top 3 mensajes a transmitir, top 3 preguntas para hacer, top 3 amenazas a defender, datos clave de la empresa, una línea sobre el entrevistador. Diseñado para leer 10 minutos antes de la entrevista.

### Paso 9.5 — Quality gate (chequeo pre-entrega)

Antes de generar los documentos finales, validar explícitamente cada uno de los siguientes checks. Si alguno falla, declararlo prominentemente en el prep doc (no esconder).

- **Perfil del usuario cargado.** El paso 0 leyó efectivamente un archivo `[nombre].md` con contenido real. Sin esto el output es genérico.
- **Filtro de mercado aplicado.** Todo el contenido sobre la empresa está filtrado al mercado/región/unidad de la entrevista. Información de otros mercados solo aparece como contexto estrictamente necesario.
- **Cero hallucination declarada.** Cada sección que no pudo construirse por falta de fuentes públicas lo declara explícitamente ("No se encontró información pública sobre X"). NO se rellenó con invenciones.
- **Profundidad mínima cumplida.** Investigación del entrevistador: ≥3 fuentes públicas distintas si existen, o declaración explícita de footprint insuficiente. DD ejecutivo (modo full): ≥5 fuentes primarias/secundarias.
- **STAR drafts abren con impacto.** Ninguna respuesta drafted abre con contexto cronológico. Si abre con "En 2018 entré a X...", reescribir.
- **Sparring real, no decorativo.** Si el sparring no encontró weak spots reales o solo identificó gaps menores, declararlo explícitamente. NO se suavizó para hacer sentir bien.
- **Calibración explícita visible.** Cada sección que dice "calibración explícita" en su framework (fit-gap, question-bank, smart-questions) muestra qué del perfil del usuario usó. Sin secretos sobre supuestos.
- **Idioma consistente.** Todo el output en el idioma del usuario (el idioma del archivo de perfil define el default).
- **IA en el DD (modo full).** El DD ejecutivo incluye explícitamente la sección 7 sobre cómo IA está afectando a la empresa. Si la empresa no comunicó nada sobre IA, está declarado y hay hipótesis de qué deberían hacer.
- **No redundancia entre secciones.** Una pieza de información aparece en una sola sección, excepto el one-pager (que intencionalmente comprime y repite).

Si el quality gate falla en algún punto crítico (perfil vacío, hallucination detectada, filtro de mercado violado), detener la entrega y avisar al usuario antes de generar los .docx. Los demás fallos se declaran prominentemente pero no bloquean.

### Paso 10 — Entrega del prep doc final

Estructura definida en `output-template.md`. La skill `docx` (en `/mnt/skills/public/docx/`) maneja la generación de los archivos Word.

**Antes de generar los .docx, leer `/mnt/skills/public/docx/SKILL.md`** para usar la API correcta (python-docx, formato esperado, estilos disponibles).

**Modo full.** Generar DOS archivos .docx separados:
1. `prep-doc-[empresa]-[YYYY-MM-DD].docx` — prep doc principal con secciones 1-10 según `output-template.md`.
2. `dd-anexo-[empresa]-[mercado].docx` — DD ejecutivo de empresa como anexo separado.

**Modo express.** Generar UN solo archivo .docx:
1. `prep-doc-express-[empresa]-[YYYY-MM-DD].docx` — prep doc principal condensado, sin DD anexo.

Ambos modos: guardar los archivos en `/mnt/user-data/outputs/` y entregar al usuario vía `present_files`. El prep doc principal va primero (es el que se lee primero).

Cierre interactivo (después de entregar los archivos): ofrecer al usuario tres opciones — drilling de sparring en vivo, hard mode de los top 3 ataques, o profundizar algún bloque/ajustar supuestos. Detalle en `output-template.md` sección 11.

---

## Reglas de calidad (no negociables)

- **No inventar.** Si la búsqueda no encuentra info sobre el entrevistador o la empresa, declararlo explícitamente. Cero hallucination.
- **Profundidad mínima.** Investigación del entrevistador requiere al menos 3 fuentes públicas distintas si existen. LinkedIn solo no alcanza.
- **Foco en impacto.** Toda respuesta STAR drafted abre con impacto/resultado, no con contexto cronológico.
- **Sparring real.** Si el modo sparring no encuentra weak spots o solo identifica gaps menores, declararlo. No suavizar para hacer sentir bien.
- **Calibración explícita.** Cada output relevante declara qué del perfil del usuario usó. Transparencia sobre supuestos.
- **Idioma del usuario.** Output en el idioma del archivo de perfil. Si la entrevista es en otro idioma, ajustar respuestas drafted a ese idioma pero el meta-contenido (análisis, sparring, recomendaciones) queda en el idioma del perfil.
- **Filtro de mercado.** Todo el análisis de empresa debe estar filtrado al mercado/región/unidad relevante de la entrevista. Información de otros mercados solo si es contexto estrictamente necesario.
- **IA en el análisis.** El DD de empresa debe incluir explícitamente cómo IA está afectando a la empresa y su industria. Sin este análisis el DD está incompleto.

---

## Notas para mantenimiento

- Cuando se ejecute en uso real, registrar qué falló o sobró para iterar la skill.
- Frameworks individuales en `frameworks/` se pueden iterar sin tocar este archivo.
- El framework de onboarding (`00-onboarding.md`) define cómo se levanta el perfil del usuario en la primera sesión. Cambios en estructura del perfil deben coordinarse entre el onboarding y el `template.md` en `user-context/`.

---
> Source: [martinjbellocq/full-interview-prep](https://github.com/martinjbellocq/full-interview-prep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
