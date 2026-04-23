## redux-todo-in-astro

> This documentation provides instructions for using the `local-rag` Docker image to run the Local RAG (Retrieval-Augmented Generation) service in containerized environments.


# Local RAG Docker Image Usage Guide

This documentation provides instructions for using the `local-rag` Docker image to run the Local RAG (Retrieval-Augmented Generation) service in containerized environments.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Getting the Docker Image](#getting-the-docker-image)
3. [Running with Docker Compose](#running-with-docker-compose)
4. [Configuration Options](#configuration-options)
5. [API Usage](#api-usage)
6. [Troubleshooting](#troubleshooting)

## Prerequisites

Before using the Local RAG Docker image, ensure you have:

- Docker Engine (20.10.0 or newer)
- Docker Compose (v2.0.0 or newer)
- Sufficient hardware resources:
  - At least 8GB RAM for running basic LLMs
  - At least 5GB free disk space
  - CPU with AVX2 instruction support for optimal performance

## Getting the Docker Image

You can obtain the Docker image in one of two ways:

### Option 1: Pull from Docker Hub

```bash
docker pull henry768/local-rag:latest
```

### Option 2: Build Locally

Clone the repository and build the image:

```bash
git clone https://github.com/githubhenrykoo/local_RAG.git
cd local_RAG
docker-compose build
```

## Running with Docker Compose

The recommended way to run the Local RAG service is with Docker Compose, as it automatically handles the Ollama service dependency.

### Step 1: Create or Edit docker-compose.yml

If you cloned the repository, a `docker-compose.yml` file is already provided. If you're creating one, use this template:

```yaml
version: '3.8'

services:
  local-rag:
    image: henry768/local-rag:latest
    ports:
      - "28302:28302"
    environment:
      - PORT=28302
    command: ["python", "app.py", "--port", "28302"]
    restart: unless-stopped
    depends_on:
      - ollama

  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    restart: unless-stopped

volumes:
  ollama_data:
```

### Step 2: Start the Services

```bash
docker-compose up -d
```

### Step 3: Verify Service is Running

```bash
curl http://localhost:28302/
```

You should see the HTML content of the web interface.

## Configuration Options

### Environment Variables

The Local RAG container accepts the following environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Port on which Flask will listen | 28302 |
| `ANONYMIZED_TELEMETRY` | Enable/disable ChromaDB telemetry | False |

### Command-Line Arguments

The following arguments can be passed to the app.py script:

| Argument | Description | Default |
|----------|-------------|---------|
| `--host` | Host address to bind to | 0.0.0.0 |
| `--port` | Port to bind to | 28302 |
| `--debug` | Run in debug mode | False |

### Persistent Storage

To persist documents and the vector database across container restarts, mount volumes for the `documents` directory and `chroma_db` directory:

```yaml
services:
  local-rag:
    # ...other settings...
    volumes:
      - ./documents:/app/documents
      - ./chroma_db:/app/chroma_db
```

## API Usage

The Local RAG service exposes the following API endpoints:

### 1. Query Documents

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"query": "What is Local RAG?"}' \
  http://localhost:28302/query
```

### 2. Upload Documents

```bash
curl -X POST \
  -F "file=@/path/to/document.pdf" \
  http://localhost:28302/upload
```

Supported file types: PDF, TXT, MD, Markdown, HTML, HTM

### 3. List Documents

```bash
curl http://localhost:28302/documents
```

### 4. Delete Documents

```bash
curl -X DELETE http://localhost:28302/documents/document_name.pdf
```

### 5. Rebuild Index

```bash
curl -X POST http://localhost:28302/rebuild-index
```

## Troubleshooting

### Common Issues and Solutions

1. **Ollama Connection Error**

   If you see `Connection refused` errors when querying:

   ```
   Error processing query: HTTPConnectionPool(host='ollama', port=11434): Max retries exceeded
   ```

   **Solution**: Ensure the Ollama container is running and the required models are loaded:

   ```bash
   docker-compose exec ollama ollama pull llama2
   docker-compose exec ollama ollama pull nomic-embed-text
   ```

2. **Port Conflicts**

   If port 28302 is already in use on your host:

   **Solution**: Change the port mapping in docker-compose.yml:

   ```yaml
   ports:
     - "8080:28302"  # Maps container port 28302 to host port 8080
   ```

3. **Memory Issues**

   If the container crashes due to memory pressure:

   **Solution**: Limit RAM usage in docker-compose.yml:

   ```yaml
   services:
     local-rag:
       # ...other settings...
       deploy:
         resources:
           limits:
             memory: 4G
   ```

4. **Slow Performance**

   If query responses are slow:

   **Solution**: Consider using a smaller LLM model by modifying the OllamaClient initialization in app.py:

   ```python
   ollama_client = OllamaClient(model="llama2:7b")  # Use smaller 7B parameter model
   ```

### Getting Logs

To check container logs:

```bash
docker-compose logs local-rag
docker-compose logs ollama
```

For real-time logs with following:

```bash
docker-compose logs -f local-rag
```

## Advanced Usage

### Custom Models

To use a different LLM model:

1. Pull the model in the Ollama container:

```bash
docker-compose exec ollama ollama pull mistral
```

2. Restart the Local RAG container to use the new model:

```bash
docker-compose restart local-rag
```

### Security Considerations

- The service is designed for local or trusted network use only
- No authentication is implemented by default
- Consider putting behind a reverse proxy (like Nginx) with authentication for production use
- Limit exposed ports if deploying on a public-facing server

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/githubhenrykoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
