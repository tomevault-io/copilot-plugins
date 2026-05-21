## mcp-servers

> Este es un repositorio centralizado para servidores MCP (Model Context Protocol) que permite a los modelos de lenguaje como Claude interactuar con herramientas y servicios externos. Cada servidor MCP en este repositorio proporciona capacidades específicas, desde extracción de contenido web hasta operaciones en repositorios Git.

# MCP Servers - Contexto del Proyecto

## 🌐 Overview

Este es un repositorio centralizado para servidores MCP (Model Context Protocol) que permite a los modelos de lenguaje como Claude interactuar con herramientas y servicios externos. Cada servidor MCP en este repositorio proporciona capacidades específicas, desde extracción de contenido web hasta operaciones en repositorios Git.

## 🎯 Meta General

El objetivo principal es crear una colección bien organizada de servidores MCP que puedan ser utilizados con Cursor para potenciar las capacidades de los modelos de lenguaje. Estos servidores permiten a Claude y otros LLMs realizar tareas que normalmente estarían fuera de su alcance, como:

- Extraer y procesar contenido web (Firecrawl)
- Interactuar con navegadores web (Browser Tools)
- Gestionar contenedores Docker
- Crear flujos de trabajo automatizados (N8N)
- Interactuar con bases de datos (Supabase)
- Y mucho más

## 📂 Estructura por Responsabilidades

El proyecto sigue una arquitectura organizada por características, donde cada servidor MCP tiene su propio directorio con todos los componentes necesarios:

```
MCP_servers/
├── .cursor/              # Configuración específica de Cursor
├── Archon/               # Servidor MCP Archon
├── browser-tools-mcp/    # Herramientas para interactuar con navegadores
├── docker-mcp/           # Servidor MCP para Docker
├── firecrawl-mcp-server/ # Servidor para extracción web
│   ├── scripts/          # Scripts para usar Firecrawl
│   │   ├── scraper.py    # Script principal de extracción
│   │   └── simple_scraper.py # Versión simplificada
├── markdown_docs/        # Documentos extraídos con Firecrawl
├── mcp-server-git-source/ # Servidor para operaciones Git
├── n8n-workflow-builder-mcp-server/ # Servidor para workflows
├── mcp.json              # Configuración de servidores MCP
└── README.md             # Documentación general
```

## 🔧 Servidores MCP Disponibles

### Firecrawl MCP
- **Propósito**: Extracción y procesamiento de contenido web
- **Capacidades**: Scraping, crawling, búsqueda, extracción, investigación profunda
- **API Key**: Requerida (configurada en mcp.json)
- **Herramientas disponibles**:
  - firecrawl_scrape: Extrae contenido de una URL
  - firecrawl_map: Mapea la estructura de un sitio
  - firecrawl_crawl: Explora sistemáticamente un sitio web
  - firecrawl_batch_scrape: Extrae múltiples URLs en paralelo
  - firecrawl_check_batch_status: Verifica estado de extracciones por lotes
  - firecrawl_search: Busca información en la web
  - firecrawl_extract: Extrae datos específicos de páginas web
  - firecrawl_deep_research: Realiza investigación profunda sobre un tema
  - firecrawl_generate_llmstxt: Genera formato optimizado para LLMs

### Browser Tools MCP
- **Propósito**: Interacción con navegadores web
- **Capacidades**: Tomar capturas, revisar logs, auditorías web

### Supabase MCP
- **Propósito**: Interacción con bases de datos PostgreSQL
- **Capacidades**: Consultas SQL, gestión de datos

### Git MCP
- **Propósito**: Operaciones con repositorios Git
- **Capacidades**: Commits, branches, pull requests

### N8N Workflow Creator
- **Propósito**: Creación y gestión de flujos de trabajo
- **Capacidades**: Automatización de tareas, integración de servicios

### Archon
- **Propósito**: Servidor MCP adicional con capacidades avanzadas

## 💻 Uso Común

### Configuración de un Nuevo Servidor MCP en Cursor
1. Abrir Configuración > Características > Servidores MCP
2. Agregar nuevo servidor MCP:
   - Nombre: nombre descriptivo
   - Tipo: "command"
   - Comando: comando específico del servidor
   - Variables de entorno: si son necesarias

### Uso de Firecrawl desde Python
```python
from firecrawl import FirecrawlApp

app = FirecrawlApp(api_key='tu-api-key')
result = app.scrape_url('https://ejemplo.com')
with open('resultado.md', 'w') as f:
    f.write(result['markdown'])
```

## 🔑 Variables de Entorno Importantes
- `FIRECRAWL_API_KEY`: Clave de API para Firecrawl

## 📝 Guías de Estilo y Mejores Prácticas
- **Organización por características**: Cada servidor MCP tiene su propio directorio con todos sus componentes
- **Módulos independientes**: Evitar dependencias innecesarias entre servidores
- **Documentación clara**: Cada script debe incluir comentarios descriptivos
- **Principios SOLID**: Seguir principios de diseño sólido, especialmente Responsabilidad Única
- **DRY (Don't Repeat Yourself)**: Evitar duplicación de código
- **Manejo de errores robusto**: Incluir adecuado manejo de excepciones

## 🚀 Próximos Pasos
- Asegurar que todos los servidores MCP estén correctamente configurados
- Documentar casos de uso específicos para cada servidor
- Crear scripts de ejemplo para demostrar la integración entre diferentes servidores
- Explorar nuevos servidores MCP que puedan añadir funcionalidades útiles

## 📚 Recursos y Documentación
- [Documentación oficial de MCP](https://docs.anthropic.com/en/docs/agents-and-tools/model-context-protocol-mcp)
- [Repositorio Firecrawl MCP](https://github.com/mendableai/firecrawl-mcp-server)
- [Cursor AI Settings](https://cursor.sh/docs/introduction/cursor-ai) 

---
> Source: [daniel-carreon/MCP_servers](https://github.com/daniel-carreon/MCP_servers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
