## material-ingestion-pipeline

> Role & Expertise Definition Rules

# Role & Expertise Definition Rules for step1_material_ingestion

## 1. Overview

This document defines the roles, responsibilities, and expertise boundaries for the AI agents involved in the **step1_material_ingestion** pipeline. Each agent is designed to handle a specific function—from document loading to graph construction—while adhering to strict modularity and robust chain-of-thought (CoT) reasoning for complex decision-making.

## 2. Domain & Technologies

### Programming Language & Frameworks
- **Language:** Python (adherence to PEP8 coding standards is mandatory)
- **Frameworks & Libraries:**
  - **Document Handling:** PyMuPDF for PDF processing
  - **NLP & Text Processing:** spaCy, OpenAI API (for dynamic chain‑of‑thought prompting)
  - **Computer Vision & OCR:** OpenCV, pytesseract
  - **Embedding & Similarity:** Sentence Transformers (e.g., all-MiniLM-L6-v2), FAISS
  - **Graph Construction:** networkx
  - **Chain-of-Thought Integration:** LangChain and Autogen (for prompting, CoT logging, and fallback triggers)

### Domain Expertise
- **Data Ingestion:** Understanding of file parsing, image extraction, and text segmentation.
- **Natural Language Processing:** Expertise in text segmentation, keyword extraction, and context summarization.
- **Computer Vision:** Knowledge of image analysis, region detection, and OCR post-processing.
- **Vector Similarity & Storage:** Familiarity with embedding generation and FAISS-based retrieval.
- **Knowledge Graphs:** Experience with hierarchical graph structures and node-link representations.
- **API & Modular Integration:** Ability to integrate third-party APIs (e.g., OpenAI) while ensuring robust error handling and fallback mechanisms.

## 3. Capability Boundaries

- **Modular Responsibility:**  
  Each agent is strictly responsible for its designated task. For example, the Document Loader Agent should only focus on extracting text and images from PDFs and must not handle transcript segmentation or image OCR.

- **Chain-of-Thought Reasoning:**  
  Agents must incorporate dynamic chain‑of‑thought (CoT) prompts for context-driven decision-making. When complex reasoning or fallback triggers are required, each agent should log its intermediate CoT steps.

- **Fallback Strategies:**  
  In scenarios where confidence levels fall below defined thresholds (e.g., image analysis in the Vision Model Agent), agents are expected to invoke predefined fallback mechanisms rather than attempting to “force” a result.

- **Error Handling:**  
  Agents should validate inputs against predetermined JSON schemas and handle exceptions gracefully, ensuring that errors are logged with sufficient context (including CoT details) for later review.

## 4. Role-Specific Responsibilities

### Document Loader Agent
- **Responsibilities:**  
  - Load PDF documents.
  - Extract text and images using PyMuPDF.
  - Generate unique identifiers for each document and its components.
- **Expertise Required:**  
  - PDF processing, file I/O, and image extraction.

### Transcript Agent
- **Responsibilities:**  
  - Process and segment transcript files (.vtt, .txt).
  - Use spaCy for basic NLP-based segmentation with room for CoT refinement.
- **Expertise Required:**  
  - Natural Language Processing, text segmentation, and error handling.

### Context Extraction Agent
- **Responsibilities:**  
  - Extract key phrases, entities, and context summaries from transcript chunks.
  - Leverage both rule-based extraction and CoT reasoning to capture nuances.
- **Expertise Required:**  
  - NLP, context analysis, and entity recognition.

### Vision Model Agent
- **Responsibilities:**  
  - Analyze images in the context of transcript-derived context.
  - Generate bounding boxes and descriptions for detected regions.
  - Invoke fallback models if confidence thresholds are not met.
- **Expertise Required:**  
  - Computer vision, image analysis, and integration with CoT prompting (e.g., OpenAI API).

### Targeted Image Extraction Agent
- **Responsibilities:**  
  - Re-run OCR on specific image regions identified by the Vision Model Agent.
  - Use OpenCV and pytesseract for enhanced extraction.
- **Expertise Required:**  
  - Image processing, OCR techniques, and post-processing of extracted text.

### Embedding Agent
- **Responsibilities:**  
  - Convert text and image information into high-dimensional vector embeddings.
  - Utilize models like Sentence Transformers and CLIP (or equivalents).
- **Expertise Required:**  
  - Deep learning for embeddings, vector space representations, and model integration.

### Vector Storage Agent
- **Responsibilities:**  
  - Store and query embeddings using FAISS.
  - Map source identifiers to vector representations.
- **Expertise Required:**  
  - FAISS, vector similarity search, and index management.

### Concept Extraction Agent
- **Responsibilities:**  
  - Extract key concepts from both text and image data.
  - Normalize duplicates via embedding similarity analysis.
- **Expertise Required:**  
  - NLP, concept extraction, and embedding comparison.

### Graph Builder Agent
- **Responsibilities:**  
  - Construct a hierarchical knowledge graph from the extracted concepts.
  - Manage node relationships (Course, Lesson, Topic, Concept).
- **Expertise Required:**  
  - Graph theory, networkx, and hierarchical data modeling.

### Coordinator Agent
- **Responsibilities:**  
  - Orchestrate the overall workflow, invoking each sub-agent in sequence.
  - Validate data consistency and manage versioning and QA.
- **Expertise Required:**  
  - System orchestration, error propagation, and integration of modular outputs.

## 5. Communication & Integration Standards

- **Standardized JSON Schemas:**  
  All agents must exchange data using predefined JSON formats, ensuring uniformity in headers (metadata) and payloads.
  
- **Synchronous vs. Asynchronous Execution:**  
  Agents should clearly document when tasks are executed synchronously versus in parallel (particularly when using Autogen-driven parallel tasks).

- **Logging & CoT Details:**  
  Detailed logging of chain‑of‑thought reasoning steps is required for any complex decision or fallback trigger. This information is critical for debugging and continuous improvement.

## 6. Best Practices & Constraints

- **Modularity & Encapsulation:**  
  Each agent must reside in its own Python module and interact only through well-defined interfaces.
  
- **Documentation & Inline Comments:**  
  Code must include comprehensive inline documentation and comments that describe complex logic and the chain‑of‑thought reasoning process.

- **Error Propagation:**  
  Any low-confidence or error outputs must be communicated upstream (via the Coordinator Agent) with sufficient contextual information for resolution.

- **Adherence to Guidelines:**  
  All implementation details must be consistent with the broader project guidelines on code consistency, testing, communication protocols, and error handling as specified in related rule files.

## 7. Summary of Do’s and Don’ts

- **Do:**
  - Follow the separation-of-concerns principle strictly.
  - Log all chain‑of‑thought reasoning steps for complex decisions.
  - Validate inputs and outputs against established schemas.
  - Use fallback strategies when confidence is low.
  - Maintain comprehensive documentation and inline comments.

- **Don’t:**
  - Allow any agent to perform tasks outside its defined role.
  - Bypass error handling or fallback mechanisms.
  - Mix responsibilities between modules or compromise the chain-of-thought process.
  - Omit context details during inter-agent communications.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siddhant61) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
