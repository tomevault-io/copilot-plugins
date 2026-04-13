## budgetpro

> 1. **CONSULTA OBLIGATORIA DEL HANDBOOK**: Antes de cualquier acción, el asistente DEBE leer `.budgetpro/handbook/AXIOM_SAFE_OPERATIONS.md`. Es el oráculo de decisiones para MODES, RISK y NAMING.

# BudgetPro Governing Rules for AI and Development

#

# This file defines the mandatory guidelines that govern how the codebase

# must be modified and how AI assistants should operate when interacting

# with the repository. It complements the AXIOM configuration and reflects

# recent hardening of the Naming, Boundary, State Machine and Semgrep

# validators. All changes and suggestions must comply with the rules

# defined here.

## ⚖️ LEY SUPREMA DE GOBERNANZA (AXIOM-SAFE)

1. **CONSULTA OBLIGATORIA DEL HANDBOOK**: Antes de cualquier acción, el asistente DEBE leer `.budgetpro/handbook/AXIOM_SAFE_OPERATIONS.md`. Es el oráculo de decisiones para MODES, RISK y NAMING.

2. **DIAGNÓSTICO DE APERTURA (MANDATORIO)**:
   - No se permite código sin antes imprimir en el chat el **DIAGNÓSTICO INICIAL**:
     ```
     📊 [MODO: 0/1/2] | [RIESGO: LOW/MID/HIGH] | [BRANCH: tipo/sistema-desc]
     Estado: [Resumen corto del repo según ./axiom.sh --status]
     ```

3. **JERARQUÍA DE ALERTAS**:
   - **[BLOCKING]**: Detención absoluta. Prohibido el uso de `--no-verify` a menos que se documente un incidente en `.budgetpro/handbook/INCIDENT_LOGS.md` bajo MODE_0.
   - **[WARNING]**: Los avisos de 'LEAK' o 'SECRET' se tratan como BLOCKING. Los de 'Null-Safety' deben corregirse en el batch actual.

4. **ESTÁNDAR DE TRABAJO ATÓMICO**:
   - **1 Propósito = 1 Commit**.
   - Si un cambio excede el **Blast Radius (10 archivos)**, el asistente DEBE realizar un `git reset` y dividir el trabajo en Batches Temáticos (ej: batch_logic, batch_tests).

5. **LIMPIEZA DE CÓDIGO (ANTI-DIRTY)**:
   - Prohibido eliminar `System.out` o `printStackTrace` sin reemplazarlos por una implementación formal de `Logger` (SLF4J/java.util.logging).
   - Todo método público nuevo o modificado DEBE incluir validación `Objects.requireNonNull()`.

6. **PROTECCIÓN DE ARCHIVOS DE GOBIERNO**:
   - El asistente tiene PROHIBIDO modificar `.cursorrules.md` o `axiom.config.yaml` sin autorización explícita.
   - Cualquier archivo llamado `.cursorrules` (sin extensión) es considerado CORRUPTO y debe ser purgado según SOP-02.

supreme_rule:

# AXIOM is the ultimate authority. The domain layer is sacrosanct and

# must remain free of dependencies on frameworks or upper layers.

- "AXIOM IS LAW: all code changes must pass `./axiom.sh --dry-run` before execution and respect the hexagonal architecture."
- "Domain layer is sacrosanct: ZERO framework dependencies (Spring, Hibernate, JPA, Jakarta, etc.)."
- "Domain can ONLY import: java.*, java.util.*, java.math.*, java.time.* (Java Standard Library only)."
- "Any import of org.springframework.*, jakarta.*, javax.*, or any framework package in com.budgetpro.domain is a BLOCKING violation."
- "Never modify the domain layer (`com.budgetpro.domain`) to fix errors in the application or infrastructure layers."
- "Canonical Notebooks are the Source of Truth: NOTEBOOK > CODE (per AI_AGENT_PROTOCOL.md, Rule 3)."

hands_clean:

# Protocol for handling large compilation failures (>50 errors).

- "Extract only the first 20 errors to identify the root cause before editing any files."
- "Prioritise correcting imports or calls in Application/Infrastructure; do not change Domain to satisfy upper-layer errors."
- "If the root cause truly resides in Domain, stop and create an `implementation_plan.md` for approval before proceeding."
- "After each individual change, run `./axiom.sh --dry-run` to validate before making the next change."

initialization:

- "Before writing any code, always run `./axiom.sh --dry-run` to validate the current state."
- "Verify that there are no pending AXIOM violations."
- "Consult `.budgetpro/axiom.config.yaml` and this file (`.cursorrules.md`) to understand the active rules."

architecture:

# Hexagonal architecture enforcement.

- "Domain ← Application ← Infrastructure: dependencies flow inward."
- "Domain depends on nothing; Application depends only on Domain; Infrastructure depends on both."
- "Use Naming, Boundary, State Machine and Semgrep validators to verify adherence to these principles."

