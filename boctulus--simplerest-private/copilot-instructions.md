## simplerest-private

> - **Bootstrap 5** (generalmente utilizado)

# Proyecto sobre framework SimpleRest

## Stack Tecnológico

### Backend
- PHP

### Frontend
- **Bootstrap 5** (generalmente utilizado)
- **jQuery** (generalmente utilizado)

### Bases de Datos
- **MySQL** (soporta otras) 

### Testing & Automation
- **PHPUnit** (unit testing)
- **Playwright 1.54+** (disponible desde NodeJS)
- **Puppeteer** (alternativa para Chrome)
- **Selenium** (disponible desde Python)

--

## Antes de Empezar

### ⚠️ Paso Obligatorio
**SIEMPRE** verifica si existe la carpeta `docs/` antes de iniciar cualquier tarea:

```bash
ls docs/
# o
dir docs
```

Esta carpeta contiene o podria contentener documentación crítica del proyecto.

Ademas leer el archivo `README.md` a nivel de root del proyecto. 

Debes leer la documentacion pertinente interna al module / package / component que vayas a utilizar que generalmente son archivos .md internos.


### Archivos Clave en `docs/`

| Archivo | Descripción 
|---------|-------------
| `login-credentials.md` | **Usuarios y passwords** para testing con diferentes roles  (vacio de momento)
| `docs/issues/` 		 | Problemas comunes y soluciones (puede requerir actualización) 

---

## Sistema de Comandos CLI

Crear nuevos comandos personalizados para realizar tareas de caracter repetitivo y de caracter general.

Ver `docs/CommandLine.md`

**💡 Tip:** Siempre revisa los comandos disponibles antes de empezar una tarea, pueden ahorrarte mucho trabajo.

---

## Testing y Debugging

### Scripting de automatizacion de tareas

Scripts creados para mover archivos, renombrar carpetas, modificar archivos, etc deben alojarse en `scripts/` y si son de caracter transitorio dentro de `scripts/tmp/`

Los archivos .js dentro de `scripts/tmp/` deben borrarse al terminar las tareas y contar esta con validacion del usuario.

Estos scripts no son para debugging ni realizan Web Scraping con la ayuda de herramientas como Playwright que utilicen web drivers.

Ningun archivo temporal a ser generado (Ej: `debug_*.png`, `debug_*.txt`, `debug_*.log`, etc) debe hacerse en el root del proyecto.


### Donde alojar los scripts

Los scripts para debugging y testing deben ir segun corresponda en `tests/` o en `tests/unit-tests/` segun sean tests normales o del tipo prueba unitaria.

Los scripts que hacen uso de Web Drivers como Playwright deben ir dentro de `web-automation/`

Debe mantenerse organizacion y bajo ninguna circunstancia tener archivos del tipo `test_*.j`s o `test-*.js` en el root del proyecto.

Al iniciar revisa si encuentras archivos del tipo `test_*.js` o `test-*.js` en el root del proyecto y si asi muevelos a donde corresponda antes de proseguir.

Si el script consideras ya no tiene utilidad lo puedes borrar sin problemas.

Si necesitas generar un reportes sobre "pruebas unitarias" crea los reportes en `reports/unit-tests/` pero nunca en el root del proyecto.


### Testing E2E y Debugging de UI

Las credenciales para diferentes roles se deberian ubicar en:
```
docs/login-credentials.md
```

Nota:

Ningun archivo temporal a ser generado incluyendo screenshots debe hacerse en el root del proyecto. Coloca las imagenes dentro de `automation/screenshoots/` o en carpetas dentro de `automation/screenshoots`

Si necesitas generar un reporte con Web Scraping crea los reportes en `reports/automation/` pero nunca en el root del proyecto.

---

## Workflow de Trabajo

### 1. Investigación Inicial
```bash
# Listar documentación disponible
ls docs/

# Revisar comandos CLI disponibles
php com help
```

### 2. Desarrollo

