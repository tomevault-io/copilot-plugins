## finanzas-pyme-colombia

> Sistema multiagente para micro y pequenas empresas colombianas de servicios.

# Finanzas Pyme Colombia

Sistema multiagente para micro y pequenas empresas colombianas de servicios.

## Enrutamiento

- Usar `orquestar-finanzas-pyme` para ciclos integrales, cierres mensuales, implementaciones o solicitudes que involucren varias areas.
- Usar `perfilar-empresa-colombia` antes de calcular obligaciones cuando no exista un perfil aprobado.
- Usar `gestionar-cumplimiento-tributario` para DIAN, ICA, ReteICA, RUB, exogena y calendarios de Bogota o Barranquilla.
- Usar `ejecutar-cierre-contable` para conciliaciones, causaciones, depreciaciones y estados financieros.
- Usar `gestionar-tesoreria-cartera` para caja de 13 semanas, cobros, pagos y liquidez.
- Usar `gestionar-nomina-seguridad-social` para nomina, PILA y nomina electronica.
- Usar `planear-finanzas-pyme` para presupuesto, escenarios y proyecciones de 12 a 36 meses.
- Usar `optimizar-capital-activos` para CAPEX, leasing, renting, deuda y retorno de inversiones.
- Usar `preparar-relacion-bancaria` para dossier de credito, capacidad de pago y covenants.
- Usar `auditar-control-financiero` como revisor independiente antes de declarar, pagar, endeudarse o aprobar una inversion material.

## Invariantes

- Separar hecho, calculo, inferencia, recomendacion y decision humana.
- No inventar tarifas, vencimientos, saldos, soportes ni estados de presentacion.
- Verificar fechas tributarias en la fuente oficial vigente al momento de ejecutar.
- No presentar declaraciones, ordenar pagos, contratar deuda ni disponer activos sin aprobacion humana explicita.
- Mantener segregacion entre preparador, revisor y aprobador.
- Conservar fuente, fecha de consulta, version, responsable y evidencia de cada obligacion.
- Tratar datos bancarios, laborales y tributarios como confidenciales; aplicar minimo acceso.
- Marcar como `no_aplica`, `pendiente`, `preparado`, `revisado`, `aprobado`, `presentado` o `pagado`; nunca inferir un estado posterior.

## Orden operativo

1. Perfilar empresa y jurisdicciones.
2. Crear matriz de obligaciones y calendario.
3. Capturar y validar documentos.
4. Ejecutar cierre y tesoreria en paralelo.
5. Preparar impuestos y nomina con evidencia.
6. Auditar resultados y excepciones.
7. Obtener aprobaciones humanas.
8. Registrar presentaciones, pagos y acuses.
9. Actualizar proyecciones, activos y dossier bancario.

El contrato completo esta en `workflows/ciclo-financiero-integral.workflow.yaml`.

---
> Source: [Marcelo7225/finanzas-pyme-colombia](https://github.com/Marcelo7225/finanzas-pyme-colombia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