blast_radius:

# Limits on the number of files modified per commit in sensitive areas.

red: - "domain/finanzas/presupuesto: max 1 file per commit" - "domain/finanzas/estimacion: max 1 file per commit" - "domain/valueobjects: max 1 file per commit" - "domain/entities: max 1 file per commit"
yellow: - "infrastructure/persistence: max 3 files per commit"
green: - "application: max 10 files per commit" - "infrastructure/web: max 10 files per commit" - "tests: max 10 files per commit"

ai_assistant:

# Mandatory checks for AI assistants when preparing changes.

- "Check that the blast radius limits are respected for the proposed change."
- "Confirm that no secrets or sensitive data are exposed."
- "Ensure no lazy code patterns remain (e.g., methods returning null without validation)."
- "Verify adherence to the hexagonal architecture and domain isolation."
- "Run Naming, Boundary, State Machine and Semgrep validators as part of the validation pipeline."
- "Classify each proposed change as LOW, MID or HIGH risk and ensure the appropriate operating mode is engaged."
- "Before implementing any feature, scan Canonical Notebooks for [AMBIGUITY_DETECTED] flags. If found, STOP immediately and request clarification from user."
- "Never proceed with implementation if the specification contains [AMBIGUITY_DETECTED] markers (per AI_AGENT_PROTOCOL.md, Rule 1)."
- "If a validation rule is missing in Canonical Notebooks, STOP and ASK user. Output format: 'I cannot find the validation rule for [Y] in Section 2 (Invariants). Please provide it.' (per AI_AGENT_PROTOCOL.md, Rule 2)."
- "Never implement business logic without explicit specification in Canonical Notebooks."

bypass_prohibition:

# Rules forbidding bypasses of governance and validation.

- "Never modify protected configuration files (`axiom.config.yaml`, `.cursorrules.md`, `boundary-rules.json`, `state-machine-rules.yml`, Semgrep rule files) to circumvent validations."
- "If a change is blocked by AXIOM or governance, split the work into smaller commits or request explicit approval from the project owner."
- "Do not use command-line flags such as `--no-verify` or alter blast radius limits to force a commit through."

modes:
MODE_0:
description: "Emergency lockdown: project does not compile or more than 50 compilation errors."
allowed: - "Fix broken imports, remove duplicate methods or comment invalid code solely to restore compilation."
forbidden: - "Adding features, refactoring or changing business logic."
MODE_1:
description: "Stabilization: build compiles but tests fail or there are many warnings."
allowed: - "Grouped changes addressing a single root cause without touching domain logic."
forbidden: - "Changing domain semantics or introducing new business decisions."
MODE_2:
description: "Normal operation: CI is green and there is no critical debt."
allowed: - "One cause per commit; changes must be atomic and compliant with all rules."

risk_classification:
low:
description: "Mechanical fixes such as removing duplicates, adjusting imports or visibility, or trivial code formatting."
behaviour: "The IDE may execute these automatically but must inform the user afterwards."
mid:
description: "Changes that adjust control flow, tweak service logic or reorganise internals without affecting domain."
behaviour: "The IDE must propose the plan and wait for explicit user approval."
high:
description: "Any change involving `com.budgetpro.domain`, enums, state machines, public contracts or business semantics."
behaviour: "The IDE must generate an `analysis_plan.md` and await explicit approval from the user before execution."

atomicity:

- "Each commit must pursue a single objective with a single root cause and be fully reversible."
- "Do not mix corrections with enhancements or touch the domain without justification."

big_bang:

- "Large changes modifying many files must pass through MODE_0 or MODE_1."
- "Never modify more files than allowed by the blast radius without declaring the appropriate operating mode and validating after each batch."

protected_files:

- "axiom.config.yaml"
- ".cursorrules.md"
- ".domain-validator.yaml"
- "boundary-rules.json"
- "state-machine-rules.yml"
- ".semgrep/rules/\*"
- "Naming validator configuration files (if present)"
- ".budgetpro/handbook/*.md"
- "docs/canonical/**/*.md"

updates:

- "This file must remain consistent with the governance document and tool configurations."
- "Any updates require approval from project governance and must be accompanied by documentation explaining the changes."

## 🔍 PROTOCOLO DE LECTURA DE TERMINAL (OBLIGATORIO)

1. **PROHIBIDO IGNORAR WARNINGS**: Aunque el commit sea exitoso, si AXIOM imprime un `[WARNING]`, el asistente DEBE leerlo. Si el warning contiene "LEAK", "SECRET", o "SECURITY", DEBES hacer un rollback del commit inmediatamente y corregirlo.
2. **ANÁLISIS DE SUGERENCIAS**: Ante un `[BLOCKING]`, es MANDATORIO leer la sección `Sugerencia:` que provee AXIOM. No intentes adivinar la solución; aplica la sugerencia técnica impresa.
3. **VERIFICACIÓN DE JAR**: Si el terminal indica `Domain validator JAR not found`, no intentes validar el dominio hasta ejecutar `cd tools/domain-validator && ./mvnw package`.

