## juego-seguridad-pg

> Este proyecto es un viedo juego secillo para la enseñansa de  consptos de ciberseguridad para niños entre 9 y 16 años, se implementara un menu inicial con cuatro escenarios

# contexto del proyecto
Este proyecto es un viedo juego secillo para la enseñansa de  consptos de ciberseguridad para niños entre 9 y 16 años, se implementara un menu inicial con cuatro escenarios
mediante botones, los escenarios seran pishing, rasonware, contraseñas seguras pendiente a determinar el ultimo escenario

# Instruccones del proyecto
1. El juego se desarrollara utilizando las mejores practicas de desarrollo web, utilizando HTML, CSS y JavaScript.
2. El juego debe ser responsivo y funcionar en dispositivos móviles y de escritorio.
3. El juego debe ser accesible para personas con discapacidades.
4. El juego debe ser seguro y no recopilar datos personales de los usuarios.
5. El juego debe tener una interfaz amigable y atractiva para el público objetivo.
6. El juego debe incluir un sistema de puntuación y recompensas para motivar a losusuarios.
7. El juego debe incluir un sistema de ayuda o tutorial para guiar a los usuarios
8. El juego debe incluir un sistema de retroalimentación para informar a los usuarios sobre su progreso y desempeño.
9. El juego debe incluir un sistema de niveles o etapas para aumentar la dificultad
   progresivamente.
10. El juego debe incluir un sistema de logros o medallas para reconocer el progreso de los usuarios.
11. El juego debe incluir un sistema de música y efectos de sonido para mejorar la
    experiencia del usuario.
12. El juego debe desarrollarse utilizando la euristica de nielsen de diseño y desarrollo web modernas, como HTML5, CSS3 y JavaScript
13. El juego debe desarrollarse utilizando un enfoque modular y reutilizable.
14. El juego debe usar las mejores practicas de seguirdad y desarrollo web..

# Paleta de Colores para Juego de Ciberseguridad (Edades 9-16)

## --- Colores Principales ---
 Un azul moderno y confiable. Ideal para botones principales, encabezados y elementos de navegación. Transmite seguridad y tecnología.
AZUL-PRINCIPAL: #007BFF

Un verde azulado (teal) enérgico. Perfecto para elementos secundarios, íconos o fondos de tarjetas. Es vibrante y atractivo.
VERDE-AZULADO-SECUNDARIO: #17A2B8

--- Colores de Fondo y Neutros ---
Un gris muy claro para el fondo general de la aplicación. Ayuda a que los otros colores resalten y no cansa la vista.
FONDO-CLARO: #F8F9FA

Un gris oscuro para el texto principal. Ofrece un excelente contraste sobre el fondo claro sin ser tan duro como el negro puro.
TEXTO-PRINCIPAL: #343A40

Un gris medio para bordes, divisiones o texto secundario.
GRIS-MEDIO: #6C757D

## --- Colores de Acento y Notificación ---
Un verde brillante para indicar éxito. Ideal para mensajes de "¡Correcto!", validaciones acertadas o recompensas.
VERDE-EXITO: #28A745

Un rojo claro para errores o alertas. Usar para mensajes de "Inténtalo de nuevo" o para resaltar elementos maliciosos en los correos de phishing.
ROJO-ERROR: #DC3545

Un amarillo para advertencias o pistas. Perfecto para llamar la atención sobre algo sin indicar que es un error, como un botón de "Pedir una pista".
AMARILLO-ADVERTENCIA: #FFC107

### --- Ejemplo de Uso ---
- Fondo de la App: FONDO-CLARO (#F8F9FA)
- Texto General: TEXTO-PRINCIPAL (#343A40)
- Botones de Acción (Ej: "Revisar Correo"): AZUL-PRINCIPAL (#007BFF)
- Respuesta Correcta: El texto o un ícono se muestra en VERDE-EXITO (#28A745)
- Respuesta Incorrecta: El texto o un ícono se muestra en ROJO-ERROR (#DC3545)
- Íconos o Tarjetas interactivas: VERDE-AZULADO-SECUNDARIO (#17A2B8)

## Contexto de los escenarios
1. Componente de "Máquina de Escribir": Utiliza una librería de React como react-type-animation para renderizar el texto letra por letra, simulando el efecto de escritura.
2. Personalización del Contexto: Carga el contenido del archivo Markdown como una cadena de
   texto en tu componente. Antes de pasarlo al componente de "máquina de escribir", usa el
   método .replace() de JavaScript para sustituir dinámicamente los placeholders (ej.
   {{NOMBRE_USUARIO}}, {{CORREO_USUARIO}}) con los datos que el jugador ingrese al inicio
   del escenario.
## escenario 1
* El escenario de pishing  pesentara al inicio  un texto de contexto el cual se encuentra en src/data/scenario1/phishing_game_intro.md
* Los datos para la emulacion de los correos se encuentra en src/data/scenario1/phishing_emails_dataset.json

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samiducatolica) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
