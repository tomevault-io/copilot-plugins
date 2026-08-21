## simulador-fluidos

> Este documento proporciona instrucciones a los agentes de inteligencia artificial que colaboren en el desarrollo de FluidLab.

# AGENTS.md

# FluidLab – Instrucciones para Agentes IA

## Objetivo

Este documento proporciona instrucciones a los agentes de inteligencia artificial que colaboren en el desarrollo de FluidLab.

Debe utilizarse como referencia antes de generar, modificar o refactorizar código.

---

# Descripción del proyecto

FluidLab es una plataforma web educativa desarrollada con Python y Django cuyo propósito es apoyar la enseñanza de Mecánica de Fluidos mediante simulaciones hidráulicas.

La versión inicial está orientada al flujo interno en tuberías circulares.

Los usuarios podrán:

- iniciar sesión;
- crear simulaciones;
- ejecutar cálculos;
- almacenar resultados;
- consultar el historial de simulaciones.

---

# Documentación disponible

Antes de realizar modificaciones, el agente debe revisar:

```
PROJECT.md
docs/ARCHITECTURE.md
docs/DOMAIN_MODEL.md
README.md
```

Estos documentos contienen la definición funcional del proyecto.

---

# Tecnologías

El proyecto utiliza exclusivamente:

- Python
- Django
- PostgreSQL
- ORM de Django
- HTML5
- CSS3
- JavaScript
- Git
- GitHub
- VS Code
- WSL Ubuntu

No deben proponerse tecnologías diferentes salvo solicitud explícita.

---

# Arquitectura

La arquitectura sigue el patrón MVT de Django.

La lógica del proyecto debe mantenerse desacoplada.

La responsabilidad de cada archivo es:

| Archivo | Responsabilidad |
|----------|-----------------|
| models.py | Persistencia |
| forms.py | Validación |
| views.py | Coordinación HTTP |
| services.py | Lógica de aplicación |
| calculations.py | Fórmulas hidráulicas |
| templates | Presentación |

---

# Filosofía del proyecto

FluidLab prioriza:

- claridad;
- simplicidad;
- mantenibilidad;
- trazabilidad;
- reutilización;
- documentación.

Se debe evitar:

- sobreingeniería;
- dependencias innecesarias;
- código duplicado;
- lógica matemática en las vistas;
- consultas SQL manuales.

---

# Desarrollo esperado

Cada nueva funcionalidad debe:

1. respetar la arquitectura existente;
2. reutilizar componentes;
3. mantener separación de responsabilidades;
4. incorporar validaciones;
5. mantener compatibilidad con PostgreSQL;
6. utilizar únicamente el ORM de Django.

---

# Motor de cálculo

Toda fórmula matemática debe implementarse en:

```
simulations/calculations.py
```

Las funciones deben ser:

- independientes de Django;
- reutilizables;
- testeables;
- documentadas;
- tipadas.

Nunca implementar fórmulas extensas en:

- views.py
- templates
- models.py

---

# Base de datos

La persistencia se realiza mediante PostgreSQL utilizando el ORM de Django.

No utilizar SQL directo salvo necesidad técnica claramente documentada.

---

# Modelos

El modelo de dominio se encuentra definido en:

```
docs/DOMAIN_MODEL.md
```

No crear nuevos modelos sin revisar dicho documento.

---

# Convenciones

Utilizar:

- PEP 8
- nombres descriptivos
- funciones pequeñas
- clases cohesivas
- comentarios únicamente cuando aporten claridad

---

# Flujo de trabajo

Antes de modificar código:

1. comprender el requerimiento;
2. revisar la documentación;
3. identificar los archivos afectados;
4. proponer una solución;
5. implementar cambios mínimos;
6. explicar qué se modificó.

---

# Restricciones

El agente no debe:

- cambiar la arquitectura del proyecto;
- eliminar archivos sin autorización;
- modificar migraciones existentes;
- cambiar PostgreSQL por otro motor;
- incorporar frameworks frontend adicionales;
- introducir dependencias innecesarias.

---

# Calidad del código

Todo código generado debe ser:

- legible;
- mantenible;
- modular;
- reutilizable;
- fácilmente comprobable mediante pruebas.

---

# Testing

Siempre que sea posible:

- crear pruebas unitarias;
- validar casos normales;
- validar casos extremos;
- evitar regresiones.

---

# Git

Trabajar mediante cambios pequeños.

Cada funcionalidad importante debe finalizar con un commit independiente.

---

# Objetivo final

El propósito del agente no es solamente escribir código.

Debe colaborar para construir una plataforma educativa robusta, mantenible y preparada para evolucionar hacia nuevas simulaciones relacionadas con Mecánica de Fluidos.

# Perfil del desarrollador

El desarrollador principal posee conocimientos de ingeniería mecánica y está utilizando FluidLab como plataforma para construir una aplicación mantenible y escalable.

El agente debe priorizar explicaciones claras y justificar las decisiones de arquitectura cuando existan varias alternativas razonables.

Siempre que sea posible, explicar brevemente el motivo técnico de las decisiones propuestas para facilitar el aprendizaje y el mantenimiento futuro del proyecto.

## Idioma del proyecto

- Los nombres de clases, funciones, variables, campos ORM y archivos deben escribirse en inglés.
- La interfaz destinada al usuario debe mostrarse en español.
- Los modelos deben usar `verbose_name` y `verbose_name_plural`.
- Los formularios, mensajes, plantillas y etiquetas visibles deben escribirse en español.
- La configuración regional principal será `es-cl`.

---
> Source: [psv1977/simulador-fluidos](https://github.com/psv1977/simulador-fluidos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
