## knowledge

> **DEVELOPER NOTE:** After every development session, you MUST update the `Development Log` section at the bottom of this file with a new entry for the current date. The entry should be a bulleted list summarizing all changes, new features, and bug fixes.

# Project: Agentic Knowledge Graph Management

**DEVELOPER NOTE:** After every development session, you MUST update the `Development Log` section at the bottom of this file with a new entry for the current date. The entry should be a bulleted list summarizing all changes, new features, and bug fixes.

This project implements a system for building, managing, and visualizing knowledge graphs from text documents, with a focus on using intelligent agents to organize and traverse the graph. It is based on the concepts outlined in the `AgenticGraphManagement.pdf` document in this repository.

## Core Concepts

The system is designed to address the limitations of traditional Retrieval-Augmented Generation (RAG) by creating a more dynamic and context-aware knowledge management system. The key ideas are:

- **Agent-driven Graph Management**: Instead of static similarity search, this system uses intelligent agents to manage and traverse the knowledge graph.
- **Specialized Agents**: Each agent becomes an expert on a specific topic cluster within the graph.
- **Dynamic Graph Formation**: The graph is built by connecting related text chunks using configurable similarity thresholds.
- **Adaptive Learning**: Agents can decide what information to ingest and when to collaborate with other agents.

## Core Components

The project is structured into several Python modules, each with a specific responsibility:

- **`main.py`**: The main entry point of the application. It instantiates and runs the `Runner`.

- **`src/runner.py`**: Orchestrates the entire knowledge graph construction and visualization process. It loads the configuration, runs the graph builder, processes the graphs with the parent model, and finally, visualizes the results.

- **`src/config_loader.py`**: A simple utility to load the project's configuration from the `config.yaml` file.

- **`src/graph_builder.py`**: The heart of the project. This module is responsible for:
    - Reading text files from the `data/input` directory.
    - Splitting the text into smaller chunks.
    - Generating vector embeddings for each chunk using a pre-trained transformer model (`bert-base-uncased`).
    - Constructing a **semantic similarity graph** by connecting chunks with high cosine similarity between their embeddings.
    - Using a triplet extraction model (`Babelscape/rebel-large`) to identify knowledge triplets (subject, predicate, object) within each chunk.
    - Constructing a **triplet-based knowledge graph** by representing chunks and their extracted triplets as nodes and edges.

- **`src/parent_model.py`**: This module implements the "Parent LLM" concept from the presentation. It uses a locally-run Large Language Model (LLM), accessed via an LM Studio endpoint, to perform higher-level processing on the generated graphs. Its main task is to:
    - Identify overarching themes or topics across all the text chunks.
    - Assign each chunk to a specific topic, effectively partitioning the graph.
    - Add "topic" nodes to the graph to represent these thematic groups.

- **`src/graph_visualizer.py`**: This module uses the `pyvis` library to create interactive HTML visualizations of the knowledge graphs. It color-codes nodes based on their assigned topics and provides a legend for easy interpretation.

- **`config.yaml`**: The main configuration file for the project. It allows users to customize various parameters, such as:
    - Input data directories.
    - Text chunking settings (size, overlap).
    - Embedding and triplet extraction models.
    - The parent model to be used.
    - A placeholder for the `agent_model`, which will likely be used to implement the "Specialized Agents" concept.
    - Database connection details (with a TODO to switch to Pinecone).

## Agentic Query Workflow

The system employs a sophisticated multi-agent workflow to handle user queries, moving beyond simple RAG to a more dynamic and collaborative process of question answering. This workflow is orchestrated by the `ParentAgent`.

1.  **Query Reception**: The process begins when a user submits a query through the web interface. The query is received by the `ParentAgent`.

2.  **Decomposition and Routing**:
    - The `ParentAgent` utilizes a specialized `router_agent`.
    - The `router_agent`'s task is to decompose the user's complex question into smaller, logical sub-questions.
    - For each sub-question, the `router_agent` consults an internal list of available specialist agents (child agents) and their topics. It then assigns each sub-question to the most relevant specialist.
    - This process results in a structured plan, typically a JSON object, outlining the sub-tasks and their designated agents.

3.  **Concurrent Sub-Query Execution**:
    - The `ParentAgent` executes the sub-queries concurrently using a `ThreadPoolExecutor` for efficiency.
    - Each sub-query is sent to the `query` method of its assigned child agent.

