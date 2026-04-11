## atrm-tu-sindicato

> Se ha integrado **Google Gemini API** como motor de IA para responder preguntas sobre los convenios colectivos de **Interiores** (Limpieza de edificios y locales) y **Viaria** (Limpieza pública viaria) de ATRM.

# Integración de Google Gemini API en ATRM

## 📋 Resumen de cambios

Se ha integrado **Google Gemini API** como motor de IA para responder preguntas sobre los convenios colectivos de **Interiores** (Limpieza de edificios y locales) y **Viaria** (Limpieza pública viaria) de ATRM.

---

## 🆕 Archivos creados

### 1. **`/api/chat-gemini.js`** - Nueva API Gemini
- Endpoint serverless que procesa preguntas y las envía a Google Gemini
- Soporta dos tipos de convenio:
  - `interiores`: Limpieza de edificios y locales
  - `viaria`: Limpieza pública viaria
- Incluye las bases de conocimiento completas de ambos convenios
- Implementa fallback con respuestas locales si la API falla
- Requisito: Variable de entorno `GEMINI_API_KEY`

```bash
POST /api/chat-gemini
Content-Type: application/json

{
  "pregunta": "¿Cuántos días de vacaciones tengo?",
  "tipo_convenio": "viaria"
}
```

### 2. **`/js/ia_viaria.js`** - Nueva clase IA para Viaria
- Clase `IAViaria` independiente para el convenio de viaria
- Estructura idéntica a `IAInteriores` para consistencia
- Carga casos específicos de viaria (`datos/casos_viaria.json`)
- Consulta Gemini API cuando no hay match local
- Método principal: `generarRespuesta(pregunta)`

### 3. **`CONFIGURAR_GEMINI.md`** - Documentación de setup
- Guía paso a paso para obtener API key de Gemini
- Instrucciones para Vercel, Netlify y Railway
- Solución de problemas comunes
- Información sobre límites y cuotas

---

## 📝 Archivos modificados

### 1. **`/js/ia_interiores.js`**
**Cambio:** Método `consultarAPI()` actualizado para usar Gemini

**Antes:**
```javascript
async consultarAPI(pregunta) {
  const convenioKB = `... base de datos incrustada ...`;
  const resp = await fetch('/api/chat', {
    // enviaba todo el contexto en el cuerpo
  });
}
```

**Ahora:**
```javascript
async consultarAPI(pregunta) {
  const resp = await fetch('/api/chat-gemini', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
      pregunta,
      tipo_convenio: 'interiores'
    })
  });
}
```

### 2. **`/js/ia_nueva.js`**
**Cambios:**
- Método `responder()` actualizado para usar Gemini como fallback
- Primero busca en datos locales (`casosData`)
- Si no encuentra coincidencia, consulta `consultarGemini()`
- Nuevo método `consultarGemini()` integrado

**Flujo:**
1. ¿Es saludo? → Responder con saludo
2. ¿Es solicitud de ayuda? → Mostrar ejemplos
3. ¿Hay match en casos locales? → Usar respuesta local
4. ¿No hay match? → **Consultar Gemini API** ← NUEVO
5. ¿Todo falla? → Respuesta genérica

---

## 🔧 Configuración requerida

### Variable de entorno
```bash
GEMINI_API_KEY=Tu_API_Key_De_Google_Gemini
```

### Plataformas soportadas
- **Vercel**: Settings → Environment Variables
- **Netlify**: Site settings → Build & deploy → Environment
- **Railway**: Variables → Add new variable
- **Desarrollo local**: Archivo `.env.local`

---

## 🚀 Flujo de procesamiento

### Para INTERIORES:
```
Pregunta del usuario
    ↓
window.iaInteriores.generarRespuesta(pregunta)
    ↓
├─ Buscar en casos_interiores.json
├─ Buscar en articulos (convenio_interiores_articulos.json)
├─ **CONSULTAR GEMINI API** ← (si no hay match)
└─ Fallback a FAQs
    ↓
Respuesta
```

### Para VIARIA:
```
Pregunta del usuario (index.html)
    ↓
window.iaViaria.generarRespuesta(pregunta)
    O
window.iaContextual.responder(pregunta)  ← ia_nueva.js
    ↓
├─ Buscar en casos locales
├─ **CONSULTAR GEMINI API** ← (si no hay match)
└─ Respuesta genérica
    ↓
Respuesta
```

---

## 📊 Modelos y tecnología

| Aspecto | Valor |
|---------|-------|
| **Proveedor IA** | Google AI (Gemini) |
| **Modelo** | `gemini-1.5-flash` |
| **Temperatura** | 0.3 (respuestas precisas) |
| **Max tokens** | 500 |
| **Costo** | GRATUITO (con límites) |
| **Límites** | 60 req/min, ~1000 req/día |

