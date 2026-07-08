## mi-transistor

> **Mi Transistor** es una app de radio online diseñada específicamente para personas mayores, con una interfaz extremadamente simple, sin publicidad y centrada en emisoras españolas.

# 📻 Mi Transistor — agents.md

## 1. Propósito del Proyecto

**Mi Transistor** es una app de radio online diseñada específicamente para personas mayores, con una interfaz extremadamente simple, sin publicidad y centrada en emisoras españolas.

### Objetivo principal

Permitir que cualquier persona mayor pueda:

- Abrir la app
- Encontrar su emisora favorita
- Escucharla con **un solo toque**

### Principios clave

- Cero fricción
- Cero publicidad
- Cero configuración

---

## 2. Filosofía de Producto

### Propuesta de valor

- Sin anuncios (core del producto)
- Interfaz grande, clara y accesible
- Sin cuentas ni registro
- Favoritas visibles desde el inicio
- Audio en segundo plano

### Usuario objetivo

- Usuario final: personas mayores (60–80 años)
- Usuario comprador: hijo/a que instala la app

---

## 3. Reglas Globales para Agentes

Cualquier agente que modifique o genere código debe cumplir:

### UX

- Priorizar **simplicidad extrema** sobre funcionalidad
- Evitar cualquier interacción compleja
- Botones grandes (mínimo 48px–56px)
- Texto legible (mínimo 16–18px)
- Máximo 1 acción por pantalla cuando sea posible

### Funcionalidad

- Todo debe funcionar sin login
- Todo debe funcionar offline parcialmente (favoritos guardados)
- Minimizar errores visibles al usuario
- Evitar estados vacíos confusos

### Producto

- No introducir anuncios
- No introducir paywalls
- No introducir funcionalidades sociales
- No introducir complejidad innecesaria

---

## 4. Arquitectura del Proyecto

### Stack

- Vue 3
- Pinia (state management)
- localStorage (persistencia)
- HTML5 Audio API

---

### Estructura de carpetas

# 📻 Mi Transistor — agents.md

## 1. Propósito del Proyecto

**Mi Transistor** es una app de radio online diseñada específicamente para personas mayores, con una interfaz extremadamente simple, sin publicidad y centrada en emisoras españolas.

### Objetivo principal

Permitir que cualquier persona mayor pueda:

- Abrir la app
- Encontrar su emisora favorita
- Escucharla con **un solo toque**

### Principios clave

- Cero fricción
- Cero publicidad
- Cero configuración

---

## 2. Filosofía de Producto

### Propuesta de valor

- Sin anuncios (core del producto)
- Interfaz grande, clara y accesible
- Sin cuentas ni registro
- Favoritas visibles desde el inicio
- Audio en segundo plano

### Usuario objetivo

- Usuario final: personas mayores (60–80 años)
- Usuario comprador: hijo/a que instala la app

---

## 3. Reglas Globales para Agentes

Cualquier agente que modifique o genere código debe cumplir:

### UX

- Priorizar **simplicidad extrema** sobre funcionalidad
- Evitar cualquier interacción compleja
- Botones grandes (mínimo 48px–56px)
- Texto legible (mínimo 16–18px)
- Máximo 1 acción por pantalla cuando sea posible

### Funcionalidad

- Todo debe funcionar sin login
- Todo debe funcionar offline parcialmente (favoritos guardados)
- Minimizar errores visibles al usuario
- Evitar estados vacíos confusos

### Producto

- No introducir anuncios
- No introducir paywalls
- No introducir funcionalidades sociales
- No introducir complejidad innecesaria

---

## 4. Arquitectura del Proyecto

### Stack

- Vue 3
- Pinia (state management)
- localStorage (persistencia)
- HTML5 Audio API

---

## 5. Estructura de carpetas

```
src/
├── pages/
│   ├── HomePage.vue
│   └── FavoritesPage.vue
├── components/
│   └── PlayerBar.vue
├── stores/
│   ├── radioStore.js
│   └── playerStore.js
```

---

## 6. Principio Rector

> Si una persona mayor no puede usarlo sin ayuda, está mal diseñado.

---

## 7. Nota Final

Este documento es la fuente de verdad del proyecto.

---
> Source: [cjlozanoDev/mi-transistor](https://github.com/cjlozanoDev/mi-transistor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
