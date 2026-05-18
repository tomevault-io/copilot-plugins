## financial-planner

> Aplicación web para organizar finanzas personales desarrollada con Ruby on Rails 8

# Project Context

## Financial Planner

Aplicación web para organizar finanzas personales desarrollada con Ruby on Rails 8

## Características Principales

- **Gestión de Cuentas**: Crea y gestiona múltiples cuentas (corriente, ahorros, inversión, tarjeta de crédito, préstamos, etc.).
- **Balance General**: Registra activos líquidos, activos fijos, pasivos y calcula automáticamente el patrimonio neto. Almacena fecha y hora exacta.
- **Presupuestos**: Crea presupuestos con periodicidad (mensual por defecto), definiendo ingresos, gastos y cálculo automático del flujo de caja libre.
- **Reportes**: Genera reportes de balance general con filtros por cuenta y fechas.

## Tech Stack

- **Framework**: Ruby on Rails 8.1.1
- **Database**: PostgreSQL
- **Authentication**: Devise
- **View Components**: Cells (Trailblazer Cells pattern) para componentes reutilizables.
- **Forms**: SimpleForm (integración con Bootstrap).
- **Styling**: Bootstrap 5.
- **Interactivity**: Hotwire (Turbo + Stimulus).

## Domain Model

- **User**: Usuarios con autenticación Devise.
- **Account**: Cuentas asociadas a usuarios (checking, savings, investment, credit_card, loan, other).
- **BalanceSheet**: Balance general con fecha/hora de registro.
- **Asset**: Activos (líquidos y fijos) asociados a un balance.
- **Liability**: Pasivos (corto y largo plazo) asociados a un balance.
- **Budget**: Presupuestos con periodicidad configurable.
- **BudgetItem**: Items de presupuesto (ingresos y gastos).

## Coding Guidelines

- **Views**: Preferir el uso de **Cells** para lógica de vista compleja o componentes reutilizables.
- **Forms**: Usar `simple_form_for` con estilos de Bootstrap.
- **Front-end**: Utilizar Hotwire para actualizaciones dinámicas en lugar de JavaScript complejo manual siempre que sea posible.
- **Language**: El código (variables, métodos) suele estar en inglés, pero la UI y comentarios de contexto pueden estar en español según el README.

---
> Source: [SunTea43/Financial-Planner](https://github.com/SunTea43/Financial-Planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
