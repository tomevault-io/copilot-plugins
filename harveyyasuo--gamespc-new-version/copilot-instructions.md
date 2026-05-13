## gamespc-new-version

> INFORMACIÓN DEL PROYECTO

INFORMACIÓN DEL PROYECTO

Configuración Base:

Proyecto: [GamesPC]

Package: [com.harvey.gamespc]

Kotlin Version: [2.1.10]

Java Version: [21]

Gradle Version: [gradle-8.12.1]

Target SDK: [35]

Min SDK: [23]





INSTRUCCIONES CRÍTICAS PARA GEMINI

&nbsp;EVITAR BUCLES Y ERRORES COMUNES

NUNCA generes código que ya existe en el proyecto
SIEMPRE realiza un commit git por cada cambio realizado, por mínimo que sea
SIEMPRE pregunta por el contexto específico antes de generar código
NO asumas la estructura del proyecto sin verificar
EVITA respuestas genéricas, sé específico al contexto actual
CONFIRMA la versión de dependencias antes de sugerir código



CHECKLIST ANTES DE GENERAR CÓDIGO



&nbsp;¿Confirmé la estructura actual del proyecto?

&nbsp;¿Verifiqué las dependencias existentes?

&nbsp;¿Entiendo el patrón arquitectónico usado?

&nbsp;¿El código sigue las convenciones del proyecto?

&nbsp;¿Incluí manejo de errores apropiado?

&nbsp;¿Consideré el ciclo de vida de Android?



ERRORES COMUNES A EVITAR



Context Leaks: Usar Application Context cuando sea apropiado

Memory Leaks: Limpiar listeners y observers

Main Thread: No bloquear el hilo principal

Lifecycle: Respetar el ciclo de vida de componentes

Resource Management: Cerrar streams y conexiones





NOTA IMPORTANTE: Antes de cada generación de código, GEMINI debe:



Preguntar por el contexto específico si no está claro

Confirmar la estructura existente del proyecto

Verificar compatibilidad con dependencias actuales

Proporcionar código completo y funcional, no fragmentos



REGLAS DE CODIFICACIÓN

&nbsp;KOTLIN BEST PRACTICES



Null Safety: Usar ?. y !! apropiadamente

Coroutines: Preferir viewModelScope y lifecycleScope

Extensions: Crear extensiones para código repetitivo

Data Classes: Para modelos inmutables

Sealed Classes: Para estados y resultados



EVITAR

kotlin// No hacer manejo de errores genérico sin contexto

try {

&nbsp;   // código

} catch (e: Exception) {

&nbsp;   // manejo genérico

}






---
> Source: [HarveyYasuo/GamesPC-new-version](https://github.com/HarveyYasuo/GamesPC-new-version) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
