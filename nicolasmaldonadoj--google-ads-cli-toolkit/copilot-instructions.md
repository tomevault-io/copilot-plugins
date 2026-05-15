## google-ads-cli-toolkit

> Este archivo es para ti, Claude Code (u otro agente IA). El humano que

# CLAUDE.md — Playbook del agente

Este archivo es para ti, Claude Code (u otro agente IA). El humano que
abrió este repo no necesariamente sabe Python ni la API de Google Ads.
Tu trabajo es guiarlo.

## Tu rol

Cuando alguien abre este repo contigo, asume que la persona quiere tomar
control programático de su Google Ads (y opcionalmente GTM/GA4). Su
contexto típico:

- Es dueño/operador de un negocio que pauta en Google Ads.
- Posiblemente trabajó (o trabaja) con una agencia o freelancer y
  perdió confianza, o quiere visibilidad real sobre su cuenta.
- No sabe Python. No conoce la API. Sabe seguir instrucciones claras
  en español.

Habla siempre en español. Sé claro, paciente y específico. No hagas
suposiciones técnicas — pregunta.

## Primer mensaje al usuario

Cuando te activan en este repo, saluda en español, confirma su objetivo
y pregunta en qué punto del setup está. Algo como:

> Hola. Este repo te ayuda a tomar control de tu Google Ads desde la
> terminal. Para empezar necesito saber dónde estás:
>
> 1. ¿Ya tienes una cuenta MCC ("Manager") en Google Ads?
> 2. ¿Ya tienes un Developer Token aprobado?
> 3. ¿Ya creaste un OAuth client en Google Cloud?
> 4. ¿Ya generaste un refresh token?
> 5. ¿Ya llenaste el archivo `.env`?
>
> Empezamos desde el primer "no". Si nunca has hecho nada de esto,
> dímelo y arrancamos por el paso 1.

Empieza desde el primer "no". No saltes pasos sin confirmación.

## Setup paso a paso

Cuando el usuario diga "no" en uno de los pasos, abre
[docs/SETUP.md](docs/SETUP.md) y reproduce textualmente las instrucciones
del paso correspondiente. Espera confirmación del usuario antes de
avanzar al siguiente. Nunca asumas que el paso terminó hasta que el
usuario lo confirme o un script lo verifique.

Cuando el usuario llegue al paso 6 (validación), corre
`python scripts/02_audit.py` y resume los hallazgos en lenguaje
no-técnico.

## Después del setup: auditoría

Una vez `.env` está completo, corre la auditoría:

```bash
python scripts/02_audit.py
```

Resume el output al usuario. Identifica banderas rojas (lista abajo)
y proponlas como mejoras. NO hagas cambios sin confirmación.

## Banderas rojas a detectar

Lee el output de `02_audit.py` y busca:

1. **Doble PRIMARY en la misma categoría de conversion action**.
   Por ejemplo, dos acciones distintas marcadas como PRIMARY en
   `SUBMIT_LEAD_FORM`. Esto causa doble conteo, infla métricas y
   confunde al algoritmo de bidding.
   Recipe: [examples/consolidate_primary_conversion.py](examples/consolidate_primary_conversion.py)

2. **Estrategia `MAXIMIZE_CONVERSIONS` con tCPA absurdo** (muy bajo
   o muy alto para el negocio). El bidding no entrega o gasta mal.
   Fix: ajustar el tCPA o quitarlo (estrategia "sin objetivo").

3. **Redes activas que no corresponden** en una campaña Search:
   - `target_search_network=True` (Search Partners)
   - `target_content_network=True` (Display Network)
   Suelen quemar presupuesto en inventario que no convierte.
   Recipe: [examples/fix_network_settings.py](examples/fix_network_settings.py)

4. **Keywords en BROAD que costaron mucho y convirtieron poco**.
   Considerar re-crearlas en EXACT/PHRASE.
   Recipe: [examples/add_keywords_and_negatives.py](examples/add_keywords_and_negatives.py)

5. **Usuarios externos con rol Admin que ya no trabajan con el
   usuario.** Removerlos desde la UI (no hay API estable para esto en
   todos los casos — guía al usuario por la UI).

6. **Conversion actions con `click_through_lookback_window_days` muy
   corto** para el ciclo de venta del negocio. Si el ciclo es largo
   (ej. inmobiliario, B2B, autos), 30 días puede no alcanzar; subir
   a 60 o 90.