- Enfocarse en una arquitectura modular.
- Consulta la documentación específica del módulo / componente / paquete dentro del modulo / componente / paquete suele haber un archivo .md y tambien verifica dentro de `docs\`
- Antes de emprender el desarrollo de una libreria compleja que por su caracter general podria existir ya desarrollada (ya sea como libreria, pacakge, etc) debes hacer una busqueda para asegurarte no exista ya una solucion "open source" que se pueda utilizar. En caso de existir pero tener una API distinta a la solicitada evaluar construir un adapter o wrapper sobre la misma. 

#### Routing

Es recomendable busques la documentacion de "routing" (sera algun archivo con extension .md) del framework.

No es necesario que ocupes el routes.php principal para el caso de packages/ Cada package puede tener su propio routes.php si hay un ServiceProvider. Revisa la documentacion y otros packages del framework. Podrias estar duplicando codigo de rutas.

Si las rutas no funcionan en un package es posible el package no este correctamente registrado. Importante ejecutar `composer dumpautoload` luego de cualquier registro / actualizacion en el "composer.json" para regenerar el autoload.


#### Consumo de API

Cuando debas consumir APIs utiliza la clase ApiClient en vez usar funciones CURL directamente.

Documentacion:
```
docs/ApiClient.md
```


## Prácticas Obligatorias

-   Tener en cuenta los patrones existentes y seguirlos con la salvedad de que si fuera necesario cambiar o implementar uno nuevo debe exponerse el problema, justificar el nuevo patron y consultar antes de implementar.  
-   Cambios incrementales. Analizar si fuera necesario un refactoring importante o re-escribir una solucion / libreria / vista de forma significativa, justificar y consultar antes de implementar.
-   DRY → funciones generales en librerías
-   Evitar logica compleja en rutas del router. Usar Middlwares.
-   SOLID / Clean Code / separación de responsabilidades
-   KISS
-   No agregar fallbacks sin consultar antes de implementarlos.
-   Lógica desacoplada de persistencia/API
-   Crear librerías → testear → persistencia/API
-   Tablas / colleciones de prueba con prefijo `test_`
-   Revisar documentación y comandos antes de programar
-   Testear de forma rigurosa.
-   Tests unitarios deben ser significativos y lo mas generales posibles
-   Documentar APIs

Leer importante adjunto `docs/core-directives.md`

### Complemento: Core Directives

Para principios de desarrollo de alto nivel y modos de trabajo adaptativos, consultar `docs/core-directives.md`, el cual define:

- **Multi-dimensional Analysis Framework**: Metodología estructurada para abordar problemas complejos considerando aspectos técnicos, arquitectónicos, de rendimiento e integración
- **Modos Adaptativos de Desarrollo**:
  - **Exploration Mode**: Para requisitos poco claros o complejos
  - **Implementation Mode**: Para tareas claramente definidas
  - **Debugging Mode**: Para resolución de problemas sistemática
  - **Optimization Mode**: Para mejoras de rendimiento
- **Solution-First Approach**: Enfoque que prioriza resolver el problema de negocio antes de los detalles de implementación
- **Estrategias de Batching y Ejecución Paralela**: Optimización de operaciones agrupadas por dependencias y recursos
- **Protocolos de Testing y Validación**: Checklists para pre/post deployment y validación manual

Este documento complementa las directivas operacionales de CLAUDE.md con frameworks mentales y principios filosóficos de desarrollo.


### 3. Testing

- **Unit tests:** Jest para lógica de negocio. Folder: tests/
- **E2E tests:** Playwright o Selenium para flujos completos
- Usar / actualiza las credenciales de `docs/login-credentials.md`

Importante:

La URL base es la que se define en el archivo `.env` bajo la clave APP_URL

Ej:
```
APP_URL=http://simplerest.lan
```

#### 3.1 Unit tests

Leer `docs/unit-tests-pruebas-unitarias.md`


### 4. Generar / actualizar documentación

Al finalizar cualquier tarea, **SIEMPRE preguntar**:
> ¿Puedo actualizar o crear la documentación correspondiente?

**Asegúrate de que quede documentado:**

- Nuevos comandos → `docs/commands/`
- Nuevos componentes → `docs/components/`  (refieren a componentes en `public/components/`)
- Nuevos modulos → `docs/modules/`
- Nuevos paquetes → `docs/packages/`
- Problemas resueltos → `docs/issues/`
- Cambios mayores → `docs/CHANGELOG-*.md`

---

## Notas Importantes

### Ambiente

Mi sistema corre Windows 11 pero tiene WSL/2 y puedes ejecutar comandos Linux bajo WSL.

Disponibles herramientas GNU para Windows como grep, fgrep, egrep, sed

Docker esta disponible. Directorio de trabajo -archivos de docker compose- es  
D:\Docker o D:\Pabloo\Docker


### Seguridad

NO BORRES TABLAS O REGISTROS O DOCUMENTOS EN LA FUENTE DE VERDAD (MySQL o cualquier otra) SIN JUSTIFICACION. Solo puedes borrar datos de prueba pero debes evitar borrar "tablas maestro" (o colecciones maestro). De ser necesario el borrado pide autorizacion.


## Obligatorio

SIEMPRE se debe probar antes de considerar que una tarea fue completada. 

Si ya se ha fallado mas de dos (2) veces en una tarea que involucra una UI, deberia ensayarse de ser posible con CURL y de no ser posible con Playwright o Selenium.

## Diferencias clave

- Ten mucho cuidado porque este framework utiliza funciones de la clase String para contains() etc con parametros en orden distinto que las funciones nativas de PHP. 

- Hay algunas diferencias sutiles entre los comandos del Query Builder de SimpleRest y Laravel, en particular DB::table() requiere archivo schema para el modelo (se genera con un comando) mientras que el helper table() no lo necesita.


## Autoría
**El author es SIEMPRE:**
```
Pablo Bozzolo (boctulus)
Software Architect
```

---

## Enlaces Rápidos

- 📝 Credenciales: `docs/login-credentials.md`
- 🧩 Componentes: `docs/packages/`
- ⚠️ Issues conocidos: `docs/issues/`
- 🧪 Testing UI: Usar Playwright o Selenium (ver sección [Testing y Debugging](#testing-y-debugging))

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boctulus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
