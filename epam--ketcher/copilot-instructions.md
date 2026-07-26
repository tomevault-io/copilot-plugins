## ketcher

> **Guidelines for Using GitHub Copilot in the Ketcher Project**

**Guidelines for Using GitHub Copilot in the Ketcher Project**

This document defines how GitHub Copilot should behave when assisting contributors working on the **Ketcher** web application. Its purpose is to keep generated code consistent with Ketcher's architecture, coding standards, and domain-specific requirements.

---

## 🔍 Project Overview

**Ketcher** is an open-source **chemical structure editor** developed in **TypeScript**, using **React**, **Redux**, and a custom **MVC model** for rendering molecules, reactions, macromolecules, and monomers.
The project also includes **canvas rendering**, **stereo-chemistry**, **V2000/V3000 parsing**, **layout algorithms**, and **complex UI/UX logic** for drawing, editing, and analyzing molecular structures.

Copilot should generate solutions compatible with:

* **TypeScript (strict mode)**
* **React functional components**
* **Redux Toolkit**
* **Immutable state principles**
* **Ketcher's MVC architecture**

    * **Model**: chemical objects (atoms, bonds, monomers, coordinates)
    * **View**: rendering with SVG
    * **Controller**: user actions, tools, interaction logic

---

## 🧭 General Rules

### ✔ DO

* Prefer **TypeScript** over JavaScript.
* Use **strict typing**, including union types, mapped types, and generics.
* Follow existing code patterns found in the Ketcher source.
* Use **pure functions** for calculations and transformations.
* Use **React + Redux Toolkit** patterns for UI logic.
* Preserve the **immutability** of model objects unless modifying through allowed actions/reducers.
* Reference existing **Ketcher services, helpers, actions, and controllers** when generating code.
* Keep code consistent with domain constraints:

    * Coordinates are stored in **Angstrom-like units**.
    * Atoms and bonds follow **Ketcher's internal IDs**, not arbitrary numbers.
* Provide **clear, concise comments** describing algorithms or domain logic.

### ❌ DO NOT

* Generate code that mutates Redux state directly.
* Introduce libraries not used in the project (e.g., MobX, Lodash, RxJS, jQuery).
* Invent new API surfaces or file structures unless explicitly requested.
* Use React class components.
* Produce “magic numbers” without explaining them in comments.
* Simplify chemical logic (e.g., stereochemistry, valence validation, CFG parsing) unless appropriate.
* Violate existing architectural boundaries (e.g., the controller should not manipulate model internals directly).

---

## 🧩 Typical Tasks Copilot Should Assist With

### Model & Chemistry

* Generating utilities for atom/bond operations.
* Parsing or producing Molfile V2000/V3000 blocks.
* Implementing coordinate transformations.
* Handling stereochemistry or CFG tags.
* Improving monomer modeling and polymer chain logic.

### Controller & Interaction

* Adding new editor tools or actions.
* Improving selection, highlight, drag or transform behaviors.

### View & Rendering

* Calculating layout changes.
* Enhancing SVG rendering helpers.

### UI / React

* Dialogs, menus, context panels.
* Wizards (like RNA presets, monomer properties).
* Error handling, field validation.

---

## 🔐 Copilot Behavior Principles

### Copilot must:

* Prioritize **existing Ketcher patterns** over generic web development patterns.
* Generate **safe, readable, testable** code.
* Explain chemistry-related logic when unclear.

### Copilot should not:

* Autogenerate large refactors unless asked.
* Suggest major architecture changes without justification.
* Produce code that violates domain rules (e.g., introducing forbidden valence states).

---

## 🏁 Example Prompts for Contributors

### Good prompts for Ketcher:

* “Generate a helper to restore monomer coordinates when switching from snake mode to flex mode.”
* “Create a Redux slice for storing macromolecule editor settings.”
* “Implement a controller action for dragging a monomer along the polymer backbone.”
* “Add a typed utility for computing CFG stereochemistry from V3000 data.”

### Prompts to avoid:

* “Rewrite the entire rendering engine.”
* “Replace Redux with another state library.”
* “Add Lodash just to map objects.”

---
> Source: [epam/ketcher](https://github.com/epam/ketcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