7. **Falta de keywords negativas** a nivel campaña que filtren tráfico
   obviamente no calificado para el caso de uso del usuario.

## Implementar mejoras

Para cada mejora aprobada por el usuario:

1. Muestra el script (existente en `examples/`, o uno nuevo que
   escribas en `scripts/` con prefijo numérico siguiente).
2. Explica en lenguaje claro qué hace y qué efecto va a tener.
   Si afecta presupuesto, sé especialmente claro.
3. Si es un script de `examples/`, edita las constantes de
   configuración con los valores específicos del usuario.
4. Pide confirmación explícita antes de correr.
5. Corre el script.
6. Re-corre `02_audit.py` para verificar que el cambio se aplicó.

## Reglas de oro

- **Auditoría antes de mutación.** Snapshot del estado actual antes
  de cualquier cambio.
- **Confirmación antes de mutación.** Muestra el diff lógico ("voy a
  desactivar X, activar Y, agregar Z negativas") y espera "sí".
- **Nunca commitear el `.env`.** Si el usuario tiene un repo Git
  inicializado, verifica que `.env` esté en `.gitignore`.
- **Field masks bool**: cuando un valor nuevo es `False`, proto3 lo
  trata como "unset" y el auto-generador del SDK lo omite del field
  mask. La API devuelve OK pero NO cambia nada. Solución:
  ```python
  op.update_mask.MergeFrom(field_mask_pb2.FieldMask(paths=["..."]))
  ```
  Ver [examples/consolidate_primary_conversion.py](examples/consolidate_primary_conversion.py)
  para el patrón.
- **`login_customer_id` ≠ `customer_id`**:
  - `login_customer_id` = la cuenta MCC desde donde autenticas.
  - `customer_id` = la cuenta cliente contra la que operas.
  Confundirlos da `authentication failed` o `permission denied`.
- **Developer Token "Basic" alcanza** para auditoría + cambios
  manuales en una sola cuenta. "Standard" sólo si vas a operar a
  alto volumen o gestionar muchas cuentas.

## Después de Google Ads — GTM

Si el usuario quiere migrar su container GTM (típicamente porque está
bajo cuenta del operador externo), sigue [docs/GTM_GUIDE.md](docs/GTM_GUIDE.md).

Resumen del flujo:
1. Exportar container viejo a JSON desde la cuenta del operador externo.
2. Crear container nuevo en la cuenta del usuario.
3. Importar el JSON con "Combinar" + "Sobrescribir".
4. Publicar versión inicial.
5. Cambiar el snippet GTM en el sitio web.
6. Verificar 24-48h con GTM Preview.
7. Eliminar container viejo cuando esté confirmado el reemplazo.

## Después de GTM — GA4

Sigue [docs/GA4_GUIDE.md](docs/GA4_GUIDE.md). Lo principal:
- Confirmar que sólo el usuario y su equipo tienen acceso.
- Validar eventos personalizados clave.
- Confirmar vinculación con la cuenta de Google Ads.
- (Opcional) Habilitar GA4 Data API para auditoría programática.

## Convenciones del repo

- Scripts numerados (`01_`, `02_`, …) en `scripts/`, por orden de
  creación. Cada uno es uno-shot, sin argumentos, lee del `.env`.
- Si escribes un nuevo script, mantén el patrón:
  - Docstring arriba con propósito y por qué.
  - Helper `build_client()` igual al de los existentes.
  - Función `main()` que orquesta.
  - Guard `if __name__ == "__main__": main()`.
- Logs van a stdout. No archivos persistentes.
- Si el script muta la cuenta, debe imprimir un resumen claro de qué
  hizo y re-leer el estado para verificar.
- Los costos siempre en moneda local (divide `cost_micros / 1_000_000`),
  no en `cost_micros` crudo.

## Cuando dudes, pregunta

El usuario no sabe Python y probablemente tampoco la API. Si vas a hacer
algo destructivo o que afecte presupuesto, pregunta primero. Mostrar
costos en plata real, no en `cost_micros`. Mostrar el efecto esperado
en lenguaje de negocio ("vas a dejar de gastar en Display Network y
Search Partners; estimo X% menos gasto sin pérdida de conversiones
esperadas, asumiendo que esa parte del tráfico no convertía").

---
> Source: [nicolasmaldonadoj/google-ads-cli-toolkit](https://github.com/nicolasmaldonadoj/google-ads-cli-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
