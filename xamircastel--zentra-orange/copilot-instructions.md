## zentra-orange

> Invitados [Xamir Castelblanco](mailto:xamir.castelblanco.ext@newryglobal.com) [Raquel Álvarez Villa](mailto:raquel.alvarez@newryglobal.com)

3 dic 2025

## Alineación portal Zentra Orange

Invitados [Xamir Castelblanco](mailto:xamir.castelblanco.ext@newryglobal.com) [Raquel Álvarez Villa](mailto:raquel.alvarez@newryglobal.com)

Archivos adjuntos [Alineación portal Zentra Orange](https://www.google.com/calendar/event?eid=N3ZhdjRhZmJtcmttbGlxODZ1Zmtnbmk1ZWcgeGFtaXIuY2FzdGVsYmxhbmNvLmV4dEBuZXdyeWdsb2JhbC5jb20) 

Registros de la reunión [Transcripción](?tab=t.omygg4nxif74) 

### Resumen

Xamir Castelblanco y Raquel Álvarez Villa discutieron la presión para finalizar los pendientes del proyecto, incluyendo los requerimientos de integración donde Xamir Castelblanco confirmó que el diseño para Orange no contempla la llamada al servicio de "register" para la suscripción, a diferencia de otros marketplaces. Los participantes detallaron el flujo de suscripción *one-click* o en la *landing page*, con Xamir Castelblanco explicando que la suscripción se activaría internamente sin necesidad de una API de la operadora para notificar, aunque sí aplicaría un fichero para las cancelaciones del servicio y los cobros futuros. Para avanzar, acordaron enfocarse en una versión funcional del portal básico, definir la arquitectura del proyecto (base de datos y servidor) en paralelo a las clarificaciones técnicas, y que Xamir Castelblanco actualizaría el mapa de *user journey* para reflejar explícitamente la *landing page* y las interacciones con actores externos como Volving y la Telco.

### Detalles

*Longitud de las notas: Estándar*

* **Presión y Pendientes del Proyecto** Xamir Castelblanco y Raquel Álvarez Villa reconocieron la presión y el movimiento para finalizar los pendientes del proyecto. Raquel Álvarez Villa comentó que había estado hablando con Michelle y Eduatus sobre el desarrollo, identificando algo que faltaba y que ella pensaba que ya había sido contemplado y discutido previamente ([00:00:00](?tab=t.omygg4nxif74#heading=h.a7kdd6cpck9m)). Posteriormente, al revisar las notas, encontró que lo que mencionaba sí se había discutido, pero ahora tenía dudas sobre el requerimiento técnico necesario ([00:00:55](?tab=t.omygg4nxif74#heading=h.xhhj7dh9rsvv)).

* **Requerimientos de Integración con la Operadora** Xamir Castelblanco explicó que inicialmente se solicitaron dos servicios de la operadora: uno para el envío de SMS y otro para el cobro, el cual se acordó que será para el próximo año. Xamir Castelblanco aclaró que el diseño de integración para Orange no incluye la llamada al servicio de "register", a diferencia de otros marketplaces, porque esto no se había contemplado ni mencionado desde el inicio ([00:00:55](?tab=t.omygg4nxif74#heading=h.xhhj7dh9rsvv)). Raquel Álvarez Villa preguntó cómo funcionaría la suscripción sin el servicio de "register", ya que un usuario debe aceptar (aunque sea con un clic) ([00:01:59](?tab=t.omygg4nxif74#heading=h.sifma2ansj3i)).

* **Flujo de Suscripción y Experiencia del Usuario** Xamir Castelblanco describió el flujo de suscripción: el usuario hace una recarga en el portal de Más Orange, ve la opción del portal Centra como beneficio, hace clic y es redirigido a la URL de ellos. Al llegar a su sitio, Xamir Castelblanco mencionó que, si el usuario hace clic en un modal para aceptar, se activa la suscripción de su lado sin necesidad de llamar al "register" de la operadora ([00:03:17](?tab=t.omygg4nxif74#heading=h.kjv6dm3j7jo4)). Raquel Álvarez Villa preguntó si no era necesario comunicarle esto a la operadora. Xamir Castelblanco confirmó que Michelle nunca ha especificado la necesidad de una llamada a una API de ellos para notificar estas suscripciones, pero sí se habló de un fichero para notificaciones de recarga (que no aplica para Centra) y para las cancelaciones ([00:04:30](?tab=t.omygg4nxif74#heading=h.k55wkh6a60xo)).

* **Notificaciones y Cobros Futuros** Xamir Castelblanco confirmó que el fichero sí aplicaría para las cancelaciones del servicio y que, una vez se inicie el tema de los cobros, ese reporte se comenzará a alimentar con los cargos aplicados a los usuarios. Mencionó que Michelle les dijo que tendrían un fichero, asumiendo que será un repositorio diferente al actual, donde se dejaría un Excel con las líneas para que el equipo de Hen aplicara los cargos. Raquel Álvarez Villa revisó el flujo de suscripción: después de la redirección, se mostraría una *landing page* para el clic de suscripción ([00:05:41](?tab=t.omygg4nxif74#heading=h.v8hjjvhkajrw)).

* **Flujo de Suscripción One-Click y PIN** Raquel Álvarez Villa resumió que, al hacer *one-click* o *click* en la *landing page*, activarían la suscripción y darían de alta al usuario. Xamir Castelblanco aclaró que, si el número de móvil del usuario viene en la URL cifrada, no será necesario enviar un PIN, pero sí un SMS de bienvenida con el enlace de acceso y una contraseña ficticia ([00:07:07](?tab=t.omygg4nxif74#heading=h.bu5q86hrxenp)). Xamir Castelblanco reiteró que la posición actual es que no necesitan integrarse con una API de la operadora para notificar las suscripciones, ya que Michelle no lo ha mencionado en correos o llamadas ([00:08:24](?tab=t.omygg4nxif74#heading=h.3nfv8jp42xge)).

* **Gamificación y Diseño del Portal** Xamir Castelblanco y Raquel Álvarez Villa acordaron avanzar con una versión funcional que se enfoque en el portal básico (suscrito y consume contenidos) ([00:10:50](?tab=t.omygg4nxif74#heading=h.igz8ijiqpe7)). Xamir Castelblanco sugirió que, en una segunda fase, se podría revisar y adaptar un módulo de gamificación para fomentar el *engagement* y los minijuegos. Raquel Álvarez Villa solicitó a Xamir Castelblanco que compartiera cómo visualizaba el diseño para que el equipo de diseño pudiera preparar el formato y avanzar ([00:11:45](?tab=t.omygg4nxif74#heading=h.lxcfuba7ads)).

* **Avance de Arquitectura y Diseño Visual (MOC)** Xamir Castelblanco mencionó la necesidad de definir la arquitectura, el esqueleto (bases de datos, servidor, etc.), ya que es crucial tener la base del proyecto ([00:11:45](?tab=t.omygg4nxif74#heading=h.lxcfuba7ads)). Él había compartido un MOC para que fuera la primera versión del borrador visual ([00:13:02](?tab=t.omygg4nxif74#heading=h.8cw2ppkxs3du)). Raquel Álvarez Villa comentó que los ajustes de *look and feel* y marca (como el logo y los colores) deben ser validados por la operadora, pero que podían avanzar con el MOC propuesto ([00:14:04](?tab=t.omygg4nxif74#heading=h.pqqh3d5k1n82)). Xamir Castelblanco sugirió avanzar con el MOC que compartió, ya que no tenían otra alternativa ([00:15:07](?tab=t.omygg4nxif74#heading=h.8ji1comikwjo)).

* **Ajustes al MOC y Componentes del Portal** Raquel Álvarez Villa compartió su pantalla para mostrar que el MOC no se veía bien en la vista móvil ([00:16:03](?tab=t.omygg4nxif74#heading=h.a1vqrehngagh)). Xamir Castelblanco explicó que el diseño final no se vería así, sino como el portal MSP actual ([00:18:02](?tab=t.omygg4nxif74#heading=h.y0pv8zurs48x)). Además, Xamir Castelblanco detalló cambios específicos: eliminar la barra de búsqueda del *header* y moverla al *footer* con una lupa, eliminar el carrito de compras, poner el logo de Orange en la parte superior derecha del *header*, y cambiar el color de los *bullet points* a naranja ([00:19:11](?tab=t.omygg4nxif74#heading=h.dtv7ihslclq)). Raquel Álvarez Villa enfatizó la necesidad de avanzar en la estructura real, como la vista de las categorías y la ficha del producto ([00:21:59](?tab=t.omygg4nxif74#heading=h.9pe7gmjgz7z7)).

* **Próximos Pasos de Desarrollo** Xamir Castelblanco comunicó que ya había pasado la primera versión del MOC a Luis Carlos y que presionaría para obtener una fecha para el "primer real" (prototipo real) del portal. Raquel Álvarez Villa enfatizó que necesitan el prototipo real para poder compartirlo con la operadora y hacer ajustes sobre el producto tangible ([00:23:03](?tab=t.omygg4nxif74#heading=h.mo937ynri23e)).

* **Documentación del Flujo del Usuario** Raquel Álvarez Villa solicitó a Xamir Castelblanco que elaborara un mapa completo del *user journey* con todas las interacciones requeridas (flechas de llamada a Evolving o sistemas de Masora) para tener el mapa claro y alineado con la operadora ([00:24:12](?tab=t.omygg4nxif74#heading=h.vxo320it22o)). Xamir Castelblanco compartió un archivo llamado "Centra características funcionales versión un" que contenía los flujos ([00:25:19](?tab=t.omygg4nxif74#heading=h.3zmkkjwl6a17)). Raquel Álvarez Villa sugirió actualizar el flujo para que reflejara explícitamente la *landing page* de aceptación del usuario ([00:26:35](?tab=t.omygg4nxif74#heading=h.esiy1k2aoiq1)).

* **Actualización del Flujo con Interacciones Externas** Xamir Castelblanco mostró que el flujo de suscripción incluye la redirección a la *landing page* para activar la suscripción. Raquel Álvarez Villa solicitó que se añadiera la intervención de otros actores como Volving y la Telco (para el SMS) en el flujo, para clarificar el responsable de cada paso ([00:27:38](?tab=t.omygg4nxif74#heading=h.4u97o2kjerl5)). Xamir Castelblanco se comprometió a actualizar el documento con los flujos y todo lo discutido en la reunión para que Raquel Álvarez Villa pueda compartirlo con el operador y presionar a Evolving con el desarrollo ([00:29:51](?tab=t.omygg4nxif74#heading=h.8j7gym9lnd7b)).

* **Avance en Paralelo y Arquitectura** Xamir Castelblanco expresó frustración por la lentitud en la clarificación de temas técnicos por parte de la operadora, como la encriptación, lo que detiene el desarrollo ([00:29:51](?tab=t.omygg4nxif74#heading=h.8j7gym9lnd7b)). Ambos acordaron que es necesario avanzar en paralelo y que el equipo debe definir la arquitectura, la base de datos y el servidor, más allá de la información pendiente de la operadora, para evitar retrasos ([00:30:50](?tab=t.omygg4nxif74#heading=h.inkzapqrsijy)).

### Pasos siguientes recomendados

- [ ] Xamir Castelblanco actualizará el documento Centra características funcionales versión 1 con los flujos de suscripción y acceso al portal, incluyendo las interacciones con Volving y Telco, y lo compartirá con Raquel Álvarez Villa para que lo comparta con la operadora.  
- [ ] Xamir Castelblanco presionará a Luis Carlos para que dé una fecha para la primera versión del prototipo real basado en el MOC y avance con el diseño.

*Revisa las notas de Gemini para asegurarte de que sean correctas. [Obtén consejos y descubre cómo toma notas Gemini](https://support.google.com/meet/answer/14754931)*

*Danos tu opinión sobre el uso de Gemini para tomar notas en una [breve encuesta.](https://google.qualtrics.com/jfe/form/SV_9vK3UZEaIQKKE7A?confid=FDtoQ6tEP1VKkCS8OyA5DxIROAIIigIgABgDCA&detailid=standard)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xamircastel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xamircastel)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
