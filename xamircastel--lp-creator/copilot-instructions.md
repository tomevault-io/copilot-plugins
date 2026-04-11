## lp-creator

> Invitados [Oscar Santiago Anguita](mailto:oscar.anguita@newryglobal.com) [Alvaro Bernal](mailto:alvaro.bernal@newryglobal.com) [Eduardo Arcones Jiménez](mailto:eduardo.arcones@newryglobal.com) [Xamir Castelblanco](mailto:xamir.castelblanco.ext@newryglobal.com) [Raquel Álvarez Villa](mailto:raquel.alvarez@newryglobal.com)

26 feb 2026

## Toma requisitos para Landings

Invitados [Oscar Santiago Anguita](mailto:oscar.anguita@newryglobal.com) [Alvaro Bernal](mailto:alvaro.bernal@newryglobal.com) [Eduardo Arcones Jiménez](mailto:eduardo.arcones@newryglobal.com) [Xamir Castelblanco](mailto:xamir.castelblanco.ext@newryglobal.com) [Raquel Álvarez Villa](mailto:raquel.alvarez@newryglobal.com)

Archivos adjuntos [Toma requisitos para Landings](https://www.google.com/calendar/event?eid=NGtlNWQ1aWdpanBhZXF2bDU5YWQyb3VzcGEgYWx2YXJvLmJlcm5hbEBuZXdyeWdsb2JhbC5jb20) [Notas de Gemini](https://docs.google.com/document/d/1R2LrcK4m8XT3lkV7YTnd7l7DCQrLDX2Buo-kQ9Dq4B4/edit?usp=meet_tnfm_calendar) 

### Resumen

Los participantes discutieron el objetivo de extraer un módulo de creación de *landing pages* basado en plantillas para uso diario, lo que permitiría a equipos como los de Raquel Álvarez Villa y Álvaro Bernal realizar modificaciones rápidas en el diseño sin depender del equipo de desarrollo, a pesar de que Raquel Álvarez Villa indicó que nunca vieron una herramienta interna previa con esa función. Se acordó que la herramienta debe permitir la selección de país y operador para precargar *templates*, generar una URL automáticamente, gestionar *scripts* de Google Tag Manager y contar con un registro de cambios, además de validar si es más viable desarrollar el módulo en un portal aparte o integrarlo en Way con Carlos y Luis Carlos.

### Detalles

* **Propósito General del Módulo de Landing Page:** El objetivo principal es extraer el módulo de creación de landing page para su uso en las operaciones diarias. Se confirmó que este es el objetivo global de la discusión.

* **Antecedentes de Herramientas Internas:** Se discutió la existencia de una herramienta interna previa que realizaba la misma función, pero dejó de utilizarse debido a que carecía de las características necesarias para la operación diaria, lo que llevó al equipo a solicitar desarrollos de landing page manuales al equipo de TI. No obstante, Raquel Álvarez Villa indicó que, si tal herramienta existió, ellas nunca la vieron, sugiriendo que podría haber sido algo del principio o un módulo manejado internamente por otros equipos, posiblemente de índole más técnica.

* **Revisión del Módulo de Creación de Landing Page de Way:** Hubo una conversación sobre si el módulo de Way para la creación de landing page era adecuado, especialmente porque se requería modificar la configuración de las landings por parte de los equipos de operación, no solo para uso interno de Way. La propuesta es aprovechar el trabajo funcional ya realizado por Xamir Castelblanco para el *telco* y adaptarlo.

* **Funcionalidad Principal del Creador de Landing Page:** Se acordó que la herramienta debe permitir la creación de *landing pages* basadas en plantillas (*templates*). La idea es liberar al equipo de desarrollo de cambios menores, permitiendo a otros departamentos (como el de Raquel Álvarez Villa o Álvaro Bernal) realizar modificaciones rápidas.

* **Requisitos para la Edición de Templates:** Se requiere que los usuarios puedan seleccionar el país y el operador para precargar un *template* que incluya configuraciones predeterminadas para los flujos habilitados (3G, OTP, enrichment, etc.). Las modificaciones deben ser sencillas, permitiendo cambios en el color de fondo, color de los textos y botones, mientras que cierta información esencial exigida por el operador (como nombre del servicio, tarifa y periodicidad) debe permanecer fija y ser traída automáticamente desde la configuración del servicio en Way.

* **Generación y Gestión de la URL y Scripts:** Se determinó que, además de la creación de *landings* a partir de *templates*, la herramienta debe ser capaz de generar automáticamente una URL con los parámetros del diseño configurado. También se consideró esencial la funcionalidad de insertar y gestionar *scripts* de Google Tag Manager.

* **Flujo de Trabajo para Guardar y Publicar Diseños:** Se discutió el proceso de guardado y publicación de diseños, siendo esencial la capacidad de modificar un diseño incluso después de su publicación. Se concluyó que lo más práctico es permitir que los cambios se sobrescriban al republicar, con la adición de una sección de registro (*login*) de cambios para identificar quién realizó las últimas modificaciones.

* **Decisión sobre la Interfaz de Usuario y Tecnología:** Xamir Castelblanco compartió el trabajo conceptual que habían desarrollado previamente para el *telco*. Se expresó preocupación por usar el estilo de Way debido a que su tecnología es obsoleta y cero intuitiva, además de la necesidad de tener una previsualización de la *landing page*, algo que la tecnología actual de Way podría no soportar. Se acordó validar con Carlos y Luis Carlos si tecnológicamente es más viable desarrollar el módulo en un portal aparte o integrarlo como un módulo en Way, buscando el camino más práctico para TI.

* **Próximos Pasos y Documentación:** Xamir Castelblanco propuso comenzar a crear un documento base para la iteración con los equipos técnicos, detallando los aspectos discutidos, incluyendo el levantamiento de información sobre países, operaciones, flujos y templates asociados. Se confirmó que existe un pedido de tecnología previo y que el trabajo funcional legado del creador de *landing page* está archivado.

* **Revisión y Feedback Adicional:** Álvaro Bernal tomará notas de la reunión y las compartirá con Xamir Castelblanco. También se solicitó que Álvaro Bernal se reúna con Edu para informarle de lo discutido y obtener su retroalimentación sobre la funcionalidad deseada.

### Pasos siguientes recomendados

- [ ] Xamir Castelblanco validará con Carlos y Luis Carlos la viabilidad tecnológica de crear el módulo de landing page en un portal aparte o dentro de Way en términos de tiempo e implementación, y comenzará a crear un documento base con lo hablado para iterar con los equipos técnicos.  
- [ ] Alvaro Bernal hablará con Diego para confirmar la necesidad de un service request para el pedido de tecnología.  
- [ ] Alvaro Bernal se juntará con Edu para contarle lo conversado y obtener su feedback sobre la funcionalidad del creador de landing page.

*Revisa las notas de Gemini para asegurarte de que sean correctas. [Obtén consejos y descubre cómo toma notas Gemini](https://support.google.com/meet/answer/14754931)*

*Danos tu opinión sobre el uso de Gemini para tomar notas en una [breve encuesta.](https://google.qualtrics.com/jfe/form/SV_9vK3UZEaIQKKE7A?confid=-jbGzrTdQZNJml2dnebsDxIQOAIIigIgABgDCA&detailid=standard)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xamircastel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xamircastel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
