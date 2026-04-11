## api-curso

> Genera una API REST completa para prácticas estudiantiles sobre música. La API debe ser autocontenida, con datos ficticios y sin depender de modificaciones externas que afecten a otros usuarios.

# gemini.md

## 🎯 Objetivo

Genera una API REST completa para prácticas estudiantiles sobre música. La API debe ser autocontenida, con datos ficticios y sin depender de modificaciones externas que afecten a otros usuarios.

---

## 🧱 Requisitos Generales

1. La API debe seguir los principios de REST:

   * Uso correcto de endpoints
   * Uso adecuado de métodos HTTP
   * Respuestas en formato JSON
   * Códigos de estado HTTP correctos

2. Debe requerir autenticación mediante un token:

   * Token fijo (hardcoded)
   * Ejemplo: `Authorization: Bearer TOKEN123`
   * Todas las rutas deben validar este token

3. No debe usar base de datos externa:

   * Usar datos en memoria (arrays/objetos)
   * Esto evita conflictos entre estudiantes

---

## 🎵 Recursos (Endpoints)

Crea **5 endpoints principales**, cada uno representando un recurso musical:

1. `/artists`
2. `/albums`
3. `/songs`
4. `/playlists`
5. `/genres`

---

## 🔁 Métodos HTTP

Cada endpoint debe soportar los 5 métodos:

* `GET` → Obtener todos o uno por ID
* `POST` → Crear nuevo recurso
* `PUT` → Reemplazar recurso completo
* `PATCH` → Actualizar parcialmente
* `DELETE` → Eliminar recurso

---

## 📦 Datos Iniciales

Cada endpoint debe iniciar con **5 registros ficticios**.

### Ejemplo estructura (songs):

```json
{
  "id": 1,
  "title": "Midnight Flow",
  "artist": "Axel Sound",
  "duration": 180,
  "genre": "Trap"
}
```

⚠️ Importante:

* IDs únicos por recurso
* Datos realistas pero ficticios
* No depender de relaciones obligatorias entre recursos

---

## 🔐 Autenticación

* Middleware que valide el token
* Si no se proporciona o es incorrecto:

  * Responder con `401 Unauthorized`

---

## 📄 Documentación

Genera un archivo `README.md` que incluya:

### Contenido requerido:

1. Descripción de la API
2. Cómo ejecutarla
3. Explicación de autenticación
4. Lista de endpoints
5. Ejemplos de requests (curl o Postman)
6. Estructura de datos
7. Ejemplos de respuestas
8. Códigos de error

---

## 🧪 Consideraciones para uso en clase

* La API debe ser **stateless**
* Los cambios no deben afectar a otros estudiantes
* Idealmente reiniciar datos al reiniciar el servidor

---

## 🧠 Retos para estudiantes

Incluye en el README una sección con **10 retos**:

1. Obtener todas las canciones
2. Obtener un artista por ID
3. Crear una nueva playlist
4. Actualizar el nombre de un álbum
5. Eliminar una canción
6. Filtrar canciones por género (simulado)
7. Agregar una canción a una playlist (lógica interna)
8. Modificar parcialmente un artista
9. Validar campos obligatorios al crear recursos
10. Manejar errores correctamente (404, 400, 401)

---

## 📌 Notas finales

* Código limpio y estructurado
* Separación de rutas, controladores y middleware
* Comentarios claros
* Fácil de entender para principiantes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Axel-Meyrek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Axel-Meyrek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
