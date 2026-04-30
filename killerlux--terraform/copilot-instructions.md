## terraform

> PHASE 3 : Déploiement des Applications avec Docker Compose

PHASE 3 : Déploiement des Applications avec Docker Compose
Remplis docker-compose.yml avec ce contenu. Ce fichier définit les services n8n, Ollama, et ChromaDB.

      
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_service
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://${N8N_HOST}:5678/
      - GENERIC_TIMEZONE=Europe/Paris
    volumes:
      - ./n8n_data:/home/node/.n8n
    depends_on:
      - ollama
      - chroma
    networks:
      - private_ai_net

  ollama:
    image: ollama/ollama:latest
    container_name: ollama_service
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ./ollama_data:/root/.ollama
    networks:
      - private_ai_net

  chroma:
    image: chromadb/chroma:latest
    container_name: chroma_service
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - ./chroma_data:/chroma/.chroma/index
    networks:
      - private_ai_net

volumes:
  n8n_data:
  ollama_data:
  chroma_data:

networks:
  private_ai_net:
    driver: bridge

    

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/killerlux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
