## ingenieria-requisitos

> Esta carpeta es una HERRAMIENTA, no un proyecto: contiene un método completo

# Ingeniería de requisitos: instrucciones para el agente

Esta carpeta es una HERRAMIENTA, no un proyecto: contiene un método completo
para entrevistar a una persona de negocio y producir los planos (los spec
files) de su aplicación, con una web local para que los valide mirando.

1. Lee `RUNBOOK.md` ENTERO y síguelo al pie de la letra: es tu manual de
   actuación. Empieza por el triaje de modos (construir de cero, auditar
   código existente, o iterar unos planos).
2. Regla dura: NO escribas JAMÁS dentro de esta carpeta. Ni proyectos, ni
   specs, ni notas, ni temporales. El proyecto del usuario vive en SU
   carpeta de trabajo, fuera de aquí; si tu sesión está corriendo dentro de
   esta carpeta, pregúntale dónde quiere guardar su proyecto y trabaja allí.
3. Los scripts de la herramienta se invocan con la ruta de ESTA carpeta:
   `visor/servir.py` (la web local), `visor/validar.py` (validación de los
   planos), `visor/generar_spec.py` (el spec de un plano) y
   `visor/compilar.py` (la documentación completa de la aplicación).

---
> Source: [nategentile/ingenieria-requisitos](https://github.com/nategentile/ingenieria-requisitos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
