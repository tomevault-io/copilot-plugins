## pielcheck

> Usage - Opción B: build-time (NEXT_PUBLIC_)

Usage - Opción B: build-time (NEXT_PUBLIC_)

1) Añade tu clave pública en tiempo de build:
   - Crea .env.local en la raíz del proyecto y añade:
       NEXT_PUBLIC_GEMINI_API_KEY=tu_clave_aqui
       NEXT_PUBLIC_GEMINI_BASE_URL=https://api.gemini.example

   Nota: NEXT_PUBLIC_* es reemplazada en build y queda visible en el bundle final.

2) Ejemplo de uso en el cliente:
   - Importa callGeminiModel desde src/lib/geminiClient:
       import { callGeminiModel } from './lib/geminiClient';
   - Llama al modelo:
       const resp = await callGeminiModel('models/gemini-flash-latest', { prompt: { text: 'Hola' } });

3) Ajustes:
   - Verifica la ruta exacta que espera tu proveedor para invocar el modelo.
     En este ejemplo se usa: {BASE}/v1/{model}:generate
   - Si tu proveedor espera otro formato del body (por ejemplo {input: [...]}) ajusta callGeminiModel y las llamadas.

4) Riesgos y recomendaciones:
   - La clave estará incluida en el bundle. Sólo usar si aceptas la clave pública.
   - Considera límites de uso o restricciones por dominio si el proveedor lo permite.
   - Si luego quieres ocultar la clave, cambia a un endpoint server-side que haga las llamadas en el servidor.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jlondonomi-eng)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jlondonomi-eng)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