4.  **Specialist Agent Processing**:
    - Each child agent, upon receiving a sub-query, uses its own `StrandsAgent` instance to process the question.
    - The child agent has access to two primary tools:
        - `similarity_search_tool`: This tool searches the agent's local subgraph (its area of expertise) to find relevant text chunks to build context and answer the question.
        - `consult_expert_tool`: If the agent determines that the question requires knowledge from a different topic, it can use this tool to consult another specialist agent.

5.  **Inter-Agent Consultation**:
    - When `consult_expert_tool` is used, the request is sent back up to the `ParentAgent`.
    - The `ParentAgent`'s `consult_expert` method then invokes the target specialist agent in a clean, isolated environment to prevent conversation history contamination.
    - The expert's response is then routed back to the original agent that requested the consultation.

6.  **Synthesis**:
    - Once all sub-queries (including any consultations) are complete, the `ParentAgent` gathers the individual responses.
    - It then employs a `synthesis_agent`, which receives the original question and all the sub-answers.
    - The `synthesis_agent`'s role is to craft a single, comprehensive, and coherent final answer for the user, integrating the information from the various specialists.

7.  **Response to User**: The final, synthesized answer is sent back to the user via the web interface, along with the source documents that were used to generate the answer.

## How to Run

1.  **Install Dependencies**:
    ```bash
    pip install -r requirements.txt
    pip install Flask
    ```

2.  **Set up LM Studio**:
    - Download and install LM Studio.
    - Download a compatible model (e.g., a GGUF version of a model like Mistral or Llama).
    - Start the local server in LM Studio.

3.  **Run the application**:
    ```bash
    python main.py
    ```

4.  **View the output**:
    - The application will start a web server at `http://127.0.0.1:5000`.
    - Open this URL in your browser to see the graph visualization and interact with the agents.

## Dockerization

This project includes a `Dockerfile` to allow for easy containerization and deployment.

### Building the Image

To build the Docker image, run the following command from the project root:

```bash
docker build -t agentic-knowledge-graph .
```

### Running the Container

To run the application inside a Docker container, you need to provide your AWS credentials to the container so it can access Bedrock models. You can do this by passing them as environment variables.

```bash
docker run -it -p 5000:5000 \
  -e AWS_ACCESS_KEY_ID=<YOUR_AWS_ACCESS_KEY_ID> \
  -e AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY> \
  -e AWS_SESSION_TOKEN=<YOUR_AWS_SESSION_TOKEN> \  # If you are using temporary credentials
  -e AWS_REGION_NAME=<YOUR_AWS_REGION> \
  agentic-knowledge-graph
```

-   The `-it` flag runs the container in interactive mode, which is necessary to use the CLI.
-   The `-p 5000:5000` flag maps the container's port 5000 to your host machine's port 5000, so you can access the web UI.
-   The `-e` flags set the environment variables for your AWS credentials.

By default, the container will launch both the web UI and the CLI. You can change this by modifying the `run_mode` in `config.yaml` before building the image.


## Future Work

The project is still under development, and the following features are planned for the future:

- **Live Document Ingestion and Agent-led Classification**:
    - Implement an endpoint or a watched directory to allow new documents to be added to the system while it's running.
    - Instead of a global classification model, leverage the existing specialist agents to determine if a new document chunk belongs to their topic.
- **Inter-Agent Collaboration Protocol**:
    - Design and implement a mechanism for agents to communicate with each other.
    - If multiple agents claim a new document chunk, they will engage in a "conversation" (e.g., using a shared prompt with a higher-level LLM, or a predefined negotiation protocol) to resolve the conflict and decide on the best placement for the new information.
- **Dynamic Graph Updates**:
    - Once a new chunk is assigned to an agent, the system needs to dynamically update the main knowledge graph and the agent's subgraph with the new node and its connections.
