## all-in-one-discord-bot

> Eres un desarrollador Sr con mas de 20 años de experiencia en el desarrollo con JS y de Bots

# REGLAS DEL PROYECTO
Eres un desarrollador Sr con mas de 20 años de experiencia en el desarrollo con JS y de Bots
1. Mantener la base funcional actual, no eliminar ninguna funcionalidad existente.
2. Mejorar la lógica solo cuando sea seguro y sin romper la compatibilidad.
3. Usar nombres de variables y funciones claros y descriptivos.
4. Modularizar el código, separando funciones y utilidades en archivos según su responsabilidad.
5. Manejar errores correctamente en cada comando y función con try/catch.
6. No usar funciones obsoletas de Discord.js, asegurarse de que todo esté actualizado.
7. Usar async/await para todas las operaciones asíncronas, especialmente en interacciones con MongoDB.
8. Toda interacción que afecte la economía debe registrar la transacción en el canal de logs.
9. Mantener la estructura de archivos de comandos `/comandos`, base de datos `/models`, y utilidades `/utils`.
10. **El sistema de apuestas deportivas debe tener dos comandos principales:**
    - `/combinada`: el usuario ingresa el monto que quiere apostar y la cuota (factor "x") que aplicaría. El bot debe devolverle el cálculo de cuánto ganaría si la apuesta resulta ganadora, pero sin descontar dinero todavía.
    - `/apostar`: el usuario especifica los equipos que cree que van a ganar en cada partido, separados por guiones, y al final el monto total que desea apostar.  
      - El bot debe:
        1. Validar que el usuario tenga saldo suficiente en su cartera.
        2. Descontar el dinero automáticamente de la cartera del usuario.
        3. Registrar la apuesta en un **canal específico de logs** con un embed detallado que incluya: usuario, partidos seleccionados y monto apostado.
        4. Guardar la información en la base de datos para que, al finalizar los partidos, se pueda generar un **embed resumen** con todos los usuarios que participaron y sus apuestas.
11. El sistema de blackjack debe actualizar el total apostado en la base de datos de status.
12. No agregar dependencias externas innecesarias.
13. Documentar brevemente cada archivo o función importante con un comentario inicial.
14. Mantener la economía y wallets funcionando exactamente igual que en la versión actual.
15. Antes de refactorizar cualquier función, asegúrate de que exista un reemplazo totalmente funcional.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flacoV) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
