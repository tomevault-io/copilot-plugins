## ia

> - Ve a: https://console.cloud.google.com/apis/library/aiplatform.googleapis.com?project=sap-b1-ai-integration

# README - Configuración de Gemini con Vertex AI

## Pasos para habilitar la API:

### 1. Habilitar Vertex AI API en Google Cloud Console:
   - Ve a: https://console.cloud.google.com/apis/library/aiplatform.googleapis.com?project=sap-b1-ai-integration
   - Haz clic en "HABILITAR"

### 2. Verificar permisos de la cuenta de servicio:
   - Ve a: https://console.cloud.google.com/iam-admin/iam?project=sap-b1-ai-integration
   - Busca: gemini-sap-bot@sap-b1-ai-integration.iam.gserviceaccount.com
   - Debe tener el rol: "Vertex AI User" o "Editor"

### 3. Si necesitas agregar permisos:
   ```
   - Haz clic en el lápiz (editar) junto a la cuenta de servicio
   - Agrega el rol: "Vertex AI User"
   - Guarda los cambios
   ```

## Alternativa: Usar API Key directamente

Si prefieres usar API Key en lugar de service account:

1. Obtén tu API Key en: https://makersuite.google.com/app/apikey
2. Configúrala como variable de entorno:
   ```powershell
   $env:GEMINI_API_KEY="tu_api_key_aqui"
   ```

## Iniciar el servidor:
```bash
python manage.py runserver 9999
```

O ejecuta: `start_server.bat`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benjaS357)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/benjaS357)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