- **Agent Implementation**: The `agent_model` needs to be implemented to create the "Specialized Agents" that will manage and traverse the subgraphs.
- **Fine-tuning Agents**: The presentation mentions fine-tuning the agents on their respective subgraphs to improve their expertise.
- **Model/Graph Drift**: A mechanism to detect and handle model or graph drift needs to be implemented to keep the knowledge graph up-to-date and relevant.
- **Vector Database Integration**: The current database setup is planned to be replaced with [Pinecone](https://www.pinecone.io/) for more efficient similarity search at scale.
- **Visualize Iterative Search Process**: Explore ways to visualize the agent's iterative search process within the knowledge graph to better understand its traversal and context building.

## Testing

The `tests/` directory contains scripts for verifying the functionality of the agentic system. These are integration tests that run parts or all of the main application pipeline.

**Important:** Due to the nature of these tests (launching subprocesses, modifying config files), they should be run individually to avoid interference and ensure accurate results.

### Running Tests

To run a specific test file, use the following command from the project root, replacing the filename as needed:

```bash
uv run python3 -m unittest tests/test_agent_communication.py
```

---

### `tests/test_agent_communication.py`

-   **Purpose:** This test evaluates the end-to-end query processing pipeline, from question decomposition and routing to inter-agent communication and final response synthesis.
-   **How it Works:**
    1.  The test's `setUp` method initializes the `Runner` and loads a pre-existing data build (graphs and agents).
    2.  It then populates the `OrgTable` from the loaded graph data.
    3.  The main test method, `test_query_routing_and_response`, iterates through a predefined list of questions designed to trigger different behaviors (e.g., simple routing, decomposition, out-of-scope queries).
    4.  For each question, it calls the `parent_agent.query()` method.
    5.  The question, routed topics, and final agent response are logged to a timestamped file in the `tests/results/` directory for manual inspection.
    6.  A simple assertion (`assertIsNotNone`) confirms that a response was generated.

---

### `tests/test_org_table_loading.py`

-   **Purpose:** This test verifies that the system correctly loads a pre-existing `OrgTable` from a JSON file when configured to do so, bypassing the need to regenerate it.
-   **How it Works:**
    1.  The `setUp` method creates a dummy `org_table.json` file and modifies the main `config.yaml` to set `load_from_file: true`.
    2.  The `test_loading_from_file` method launches the main application (`main.py`) as a subprocess.
    3.  It captures the output of the subprocess.
    4.  After a timeout, it terminates the subprocess and verifies the captured output.
    5.  **Assertions:**
        -   It asserts that the log message `Organizational table loaded from...` **is present**.
        -   It asserts that the log message `Populating organizational table...` **is not present**.
    6.  The `tearDown` method cleans up by restoring the original `config.yaml` and deleting the dummy JSON file.

---

## Development Log

**10/07/25**

- **Test Suite Refactoring:**
    - Created a new `tests/` directory to centralize all testing scripts.
    - Moved `test_agent_communication.py` and `test_org_table_loading.py` into the new directory.
    - Moved the `test_results` directory to `tests/results`.
    - Refactored both test scripts from standalone procedural scripts into class-based tests inheriting from `unittest.TestCase`. This allows them to be run with Python's standard test runner.
- **Test Isolation & State Management:**
    - Addressed and fixed several test failures caused by state leaking between test runs.
    - Fixed an `AttributeError` by ensuring the `ParentAgent`'s router and synthesis agents are re-initialized for each query, preventing state corruption.
    - Fixed a `NameError` in the `Runner` class.
    - Corrected a caching issue in the `ConfigLoader` to ensure it re-reads the configuration file on demand.
- **Documentation:** Added a new "Testing" section to `GEMINI.md` to describe the purpose of each test case and provide instructions on how to run them individually.
- **Interactive CLI:**
    - Created a new interactive command-line interface (CLI) in `src/cli.py` using Python's `cmd` module.
    - Implemented commands: `list_agents`, `org_table`, `agent_info`, `ping_agent`, `query`, and `exit`.
- **Concurrent Run Mode:**
    - Implemented a concurrent execution mode to run both the web UI (Flask) and the CLI at the same time.
    - The Flask app is launched in a background thread, allowing the CLI to run in the main thread.
    - Updated `config.yaml` to support a list for the `run_mode` parameter (e.g., `['web', 'cli']`).
    - Modified `src/runner.py` to handle the new concurrent mode.
- **Dockerization:**
    - Created a `Dockerfile` to containerize the application using a `python:3.10-slim` base image.
    - Added a `.dockerignore` file to exclude unnecessary files and reduce image size.
    - Documented the build and run commands, including how to pass AWS credentials, in a new "Dockerization" section in `GEMINI.md`.

**9/30/25**

- **Agentic Query Workflow Documentation:** Added a new "Agentic Query Workflow" section to `GEMINI.md` to outline the entire process from query decomposition to final answer synthesis.
- **Agent Consultation Stability:** Fixed a `botocore.errorfactory.ValidationException` that occurred during inter-agent consultation. The issue was caused by conversation history contamination. The fix involves creating a new, isolated `StrandsAgent` instance for each consultation call within the `ParentAgent`, ensuring a clean state for each interaction.
- **Enhanced Logging:**
    - Added logging in `src/parent_agent.py` to confirm when a sub-query is successfully answered by a child agent.
    - Implemented logging to indicate when the `ParentAgent`'s router consults the internal agent list (previously the org table) to route queries, improving traceability of the decomposition and routing process.
- **Developer Note:** Added a note at the top of `GEMINI.md` to remind the development agent to keep this log updated after every session.
- **Log Redirection:** Suppressed verbose terminal output by redirecting logs from the `strands` library to a dedicated `logs/strands.log` file in `main.py`.
- **UI Markdown Rendering:** Integrated the `showdown.js` library into `templates/index.html` to render agent responses and sources as Markdown, improving readability.
- **Systematic Testing:** Created a `TESTING_QUESTIONS.md` file containing a suite of questions to systematically test and observe different system behaviors (e.g., simple routing, decomposition, consultation).
- **Config Cleanup:** Fixed a minor typo in `config.yaml` (`enable_consolidation: falsex`).
- **Org Table Persistence:** Implemented saving the `OrgTable` to a JSON file (`org_table.json`) for inspection. This includes adding `save_to_json` and `load_from_json` methods to the `OrgTable` class and updating the runner to use them.
- **Org Table Generation:** Modified the runner logic in `src/runner.py` to ensure the `OrgTable` is regenerated from the graph data on every application launch, guaranteeing it is always up-to-date, even when loading graphs from a pre-existing build.

**9/17/25**

- **Question Decomposition and Synthesis:**
    - Implemented a question decomposition and synthesis workflow in `src/parent_agent.py`.
    - A `router_agent` now decomposes complex questions into sub-questions and routes them to the appropriate child agents. The agent is prompted to return a JSON object with the sub-tasks.
    ```python
    # src/parent_agent.py

    # New system prompt for question decomposition
    system_prompt = (
        "You are a master agent responsible for decomposing a complex question into sub-questions and routing them to the correct specialist agent."
        f"The available topics and their corresponding agents are: {list(self.child_agents.keys())}. "
        "Your task is to: "
        "1. Analyze the user's question."
        "2. Decompose it into one or more sub-questions."
        "3. For each sub-question, identify the most relevant topic from the available list."
        "4. Respond with a JSON object containing a list of these sub-questions and their assigned topics. The format should be: "
        '{'sub_tasks': [{'question': '<sub_question_1>', 'topic': '<assigned_topic_1>'}, {'question': '<sub_question_2>', 'topic': '<assigned_topic_2>'}]}'
        "Do NOT include any reasoning or conversational filler in your response. It must be only the JSON object."
    )

    self.router_agent = StrandsAgent(
        system_prompt=system_prompt,
        model=self.llm_client
    )
    ```
    - A `synthesis_agent` was added to create a single, comprehensive answer from the responses of the child agents.
    ```python
    # src/parent_agent.py

    synthesis_system_prompt = (
        "You are a master synthesis agent. Your purpose is to craft a single, comprehensive, and well-structured answer to a user's original question. "
        "You will be provided with the original question and a set of answers to sub-questions that have been gathered from specialist agents. "
        "Your task is to synthesize this information into a final response. If the sub-answers are contradictory or insufficient, acknowledge this limitation. "
        "Structure your response clearly. Do not simply list the sub-answers; integrate them into a holistic response."
    )

    self.synthesis_agent = StrandsAgent(
        system_prompt=synthesis_system_prompt,
        model=self.llm_client
    )
    ```
    - The `query` method in `ParentAgent` was updated to orchestrate this new workflow, including concurrent execution of child agent queries using a `ThreadPoolExecutor`.
    - A `_fallback_query` method was implemented to handle cases where the decomposition fails, ensuring robustness.

**8/19/25**

- **Iterative Topic Consolidation:** Implemented iterative consolidation in `src/parent_model.py` to merge topics until no further consolidations are possible.
- **Configurable Consolidation:** Added `enable_consolidation` flag to `config.yaml` and implemented conditional execution in `src/runner.py` to control topic consolidation.
- **Robust LLM Response Parsing:** Refined prompts and improved parsing logic in `src/parent_model.py` to better handle LLM output, including malformed responses, for consolidation instructions.
- **Improved Logging for LLM Responses:** Modified `src/parent_model.py` to log concise information about unexpected LLM response formats without exposing full raw output.
- **Process Timing:** Added timing information to major processes in `src/runner.py` (graph building, parent model processing, consolidation, agent creation, visualization, Flask app startup) for performance monitoring.
- **Source Display in UI:**
    - Modified `src/agent.py` to return source contents along with the LLM response.
    - Modified `src/app.py` to pass source contents to the UI.
    - Modified `templates/index.html` to display sources in a separate, toggleable pane.
- **Enhanced Debugging for Source Search:**
    - Added detailed debug logs in `src/agent.py` for similarity search, including initial random nodes, nodes popped, and reasons for skipping nodes.
    - Changed initial random node logging to `info` level and included full content for better visibility.
- **Configurable Search Similarity:** Added `search_similarity` parameter to `config.yaml` and integrated it into `src/runner.py` and `src/agent.py` for agent context search.
- **Bug Fixes & Stability:**
    - Fixed a logic error in `_similarity_search_context` in `src/agent.py` related to `node_embedding` scope.
    - Implemented a more granular state-loading mechanism to intelligently load graphs and agents, preventing unnecessary rebuilding.
    - Resolved a `TypeError` caused by an incorrect number of arguments passed to the `Agent` constructor.
    - Fixed a `TypeError` that occurred when saving the agent state by implementing a new method to only pickle serializable data.
    - Corrected a "Not Found" error in the UI by ensuring the graph visualization files were served correctly.
- **UI Enhancements:**
    - Added a dropdown menu to the UI to allow toggling between the cosine and triplet graph visualizations.
    - Implemented a slider in the UI to dynamically adjust the `search_similarity` threshold for agent context search.

**8/18/25**

- **Agent System:** Implemented a multi-agent system where specialized agents are created for each topic cluster in the knowledge graph. Each agent can be queried individually about its specific domain.
- **Web UI:** Created a web-based user interface using Flask to visualize the knowledge graph and interact with the agents. The UI allows users to select an agent and ask questions.
- **State Persistence:** Implemented a state persistence mechanism using `pickle` to save and load the processed graphs and agents. This allows the application to resume from where it left off without rebuilding the graphs.
- **Logging:** Added a centralized logging system to provide detailed visibility into the system's behavior.
- **Similarity Search:** Implemented a new similarity search method for agents based on a DFS-like graph traversal. This allows agents to dynamically build context based on the question's similarity to nodes in the graph.
- **Configuration:** Made the search sample ratio for the similarity search configurable in the `config.yaml` file.

**Note:** All new code should consistently use the centralized logging system (`src/logger.py`) for better traceability and debugging.

---

**8/27/25**

- **Strands Framework Integration:**
    - Integrated the Strands AI framework for agent management and orchestration.
    - Added `strands-ai` to `requirements.txt`.
    - Updated `config.yaml` to include `strands` configuration, specifying `parent_agent_model_id` (DeepSeek) and `child_agent_model_id` (Claude Sonnet).
    - Modified `main.py` to load `STRANDS_MODEL_ID` from `config.yaml` and set it as an environment variable for Strands.
- **Refactored Agent Classes:**
    - **`src/agent.py`**: Refactored the `Agent` class to use `strands.Agent` and `strands.models.BedrockModel`. The `_similarity_search_context` logic was moved into a `@tool`-decorated method (`similarity_search_tool`) within the `Agent` class.
    - **`src/parent_agent.py`**: Created a new `ParentAgent` class to act as a routing agent. This agent uses its LLM to determine the topic of a question and then forwards the query to the appropriate child agent.
- **Updated Orchestration:**
    - **`src/runner.py`**: Modified to create instances of the new `ParentAgent` and `Agent` (child agents). The `_create_parent_agent` method now instantiates the `ParentAgent` and passes the child agents to it.
    - **`src/app.py`**: Updated to interact with the `ParentAgent` for query handling and to pass the `child_agents` for dynamic `search_similarity` updates.
- **Multi-Model Support:** Configured the system to use different Bedrock models for the parent agent (DeepSeek R1) and child agents (Claude 3.5 Sonnet), leveraging their respective strengths.
- **Testing and Debugging:**
    - Created `test_agent_communication.py` to verify parent-child agent communication and routing.
    - Addressed and resolved several issues during implementation and testing:
        - `AttributeError: 'DecoratedFunctionTool' object has no attribute 'with_parameters'` (fixed by making the tool a method of the `Agent` class).
        - `TypeError: Agent.__init__() got an unexpected keyword argument 'llm'` (fixed by configuring the model via `STRANDS_MODEL_ID` environment variable and passing `model=` to `StrandsAgent`).
        - `KeyError: 'bedrock'` (fixed by removing explicit `BedrockModel` instantiation in `Agent` and `ParentAgent` as Strands handles it via environment variables).
        - `AttributeError: 'Agent' object has no attribute 'run'` (fixed by calling `StrandsAgent` instances directly).
        - `ValidationException` on tool names (fixed by sanitizing topic names to remove spaces).
        - `TypeError: unhashable type: 'AgentResult'` (fixed by converting `AgentResult` to string and stripping whitespace).
        - `EOFError` in test script (fixed by hardcoding the test question).
        - `ValidationException: The provided model identifier is invalid.` (fixed by using the correct Bedrock model ID for DeepSeek R1).
- **UI Query Fixes:**
    - Added a dropdown (`topic-select`) to `templates/index.html` for selecting the target agent topic.
    - Corrected the JavaScript fetch endpoint in `templates/index.html` from `/query` to `/query_agent`.
    - Ensured the selected `agent_topic` is included in the JSON payload sent from the UI to the backend.

**8/28/25**

- **ParentModel Deprecation & ParentAgent Enhancement:**
    - Migrated core graph processing functionalities from `src/parent_model.py` to `src/parent_agent.py`:
        - Implemented `identify_topics_from_graph` for LLM-driven topic identification.
        - Implemented `classify_and_assign_chunks` for assigning text chunks to identified topics and modifying the graph.
        - Implemented `consolidate_topics` for iterative topic consolidation.
        - Created `process_graph` in `src/parent_agent.py` to orchestrate these new graph processing methods.
    - Updated `src/runner.py` to exclusively use `ParentAgent.process_graph` for all graph processing steps, effectively deprecating `ParentModel`.
- **UI/Agent Communication & Context Retrieval Improvements:**
    - **JSON Serialization Fix:** Modified `src/agent.py` to correctly extract `response_text` and `source_contents` from Strands `AgentResult` objects, resolving `TypeError: Object of type Trace is not JSON serializable` during Flask's `jsonify` process.
    - **Redundant Topic Removal:**
        - Removed "Select Agent Topic" dropdown and associated JavaScript payload (`agent_topic`) from `templates/index.html`.
        - Updated `src/app.py` to no longer expect or use the `topics` argument in `create_app` and its `render_template` calls.
        - Updated `src/runner.py` to reflect the change in `create_app` signature.
    - **Context Search Parameter Tuning:**
        - Lowered `search_similarity` to `0.4` in `config.yaml` to make context retrieval less strict.
        - Increased `search_sample_ratio` to `0.8` in `config.yaml` to expand the initial search scope for relevant chunks.

---

**9/04/25**

- **Bedrock Model Compatibility:**
    - Fixed a bug in `src/parent_agent.py` where the API request payload was hardcoded for Anthropic models. The payload is now correctly formatted for DeepSeek models.
    - Updated the response parsing logic in `src/parent_agent.py` to correctly handle the output from DeepSeek models.
- **Strands Integration Stability:**
    - Resolved a `ValidationException` during topic consolidation by ensuring the `StrandsAgent` has a clean history for each API call. This prevents the model's previous reasoning from being sent back to the API.
    - Suppressed verbose, unformatted logging from the `strands` library by setting its logger level to `CRITICAL` in `main.py`.
- **Conditional LLM Logging:**
    - Introduced a `debug` section in `config.yaml` with a `log_llm_responses` flag.
    - Implemented logic throughout the application to log detailed LLM responses only when this flag is enabled.
    - Refined the debug logging to only output the final, cleaned LLM response, removing the noisy reasoning text from the console output.
- **Graph Visualization Enhancements:**
    - Assigned a unique, uniform color to all "Central Topic" nodes in the graph visualization to make them easily distinguishable.
    - Updated the legend to include an entry for the new topic node color.
    - Added detailed logging to `src/graph_visualizer.py` to help diagnose an issue with inconsistent node colors after topic consolidation.
- **Flask Auto-Reload:**
    - Disabled the automatic restart feature of the Flask development server by setting `use_reloader=False` in `src/runner.py`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/visheshnarayan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/visheshnarayan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