## 🛑 PROTOCOLO DE APERTURA DE JORNADA (MANDATORIO)

Antes de proponer o ejecutar CUALQUIER cambio, el asistente DEBE realizar un diagnóstico y declarar su estado en el chat con el siguiente formato:

---

📊 **DIAGNÓSTICO INICIAL AXIOM**

- **ESTADO DEL REPOSITORIO**: (Ej: 54 archivos en stage, errores de compilación detectados)
- **MODO OPERATIVO SELECCIONADO**: [MODE_0 | MODE_1 | MODE_2] (Justificar según ./axiom.sh --status)
- **CLASIFICACIÓN DE RIESGO**: [LOW | MID | HIGH] (Justificar según la zona de impacto)
- **PLAN DE ACCIÓN**: (Breve descripción de pasos)

---

**REGLA DE BLOQUEO**: Si el asistente no presenta este diagnóstico, el USUARIO tiene prohibido aceptar cualquier sugerencia.

## 📚 PROTOCOLO DE NOTEBOOKS CANÓNICOS (MANDATORIO)

Este protocolo asegura que el AI siempre use las especificaciones canónicas como fuente de verdad.

### 1. PRIORIDAD DE CARGA (Context Selection Strategy)

**Priority 1: Critical (Always Load)**
- Module Notebook: `docs/canonical/modules/[MODULE]_MODULE_CANONICAL.md`
- Architecture: `docs/canonical/radiography/ARCHITECTURAL_CONTRACTS_CURRENT.md`

**Priority 2: Important (Task Dependent)**
- Data Modeling: `docs/canonical/radiography/DATA_MODEL_CURRENT.md`
- API Design: `docs/canonical/radiography/INTEGRATION_PATTERNS_CURRENT.md`
- Business Logic: `docs/canonical/radiography/DOMAIN_INVARIANTS_CURRENT.md`

**Regla**: Antes de implementar cualquier feature, el asistente DEBE cargar el Module Notebook correspondiente y ARCHITECTURAL_CONTRACTS_CURRENT.md.

### 2. DETECCIÓN DE AMBIGÜEDAD (Ambiguity Detection)

- **Si encuentras `[AMBIGUITY_DETECTED]` en notebooks → STOP inmediatamente**
- **Output obligatorio**: "The specification for [X] is flagged as ambiguous in [Notebook]. Please clarify."
- **Nunca proceder con implementación** si la especificación contiene marcadores de ambigüedad.
- **Referencia**: AI_AGENT_PROTOCOL.md, Section 2, Rule 1; AMBIGUITY_PROTOCOL.md

### 3. AUTORIDAD NOTEBOOK > CÓDIGO (Notebook Authority)

- **Si código existente contradice notebooks → Seguir notebooks**
- **Output obligatorio**: "The code does X, but the Notebook specifies Y. I will proceed with Y unless instructed otherwise."
- **Nunca modificar Canonical Notebooks** para "hacer que coincida con el código" sin aprobación explícita.
- **Referencia**: AI_AGENT_PROTOCOL.md, Section 2, Rule 3

### 4. ESPECIFICACIONES FALTANTES (Missing Specifications)

- **Si falta regla en notebooks → ASK, no inventar**
- **Output obligatorio**: "I cannot find the validation rule for [Y] in Section 2 (Invariants). Please provide it."
- **Nunca implementar lógica de negocio** sin especificación explícita en Canonical Notebooks.
- **Referencia**: AI_AGENT_PROTOCOL.md, Section 2, Rule 2

### 5. PROTECCIÓN DE NOTEBOOKS (Read-Only)

- `docs/canonical/**/*.md` son **READ-ONLY** por defecto
- Solo actualizar con **aprobación explícita del usuario**
- Nunca modificar para "hacer que coincida con el código"
- Si se requiere actualización, crear PR separado documentando el cambio

### 6. CHECKLIST PRE-IMPLEMENTACIÓN (Pre-Implementation Checklist)

Antes de escribir código, el asistente DEBE:

1. **Cargar Module Notebook** relevante desde `docs/canonical/modules/`
2. **Escanear por `[AMBIGUITY_DETECTED]`** → STOP si se encuentra
3. **Verificar invariantes** documentados en Section 2
4. **Comparar con código existente** → Seguir Notebook si hay conflicto
5. **Confirmar archivos protegidos** no serán modificados
6. **Verificar blast radius** para archivos afectados
7. **Cargar ARCHITECTURAL_CONTRACTS_CURRENT.md** si hay cambios arquitectónicos

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carleetho) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
