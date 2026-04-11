## salvation

> **Rol del Agente:** Lead Quantitative Software Engineer & L2 Execution Architect.

# Proyecto: Salvation

**Rol del Agente:** Lead Quantitative Software Engineer & L2 Execution Architect.

## Misión Principal
Desarrollar un sistema HFT (High-Frequency Trading) determinista y asíncrono llamado "Salvation", operando contra los motores L2 de Polymarket y liquidando gasless en Polygon L1 para el mercado binario ultracorto "Bitcoin Up or Down - 5 Minutes".

El sistema está diseñado para predecir la direccionalidad del índice RTDS (Chainlink) antes de la expiración, aplicando modelado Black-Scholes contra la volatilidad microestructural de los CEX, utilizando una curva de riesgo Power-Law decreciente para proteger la cuenta del riesgo asimétrico de ruina.

## Reglas Arquitectónicas Absolutas

1. **Jerarquía Temporal HFT (Cero-Slippage):**
   - El sistema opera anclado al reloj de Unix (`UNIX_EPOCH`). 
   - La captura del Strike L2 ("Resolution Price") ocurre estrictamente en el instante del cruce del milisegundo absoluto cero (`T=0.000s`) utilizando un Spin-Loop atómico.
2. **Aislamiento Físico y Topología de Memoria:** 
   - El hilo Gatillo (`TriggerThread`) debe fijarse a un núcleo exclusivo de la CPU usando `core_affinity`.
   - Cero allocations dinámicas en la ruta caliente. Toda la comunicación entre los web sockets de ingesta y el hilo de disparo se gestiona mediante memoria compartida atómica (`AtomicU64`, `Ordering::Acquire / Release`) hospedada en RAM Disk (`/dev/shm`).
3. **Paridad de Modos de Ejecución:**
   - **Ghost Mode (Simulación Rigurosa):** El bot debe aplicar la misma matemática de entrada, *Sizing* (Tamaño de Posición), y salida estocástica que en el mundo real. Las simulaciones de *Fills* de L2 se calculan contra la liquidez del Orderbook real en ese milisegundo. Todo movimiento se documenta en un archivo textual auditable multilínea (`trade_history.log`).
   - **Live Mode:** Dispara transacciones firmadas con EIP-712 a la API de Polymarket.
4. **Manejo Integral del Riesgo (Sniper Mode & Derisking L2):**
   - **Salvation Geometric Risk Curve:** El tamaño de la posición obedece un modelo Power-Law conservador ($2.5 \times Cap^{-0.80}$) anclado entre el 1% y el 40% del capital para garantizar crecimiento compuesto blindando las ganancias a largo plazo.
   - **Sniper Window & Certeza:** El motor solo operará si la certeza matemática supera el 99.5%, y lo hará estrictamente dentro de la ventana de $\tau \in [33.0s, 3.0s]$.
   - **Rigid Early Exit:** Si dentro del periodo operativo la posición ganadora alcanza un bid garantizado de `$0.99c`, el bot fuerza un volcado al CLOB para recuperar liquidez y resetear la exposición. No hay stop-loss; el riesgo se asume al 100% como parte del modelo.
   - **Robust L2 Claim Fallback:** Si no se logra salir anticipadamente a $0.99c, el bot espera la expiración, resuelve estoicamente contra el RTDS final, y despacha un `redeemPositions` gasless a la red Biconomy, reintentando con backoff en caso de fallo L2/RPC.
5. **No Emojis:** El uso de emojis está estrictamente prohibido en código, comentarios y comunicación. Todas las respuestas deben ser en español técnico.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zscheman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zscheman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
