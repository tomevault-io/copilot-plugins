## movie-guru-adk

> Movie Guru is an AI-powered movie recommendation chatbot built using the **AI Development Kit (ADK)** and powered by **Google Gemini** models. This project demonstrates a multi-agent orchestration pattern for building sophisticated AI applications on Google Cloud.

# Movie Guru ADK - Gemini Powered

Movie Guru is an AI-powered movie recommendation chatbot built using the **AI Development Kit (ADK)** and powered by **Google Gemini** models. This project demonstrates a multi-agent orchestration pattern for building sophisticated AI applications on Google Cloud.

## Gemini Model Stack

The application leverages several Gemini models, primarily hosted on **Vertex AI**, to provide high-quality responses and analysis:

*   **Gemini 2.0 Flash Lite**: The default model used for core orchestration and sentiment analysis. It provides a great balance of speed and reasoning capabilities.
*   **Gemini 2.5 Flash**: Available as a high-performance alternative for complex reasoning tasks.
*   **Gemma 3 (4b)**: Integrated via **Ollama** for local development and testing.

## Integration Architecture

The Gemini models are integrated through a unified interface:

1.  **LiteLLM Proxy**: Acts as a central gateway, providing an OpenAI-compatible API for Vertex AI Gemini models. This simplifies model management and fallback logic.
2.  **Movie Guru Agent**: The main backend agent that uses Gemini to understand user intent, search for movies, and maintain conversation context.
3.  **Analysis Agent (sub-agent)**: A specialized ADK-based agent that uses Gemini to perform real-time sentiment analysis on user interactions.

## Configuration for Gemini

The project is highly configurable via environment variables and configuration files:

*   **Model Selection**: Managed via `MODEL_NAME` (e.g., `openai/gemini-2.0-flash-lite`).
*   **Vertex AI Setup**: Configured in `litellm-proxy/litellm_config.yaml`, mapping Gemini models to Vertex AI endpoints.
*   **Key Environment Variables**:
    *   `GOOGLE_CLOUD_PROJECT`: Your GCP Project ID.
    *   `GOOGLE_CLOUD_LOCATION`: The region for Vertex AI (e.g., `us-central1`).
    *   `OPENAI_API_BASE`: Points to the LiteLLM proxy for unified access.

## Getting Started with Gemini

To run the application with Gemini models, ensure you have a Google Cloud project with the Vertex AI API enabled. The [Terraform](./terraform/README.md) scripts provided in the repository will help you provision the necessary infrastructure and set up permissions.

For more detailed information on each component, refer to the main [README.md](./README.md).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/srinandan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
