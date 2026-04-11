## wapp-socket

> Este archivo contiene prompts estáticos reutilizables para trabajar con Gemini Code en el proyecto TemplateGo. Cada prompt está adaptado para el ecosistema Go y sigue los estándares definidos en `Cases/Documentation Main/`.

# Gemini Prompts - TemplateGo

Este archivo contiene prompts estáticos reutilizables para trabajar con Gemini Code en el proyecto TemplateGo. Cada prompt está adaptado para el ecosistema Go y sigue los estándares definidos en `Cases/Documentation Main/`.

Comunicate siempre en español, incluso los logs que emites cuando vas realizando las tareas.


## Índice de Prompts

- [mvp-sprint](#mvp-sprint) - Ejecutar tareas de un MVP específico
- [code-review](#code-review) - Revisión de código para un MVP
- [build-test](#build-test) - Compilar y probar un módulo específico
- [architecture-analysis](#architecture-analysis) - Analizar arquitectura del proyecto
- [migration-guide](#migration-guide) - Crear guía de migración entre versiones

## Prompts Disponibles

### mvp-sprint
**Descripción**: Ejecuta las tareas definidas para un MVP específico siguiendo los estándares del proyecto.

**Uso**: `ejecuta el prompt mvp-sprint con MVP-06`

**Variables**:
- `MVP_NUMBER`: Número del MVP (ej: MVP-05, MVP-06)

**Prompt**:
```
En la carpeta Cases/Modulos/{MVP_NUMBER}/ están los documentos necesarios para trabajar este sprint. Incluye el detalle y todas las especificaciones. En la carpeta Cases/Documentation Main/ hay archivos .md que indican los estándares que debes seguir para cumplir este sprint.

Tu tarea principal es:
1. Revisar y entender los documentos en Cases/Modulos/{MVP_NUMBER}/
2. Revisar los estándares en Cases/Documentation Main/
3. Implementar todas las tareas especificadas en tareas.md
4. Al finalizar, completar el archivo Cases/Modulos/{MVP_NUMBER}/resultado.md

Sigue estos principios:
- Utiliza la arquitectura limpia definida en DEVELOPMENT_GUIDELINES.md
- Mantén compatibilidad multiplataforma (Linux, MacOS, Windows)
- Escribe pruebas unitarias con cobertura mínima del 60%
- Documenta todas las decisiones arquitectónicas

Cuando termines todas las tareas, actualiza el archivo resultado.md con:
- Resumen de funcionalidades implementadas
- Versiones utilizadas
- Estructura final del proyecto
- Decisiones y justificaciones
- Ejemplos de código relevantes
- Resultados de build y tests (`go build`, `go test -cover`)
- Notas para futuros sprints
```

### code-review
**Descripción**: Realiza una revisión completa del código implementado en un MVP.

**Uso**: `ejecuta el prompt code-review con MVP-04`

**Variables**:
- `MVP_NUMBER`: Número del MVP a revisar

**Prompt**:
```
Realiza una revisión completa del código implementado para {MVP_NUMBER}. 

Analiza los siguientes aspectos:
1. **Arquitectura**: Verificar que sigue Clean Architecture y principios SOLID
2. **Estándares**: Cumplimiento de DEVELOPMENT_GUIDELINES.md
3. **Testing**: Cobertura y calidad de pruebas (`go test -cover`)
4. **Performance**: Identificar posibles cuellos de botella
5. **Security**: Revisar exposición de datos sensibles
6. **Maintainability**: Legibilidad y documentación del código

Genera un reporte con:
- ✅ Aspectos que cumplen con los estándares
- ⚠️ Mejoras sugeridas
- ❌ Problemas críticos que deben solucionarse
- 📝 Recomendaciones para futuros desarrollos

Enfócate en los archivos creados o modificados en el {MVP_NUMBER}.
```

### build-test
**Descripción**: Compila y prueba módulos específicos del proyecto.

**Uso**: `ejecuta el prompt build-test con internal/app`

**Variables**:
- `MODULE_PATH`: Ruta del módulo (internal/app, pkg/logger, etc.)

**Prompt**:
```
Ejecuta una compilación y prueba completa del módulo {MODULE_PATH}.

Realiza las siguientes tareas en orden:
1. Compilar el módulo: go build {MODULE_PATH}
2. Ejecutar pruebas unitarias: go test {MODULE_PATH} -v -cover
3. Análisis estático: golangci-lint run {MODULE_PATH}
4. Verificar que no hay warnings críticos

Si hay errores:
- Analizar y corregir problemas de compilación
- Resolver fallos en tests
- Arreglar violaciones de lint

Proporciona un resumen final con:
- ✅ Estado de compilación
- 📊 Resultados de tests (número de tests ejecutados/pasados/fallidos)
- 🔍 Problemas de análisis estático encontrados y corregidos
- 📈 Métricas de cobertura si están disponibles
```

### architecture-analysis
**Descripción**: Analiza la arquitectura actual del proyecto y sugiere mejoras.

**Uso**: `ejecuta el prompt architecture-analysis`

**Prompt**:
```
Realiza un análisis completo de la arquitectura del proyecto TemplateGo.

Analiza los siguientes aspectos:
1. **Modularización**: Estructura de carpetas y dependencias
2. **Clean Architecture**: Separación de capas (domain, interfaces, infrastructure, pkg)
3. **Compatibilidad multiplataforma**: Linux, MacOS, Windows
4. **Dependency Injection**: Uso de interfaces y composición nativa de Go
5. **Database Layer**: Uso de GORM, sqlx o nativo
6. **Network Layer**: Uso de net/http, gin, o frameworks nativos
7. **State Management**: Uso de context y manejo de concurrencia
8. **Testing Strategy**: Estructura de tests y mocks con testify

Genera un reporte con:
📋 **Estado Actual**:
- Descripción de la arquitectura actual
- Diagrama de dependencias entre módulos

🎯 **Fortalezas Identificadas**:
- Aspectos bien implementados
- Patrones correctamente aplicados

⚠️ **Áreas de Mejora**:
- Problemas arquitectónicos
- Violaciones de principios SOLID
- Acoplamiento excesivo

🚀 **Recomendaciones**:
- Refactorizaciones sugeridas
- Nuevos patrones a implementar
- Plan de mejora por fases
```

### migration-guide
**Descripción**: Crea una guía de migración entre versiones de dependencias o Go.

**Uso**: `ejecuta el prompt migration-guide de Go-1.19 a Go-1.20`

**Variables**:
- `FROM_VERSION`: Versión origen
- `TO_VERSION`: Versión destino

**Prompt**:
```
Crea una guía de migración para actualizar de {FROM_VERSION} a {TO_VERSION} en el proyecto TemplateGo.

La guía debe incluir:

1. **Análisis de Compatibilidad**:
   - Cambios breaking
   - Nuevas features disponibles
   - Dependencias afectadas

2. **Pasos de Migración**:
   - Actualización de go.mod y go.sum
   - Cambios en Makefile y scripts
   - Modificaciones de código necesarias
   - Actualizaciones en configuración

3. **Testing Plan**:
   - Tests que ejecutar después de la migración (`go test ./...`)
   - Validaciones de compatibilidad multiplataforma
   - Verificaciones de build en todos los sistemas soportados

4. **Rollback Plan**:
   - Cómo revertir los cambios si hay problemas
   - Puntos de verificación durante la migración

5. **Documentación**:
   - Actualizar README y documentación técnica
   - Crear entrada en CHANGELOG.md

Ejecuta la migración paso a paso y documenta cualquier problema encontrado.
```

## Uso de los Prompts

Para usar cualquier prompt, simplemente escribe:
```
ejecuta el prompt [nombre-del-prompt] con [parámetros]
```

Ejemplos:
- `ejecuta el prompt mvp-sprint con MVP-06`
- `ejecuta el prompt code-review con MVP-05`
- `ejecuta el prompt build-test con internal/app`
- `ejecuta el prompt architecture-analysis`
- `ejecuta el prompt migration-guide de Go-1.19 a Go-1.20`

## Añadir Nuevos Prompts

Para añadir un nuevo prompt:

1. Crea una nueva sección en este archivo
2. Define claramente el propósito y uso del prompt
3. Especifica las variables necesarias
4. Escribe el prompt siguiendo las mejores prácticas
5. Añade un ejemplo de uso
6. Actualiza el índice

**Template para nuevos prompts**:
```markdown
### nombre-del-prompt
**Descripción**: [Descripción breve del prompt]

**Uso**: `ejecuta el prompt nombre-del-prompt con [parámetros]`

**Variables**:
- `VARIABLE_NAME`: Descripción de la variable

**Prompt**:
```
[Contenido del prompt aquí]
```

## Notas

- Los prompts están diseñados para ser eficientes y minimizar tokens
- Siguen las convenciones y estándares definidos en `Documentation Main/`
- Se actualizan según la evolución del proyecto
- Mantén la coherencia con la arquitectura limpia y multiplataforma de Go

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/medinatello)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/medinatello)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