---

## ✅ Ventajas de Gemini

✅ **Gratuito** - API gratuita con límites generosos  
✅ **Rápido** - Modelo flash ultra-rápido  
✅ **Preciso** - Temperatura baja para respuestas consistentes  
✅ **Multimodal** - Soporta texto, imágenes, audio (futura extensión)  
✅ **Contexto largo** - Puede procesar convenios completos  
✅ **Actualizado** - Modelo reciente (2024)  

---

## 🔍 Ejemplos de uso

### Ejemplo 1: Pregunta sobre Viaria
```
Usuario: "¿Cuántos días de vacaciones tengo en viaria?"

Flujo:
1. ia_nueva.js busca en casos locales → Sin match
2. Llama a /api/chat-gemini con tipo_convenio='viaria'
3. Gemini procesa con contexto del convenio de viaria
4. Respuesta: "28 días laborales + días asuntos propios según art. XX..."
```

### Ejemplo 2: Pregunta sobre Interiores
```
Usuario: "¿Qué pasa si me hospitalizo?"

Flujo:
1. ia_interiores.js busca artículos → Sin match exacto
2. Consulta /api/chat-gemini con tipo_convenio='interiores'
3. Gemini responde con info de hospitalización del convenio
4. Respuesta: "5 días retribuidos por hospitalización de familiar..."
```

---

## 🛠️ Endpoint API

### POST `/api/chat-gemini`

**Request:**
```json
{
  "pregunta": "string - pregunta del usuario",
  "tipo_convenio": "interiores | viaria (opcional, por defecto interiores)"
}
```

**Response:**
```json
{
  "respuesta": "string - respuesta generada por Gemini o fallback",
  "fuente": "string - referencias del convenio",
  "modelo": "Google Gemini | Fallback",
  "tipo_convenio": "interiores | viaria"
}
```

**Errores:**
```json
{
  "error": "string - descripción del error",
  "respuesta": "fallback local"
}
```

---

## 📱 Compatibilidad

- ✅ Interiores.html - Usa `window.iaInteriores`
- ✅ Index.html - Usa `window.iaContextual` (ia_nueva.js)
- ✅ Chat flotante - Compatible
- ✅ Preguntas inline - Compatible
- ✅ Sugerencias - Compatible

---

## 🧪 Testing

### Test en consola del navegador
```javascript
// Para interiores
await window.iaInteriores.generarRespuesta('¿Cuántos días de vacaciones?')

// Para viaria (contextual)
await window.iaContextual.responder('¿Qué es el plus de nocturnidad?')

// Para viaria (directo)
await window.iaViaria.generarRespuesta('¿Cuántas horas trabajo a la semana?')
```

### Test desde terminal
```bash
curl -X POST https://tu-dominio.com/api/chat-gemini \
  -H "Content-Type: application/json" \
  -d '{
    "pregunta": "¿Cuántos días de vacaciones?",
    "tipo_convenio": "viaria"
  }'
```

---

## ⚠️ Consideraciones importantes

### Límites de Gemini API
- **60 solicitudes por minuto** (gratuito)
- **~1000 solicitudes diarias** (aproximado)
- Si superas estos límites, Gemini devuelve error 429

### Seguridad
- ❌ **NUNCA** compartas tu `GEMINI_API_KEY` públicamente
- ❌ **NO** la commites en Git
- ✅ Usa variables de entorno en producción
- ✅ Usa `.env.local` (ignorado en `.gitignore`) en desarrollo

### Performance
- Primer llamado a Gemini: ~1-2 segundos
- Llamados posteriores: ~500-800ms
- Respuestas locales (cache): <100ms

---

## 🔄 Migración desde OpenRouter/DeepSeek

Si antes usabas OpenRouter o DeepSeek:

1. **Elimina `OPENROUTER_API_KEY` y `DEEPSEEK_API_KEY`**
2. **Agrega `GEMINI_API_KEY`** en variables de entorno
3. **Verifica que `/api/chat-gemini.js` existe**
4. **Redeploy el sitio**
5. **Prueba en el chat**

---

## 📞 Soporte y contacto

- **Documentación oficial Gemini:** https://ai.google.dev/
- **Google AI Studio:** https://ai.google.dev/
- **ATRM:** 968 30 00 37

---

## 📅 Changelog

### Versión 1.0 (24 de enero de 2026)
- ✅ Integración de Google Gemini API
- ✅ Soporte para convenio de Interiores
- ✅ Soporte para convenio de Viaria
- ✅ Archivo de configuración y documentación
- ✅ Fallback a respuestas locales si API falla
- ✅ Compatible con Vercel, Netlify, Railway

---

**¡Actualizado a Google Gemini! 🚀**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacaparr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jacaparr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
