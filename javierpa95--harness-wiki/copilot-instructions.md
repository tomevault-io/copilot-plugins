## harness-wiki

> **Normas comunes a todos los agentes que trabajan aquí.** Canónico. Si algún otro fichero de

# AGENTS.md — contrato para agentes en la raíz

**Normas comunes a todos los agentes que trabajan aquí.** Canónico. Si algún otro fichero de
orientación lo contradice, **gana este**.

> Esto es el **contrato**: cómo comportarse. El **catálogo** (qué hay y dónde) está en
> [`index.md`](index.md), que es la otra mitad y no se repite aquí.

## 1. Qué fichero manda

| Fichero | Rol | Dónde |
|---------|-----|-------|
| `AGENTS.md` | **Contrato** — cómo comportarse | raíz + los vaults donde las reglas *cambian* |
| `index.md` | **Catálogo** — qué contiene esta carpeta | todos los vaults, y subcarpetas si ayuda |
| `memory.md` | **Decisiones transversales** — append-only, una línea por evento | raíz |
| `CAPTURA.md` | **Bandeja de entrada** — todo lo suelto del humano va aquí primero; el agente lo enruta después (*"ordena la captura"*) | raíz |
| `README.md` | Portada para humanos (GitHub). No es contrato | raíz |

**Gana el `AGENTS.md` más profundo.** El de un vault sobrescribe al de la raíz; el de una
subcarpeta, al del vault. Es el fichero *más cercano al trabajo* el que sabe cómo se trabaja ahí.

**No pongas un `AGENTS.md` en cada carpeta.** El único criterio para crear uno:

> ### ¿Un agente trabajando aquí debe comportarse **distinto** que en la carpeta padre?

Si la respuesta es no, esa carpeta no necesita contrato — como mucho un `index.md`. Un contrato
desactualizado no es neutro: **miente con autoridad, y encima gana por profundidad.**

## 2. Al empezar

1. Lee [`index.md`](index.md) de la raíz (catálogo y Regla de oro).
2. Lee el `index.md` de la carpeta donde vas a trabajar (qué hay).
3. Lee el `AGENTS.md` más profundo que aplique (cómo se trabaja).

## 3. Commits

El agente que hace el trabajo es el que lo commitea. Si no lo haces tú, no lo hace nadie.

### Formato

```
tipo(ámbito): qué se hizo, en imperativo y en minúscula
```

| Tipo | Cuándo |
|------|--------|
| `feat` | nota nueva |
| `docs` | ampliar o editar una nota existente |
| `fix` | corregir algo que estaba mal |
| `refactor` | reorganizar: mover, renombrar, dividir |
| `chore` | mantenimiento: `.gitignore`, índices, permisos |

El **ámbito** es la parte del vault que tocaste. Es lo que más valor aporta al leer el
historial de un vistazo.

```
feat(ejemplo): añadir nota sobre modelos mixtos
docs(ejemplo): ampliar el apartado de fuentes
fix(harness): corregir el permiso de edición del subagente
chore(harness): actualizar el .gitignore
```

### Reglas

- **Commit atómico.** Un commit = un cambio con sentido propio. Si lo que hiciste necesita
  una "y" en el mensaje, probablemente son dos commits.
- **Commitea antes de cerrar la sesión.** No acumules trabajo sin guardar.
- **Nunca dejes cambios sin commitear al terminar.**

## 4. Innegociables

- **Idioma: español.** En notas y en conversación (cambiable en tu fork; sé consistente).
- **Nunca un secreto en un fichero de notas.** Punteros a dónde está la credencial, jamás el
  valor. El conocimiento se comparte y se versiona; las claves no.
- **El conocimiento vive en su vault, no en la raíz.** La raíz enruta.
- **Agentes no escriben ficheros vía shell** (redirecciones `>`, `Set-Content`, `Out-File`,
  `WriteAllText`): un fallo a medias corrompe el fichero. Para contenido, SIEMPRE las
  herramientas de edición del harness; bash lee y ejecuta, no escribe.
- **Los `.venv` nunca dentro del vault**, y los notebooks se guardan **sin salidas**.

## 5. Agentes (`.opencode/agents/`)

**Un orquestador primario (`obsidian`) + un subagente por vault.**

El prompt de un `vault-x` **apunta** al `AGENTS.md` del vault ("léelo y obedece"), **nunca lo
copia**, y es deliberadamente fino (~20 líneas: arranque + mandatos duros de permiso). Duplicar
un contrato está prohibido: es fabricar una segunda verdad que diverge.

- **OpenCode no lanza agentes "desde dentro" de un vault**: los agentes se descubren solo del
  config de la raíz y del global. Por eso la identidad vive en el vault (su contrato, que el
  subagente lee) y el enrutamiento en la raíz.
- **Cada `vault-x` tiene `edit` acotado a su carpeta** (permiso duro, no convención) y
  `task: deny` — un subagente no re-orquesta. Lo transversal (`memory.md`) lo escribe solo
  `obsidian`.
- Skills del harness en `.opencode/skills/` de la raíz. Guía para tocar config: skill
  `harness-config`.

| Agente | Modo | Qué hace | Cuándo usarlo |
|--------|------|----------|---------------|
| `obsidian` | primary | Orquestador: enruta según la Regla de oro, delega en los `vault-*`, escribe lo transversal (`memory.md`), teje relaciones y explica el sistema a quien empieza (uso; la config técnica va a `opencode-expert`) | Por defecto (Tab) |
| `opencode-expert` | primary | Experto en OpenCode y en este vault: mantiene la config, crea skills, y explica el sistema a quien empieza | Dudas de config o "¿cómo funciona esto?" |
| `vault-*` | subagent | Uno por vault: lee el `AGENTS.md` del vault y obedece; solo escribe en su carpeta | Delegado por `obsidian`, o directo con `@vault-x` |
| `vault-crear` | subagent | Ritual completo de alta de un vault nuevo | Al pedir un vault nuevo |
| `harness-auditor` | subagent | Compara `index.md` y contratos contra la realidad de las carpetas (solo lee, reporta) | Tras crear/renombrar carpetas, o al sospechar que la doc miente |

Dónde escribe cada cosa exactamente: [`index.md`](index.md) → *Regla de oro*.

## 6. Memoria del sistema

- `memory.md` (raíz) — decisiones **transversales** y acuerdos de cómo trabajamos. Append-only,
  una línea por evento, con fecha. Lo escribe solo el orquestador.
- Cada vault puede tener su propio registro (`log.md`) y su contrato si sus reglas cambian.

## 7. Añadir un vault nuevo

No lo improvises: existe la skill **`create-vault`** y el subagente `vault-crear`, que ejecutan
el ritual completo (estructura, contrato, alta en la raíz, subagente, registros). Úsalo.

## 8. YAML de notas: `config.json` y scripts

- Los **VALORES cerrados** del frontmatter (tipos, estados, campos por tipo) viven en
  [`config.json`](config.json) (raíz), legible por máquina. Los agentes lo LEEN para escribir
  YAML; los scripts de [`.opencode/scripts/`](.opencode/scripts/) lo validan de forma
  determinista (`check_schema.py`). Añadir o cambiar un valor = editar `config.json` y
  documentar en el contrato en la MISMA sesión.
- Con humanos sin filosofía AI: skill **`harness-proactive`** — el humano habla en lenguaje
  natural, el agente ejecuta de principio a fin (YAML correcto, índices, validador, commit) y
  responde sin jerga.

---
> Source: [javierpa95/harness-wiki](https://github.com/javierpa95/harness-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
