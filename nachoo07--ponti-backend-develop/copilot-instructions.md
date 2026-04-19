## ponti-backend-develop

> - NUNCA usar valores hardcodeados a menos que se autorice explícitamente

# Cursor Rules para Ponti Backend

## 🚨 REGLAS CRÍTICAS
- NUNCA usar valores hardcodeados a menos que se autorice explícitamente
- NUNCA usar ROUND() en migraciones SQL, mantener precisión completa
- NUNCA cambiar nada que no sea quirúrgicamente lo que se pide
- Siempre usar parámetros dinámicos y configurables
- Siempre comenzar con TLDR
- Siempre antes de implementar esperar que acepte para lo pretendido.

## Idioma
- Comentarios en español (//, /* */, --)
- Código en inglés
- TODOs en inglés

## Go
- Usar pointers para campos no-slice
- Slices como valores ([]Type)
- decimal.Decimal para campos monetarios
- 3 decimales de precisión en dashboard
- Usar shareddomain.Base al final de structs

## Arquitectura
- Adapters usan DTOs para comunicación externa
- Use cases solo trabajan con primitives o domain types
- Repository usa models del package models
- Migraciones separadas por entidad

## Base de Datos
- Priorizar SQL migrations sobre modelos Go
- No modificar migraciones existentes, crear nuevas
- Usar seeds-dashboard.sql para datos de prueba
- NUNCA usar ROUND() en migraciones SQL, mantener precisión completa
- Migraciones en inglés, comentarios en español

## Tests
- Table-driven tests
- gomock para mocking
- Ejecutar tests inmediatamente sin preguntar

## Docker
- Comandos se ejecutan dentro de contenedores
- Usar make targets para operaciones comunes
- Un proyecto a la vez, nunca más de uno

## Manejo de Errores
- Usar tipos de error consistentes
- Manejar errores de dominio apropiadamente
- No hacer rollback analysis, enfocarse en que funcione

## Cálculos
- Usar valores de base de datos para porcentajes
- No recalcular en repository si ya está en DB
- Usar vista fix_labors_list para cálculos de labores

## Pruebas con curl
- Usar estos headers: X-API-KEY abc123secreta, 
X-USER-ID 123

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nachoo07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
