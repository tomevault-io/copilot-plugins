## deep-research-mcp-server

> @project-documentation(projectName: "open-deep-research - Ultimate Development Guide & Code Snippets Collection") {

# .cursorrules

@project-documentation(projectName: "open-deep-research - Ultimate Development Guide & Code Snippets Collection") {

  @section(name: "Project Overview", level: 1) {
    @project-overview {
      @short-description: "**open-deep-research: Your AI-Powered Research Assistant.**  Conduct iterative, deep research using search engines, web scraping, and Gemini LLMs, all within a lightweight and understandable codebase."
      @mcp-tool-availability: "**Seamlessly Integrate with AI Agents via MCP.**  Available as a Model Context Protocol (MCP) tool for easy integration into larger AI agent systems."
      @core-libraries: "**Powered by Key Libraries.** Leverages Firecrawl for efficient web data extraction and Gemini for advanced language understanding and report generation."
      @goal: "**Keep it Simple, Keep it Deep.**  Provides the *simplest* yet *most effective* implementation of a deep research agent, designed for clarity and easy extension (<500 LoC goal). "
      @workflow-reference: "**Workflow Diagram Included.**  Refer to the 'Project Workflow Diagram' section for a visual representation of the research process."
      @license: "**MIT Licensed.**  Freely use, modify, and build upon open-deep-research under the permissive MIT License."
    }
    @note: "**Key Project Philosophy:** 'open-deep-research' prioritizes simplicity and clarity, aiming to provide a foundational research agent that is easy to understand, modify, and extend.  It's designed to be a starting point for building more sophisticated AI-driven research tools."
  }

  @section(name: "Project Workflow Diagram", level: 1) {
    @workflow-diagram(description: "Mermaid flowchart representation of the Deep Research workflow (see README)") {
      @flowchart-mermaid {
        ```mermaid
        flowchart TB
            subgraph Input
                Q[User Query]
                B[Breadth Parameter]
                D[Depth Parameter]
            end

            DR[Deep Research] -->
            SQ[SERP Queries] -->
            PR[Process Results]

            subgraph Results[Results]
                direction TB
                NL((Learnings))
                ND((Directions))
            end

            PR --> NL
            PR --> ND

            DP{depth > 0?}

            RD["Next Direction:
            - Prior Goals
            - New Questions
            - Learnings"]

            MR[Markdown Report]

            %% Main Flow
            Q & B & D --> DR

            %% Results to Decision
            NL & ND --> DP

            %% Circular Flow
            DP -->|Yes| RD
            RD -->|New Context| DR

            %% Final Output
            DP -->|No| MR

            %% Styling
            classDef input fill:#7bed9f,stroke:#2ed573,color:black
            classDef process fill:#70a1ff,stroke:#1e90ff,color:black
            classDef recursive fill:#ffa502,stroke:#ff7f50,color:black
            classDef output fill:#ff4757,stroke:#ff6b81,color:black
            classDef results fill:#a8e6cf,stroke:#3b7a57,color:black

            class Q,B,D input
            class DR,SQ,PR process
            class DP,RD recursive
            class MR output
            class NL,ND results
        ```
      }
      @note: "**Workflow Visualization:** This Mermaid diagram provides a visual overview of the core research process within 'open-deep-research'.  Use it to understand the flow of data and control within the agent."
    }
  }

  @section(name: "Key Features", level: 1) {
    @features-section(description: "Key features of the open-deep-research agent") {
      @feature(name: "MCP Integration", description: "**MCP Ready:** Seamlessly integrates as a Model Context Protocol tool into AI agent ecosystems, enabling plug-and-play research capabilities.")
      @feature(name: "Iterative Research", description: "**Iterative Deep Dive:**  Explores topics deeply through iterative query refinement and result processing, mimicking the in-depth approach of expert human researchers.")
      @feature(name: "Intelligent Query Generation", description: "**Gemini-Powered Queries:**  Leverages the power of Gemini LLMs to generate smart, targeted search queries, adapting to research goals and accumulated learnings for optimal information retrieval.")
      @feature(name: "Depth & Breadth Control", description: "**Tuneable Research Scope:** Offers highly configurable depth and breadth parameters, allowing users to precisely control the scope and intensity of research exploration, from focused investigations to broad surveys.")
      @feature(name: "Smart Follow-up Questions", description: "**Clarify Research Needs with Follow-up Questions:**  Intelligently generates follow-up questions to refine ambiguous user queries, ensuring the research agent is precisely aligned with the user's intended topic.")
      @feature(name: "Comprehensive Markdown Reports", description: "**Detailed, Ready-to-Use Markdown Reports:**  Generates well-structured, human-readable Markdown reports, summarizing key findings, insights, and providing a clear list of sources for verification and further exploration.")
      @feature(name: "Concurrent Processing for Speed", description: "**Efficient & Fast with Concurrent Processing:**  Maximizes research efficiency and speed by handling multiple searches and data analysis tasks in parallel, leveraging asynchronous operations.")
    }
     @note: "**Feature Highlights:** These key features are designed to make 'open-deep-research' a powerful, versatile, and efficient research tool, while maintaining a clear and understandable codebase."
  }

  @section(name: "Software and API Requirements", level: 1) {
    @requirements-section(description: "Software and API key requirements - ensure these are set up before running the agent") {
      @requirement(type: "software", name: "Node.js", version: "v22.x", description: "**Node.js v22.x or later:**  Recommended runtime environment for optimal performance and compatibility. Download from [nodejs.org](https://nodejs.org).")
      @requirement(type: "api-key", name: "Firecrawl API Key", description: "**Firecrawl API Key:** Required for accessing Firecrawl's web search and content extraction capabilities.  Sign up for a free or paid plan at [mendable.ai](https://mendable.ai).")
      @requirement(type: "api-key", name: "Gemini API Key", description: "**Gemini API Key:**  Essential for accessing Gemini LLMs (o3 mini model recommended). Obtain your free API key from [makersuite.google.com](https://makersuite.google.com).")
      @requirement(type: "env-variable", name: "GEMINI_API_KEY", location: ".env.local", description: "`GEMINI_API_KEY` environment variable in `.env.local` file:  **Must be set to your Gemini API key.**  Crucial for Gemini API access.")
      @requirement(type: "env-variable", name: "FIRECRAWL_API_KEY", location: ".env.local", description: "`FIRECRAWL_API_KEY` environment variable in `.env.local` file: **Must be set to your Firecrawl API key.**  Essential for web search functionality.")
      @requirement(type: "env-variable", name: "FIRECRAWL_BASE_URL", location: ".env.local (optional)", description: "`FIRECRAWL_BASE_URL` environment variable (optional) in `.env.local`: **Only required if you intend to use a self-hosted Firecrawl instance.** Leave this variable unset to use Mendable's default hosted Firecrawl service.")
    }
     @note: "**API Key Management:**  Storing API keys as environment variables in `.env.local` is a best practice for security and configuration management. **Never commit your `.env.local` file to version control.**"
     @example(title: "Example .env.local file:", content: "```env\nGEMINI_API_KEY=YOUR_GEMINI_API_KEY_HERE\nFIRECRAWL_API_KEY=YOUR_FIRECRAWL_API_KEY_HERE\n# FIRECRAWL_BASE_URL=http://localhost:3002 (Optional - for self-hosted Firecrawl)\n```")
  }

  @section(name: "Setup and Installation Instructions", level: 1) {
    @setup-section(description: "Step-by-step instructions to set up and install open-deep-research - get started quickly") {
      @setup-step(stepNumber: 1, description: "**Clone the Repository:** Clone the 'open-deep-research' repository to your local machine using Git: `git clone [your-repo-link-here]` (replace '[your-repo-link-here]' with the actual repository URL from GitHub or your hosting provider).")
      @setup-step(stepNumber: 2, description: "**Install Dependencies (npm install):** Navigate to the newly cloned 'open-deep-research' directory in your terminal and run the command `npm install`. This will install all necessary Node.js packages listed in `package.json`.")
      @setup-step(stepNumber: 3, description: "**Configure API Keys in `.env.local`:** Create a file named `.env.local` in the project root directory.  Open `.env.local` in a text editor and add your API keys, following the example in the 'Software and API Requirements' section. **Remember to replace the placeholder values with your actual API keys.**")
      @setup-step(stepNumber: 4, description: "**Build the Project (npm run build):** Compile the TypeScript code to JavaScript by running the command `npm run build` in your terminal. This will generate the compiled JavaScript files in the 'dist' directory, ready for execution.")
    }
    @note: "**Quick Setup Tip:**  Ensure you have Node.js v22.x or later installed before starting the setup process. Double-check that your API keys are correctly placed in the `.env.local` file - incorrect API keys are a common source of errors."
  }

  @section(name: "Usage Instructions", level: 1) {
    @usage-section(description: "Detailed instructions on how to use open-deep-research in different modes - MCP Tool and Standalone CLI") {
      @section(name: "As MCP Tool", level: 2) {
        @usage-method(name: "As MCP Tool", description: "**Integrate deep-research as a Model Context Protocol (MCP) tool into AI agents.**  Run the MCP server to expose the research functionality to other MCP-compatible agents and tools.") {
          @command(name: "Start MCP Server", commandLine: "node --env-file .env.local dist/mcp-server.js", description: "**Start the MCP Server:** Run this command in your terminal from the project root to launch the MCP server, making the 'deep-research' tool accessible via MCP.")
          @tool-parameters {
            @parameter(name: "query", type: "string", description: "**query (string) - Required:** The central research query string that the deep research agent will investigate.  This is the primary input for the research tool.")
            @parameter(name: "depth", type: "number", range: "1-5", description: "**depth (number, 1-5) - Optional:**  Controls the research depth, determining the number of recursive iterations.  Defaults to a moderate depth if not specified.")
            @parameter(name: "breadth", type: "number", range: "1-5", description: "**breadth (number, 1-5) - Optional:**  Sets the research breadth, influencing the scope of each research level. Defaults to a moderate breadth if not specified.")
            @parameter(name: "existingLearnings", type: "string[]", optional: "true", description: "**existingLearnings (string[], optional):**  An optional array of pre-existing research findings (strings).  Provide existing knowledge to guide and focus the research process, allowing the agent to build upon prior context.")
          }
          @tool-returns {
            @return-value(type: "markdown", description: "**content (markdown):**  Returns a comprehensive research report in Markdown format. The report summarizes key findings, insights, and includes a detailed list of sources for verification.")
            @return-value(type: "list<&lt;string>", name: "sources", description: "**sources (list<&lt;string>):**  An array of URLs representing all the web sources utilized by the agent during its research. Provides a clear and traceable list of information sources.")
            @return-value(type: "metadata", description: "**metadata (object):**  Returns a metadata object containing structured information about the research process, including 'learnings' (key insights extracted by the agent) and 'visitedUrls' (a complete list of all URLs visited during research), facilitating further analysis and integration into agent workflows.")
          }
          @usage-example(language: "typescript", code: "const result = await mcp.invoke(\"deep-research\", { query: \"What are the latest developments in quantum computing?\", depth: 3, breadth: 3 });")
          @usage-example(language: "typescript", code: "// Accessing report, sources, metadata from 'result' object:\n// console.log(\"Report:\\n\", result.content[0].text); // Access the Markdown report text\n// console.log(\"Sources:\", result.metadata.sources);     // Access the array of source URLs\n// console.log(\"Learnings Metadata:\", result.metadata.learnings); // Access the metadata object containing learnings")
        }
        @example(title: "MCP Tool Invocation Example (Typescript):", content: "```typescript\nconst mcp = new ModelContextProtocolClient(); // Assuming MCP client is initialized\n\nasync function invokeDeepResearchTool() {\n  try {\n    const result = await mcp.invoke(\"deep-research\", {\n      query: \"Explain the principles of blockchain technology\",\n      depth: 2,\n      breadth: 4\n    });\n\n    if (result.isError) {\n      console.error(\"MCP Tool Error:\", result.content[0].text);\n    } else {\n      console.log(\"Research Report:\\n\", result.content[0].text);\n      console.log(\"Sources:\\n\", result.metadata.sources);\n    }\n  } catch (error) {\n    console.error(\"MCP Invoke Error:\", error);\n  }\n}\n\ninvokeDeepResearchTool();\n```")
      }

      @section(name: "Standalone Usage (CLI)", level: 2) {
        @usage-method(name: "Standalone Usage (CLI)", description: "**Run deep-research directly from your command line** for quick research and report generation, ideal for individual research tasks and explorations.") {
          @command(name: "Run Standalone CLI", commandLine: "npm run start \"your research query\"", description: "**Execute Standalone Research via CLI:**  Run this command in your terminal from the project root. Replace '\"your research query\"' with the specific topic you want to research, enclosed in double quotes if it contains spaces.")
          @example-command(name: "Example Command", commandLine: "`npm run start \"what are latest developments in ai research agents\"`")
          @note: "**Command Line Input:** The research query is passed as a command-line argument after `npm run start`. Ensure to enclose queries with spaces in double quotes to be parsed correctly by the script."
        }
      }

      @section(name: "MCP Inspector Testing", level: 2) {
        @usage-method(name: "MCP Inspector Testing", description: "**Utilize the Model Context Protocol Inspector for interactive testing and debugging of the MCP server.**  The Inspector provides a visual interface to send commands to the 'deep-research' MCP tool and examine responses in detail.") {
          @command(name: "Run MCP Inspector", commandLine: "npx @modelcontextprotocol/inspector node --env-file .env.local dist/mcp-server.js", description: "**Launch MCP Inspector Connected to deep-research:** Execute this command to start the MCP Inspector, automatically connecting it to your local 'deep-research' MCP server for interactive testing and command execution.")
          @note: "**Interactive Testing & Debugging:** The MCP Inspector is invaluable for testing the 'deep-research' MCP tool during development. It allows you to send commands, inspect request/response payloads, and verify the tool's behavior in a controlled environment."
        }
      }
    }
  }

  @section(name: "License Information", level: 1) {
    @license-section(description: "License information - open-deep-research is MIT licensed - use it freely!") {
      @license-type: "MIT License"
      @license-text: "**MIT License - Free and Open Source:**  'open-deep-research' is released under the permissive MIT License, encouraging open use, modification, and distribution for both commercial and non-commercial applications.  Attribution is appreciated but not required.  See the full LICENSE file in the repository for complete legal details."
    }
  }

  @section(name: "Project File Structure", level: 1) {
    @project-structure-section(description: "Detailed file and directory structure of the open-deep-research project - understand the codebase organization") {
      @directory(name: "src", description: "**src/: Source Code Root Directory:** Contains all the primary TypeScript source code files for the 'open-deep-research' agent and its MCP server component.") {
        @directory(name: "ai", description: "**src/ai/: AI Provider Implementations:**  Dedicated directory for AI provider-specific code. Currently focuses on Gemini API integration, but can be extended to support other LLMs in the future.") {
          @file(name: "providers.ts", type: "typescript", description: "**src/ai/providers.ts:**  Defines interfaces and concrete implementations for interacting with AI providers like Gemini. Includes API client initialization, model selection, and utility functions for prompt management and handling API responses.")
        }
        @file(name: "deep-research.ts", type: "typescript", description: "**src/deep-research.ts (Core Research Logic):**  The heart of the 'open-deep-research' agent. Implements the main 'deepResearch' function, which orchestrates the iterative research workflow, including query generation, search execution, result processing, and report assembly.")
        @file(name: "feedback.ts", type: "typescript", description: "**src/feedback.ts:**  Handles user query refinement and feedback mechanisms. Contains the 'generateFeedback' function, which uses Gemini to create follow-up questions for clarifying research goals.")
        @file(name: "mcp-server.ts", type: "typescript", description: "**src/mcp-server.ts (MCP Server Entry Point):**  The entry point for the Model Context Protocol (MCP) server. Sets up and starts the MCP server, exposing the 'deep-research' functionality as an MCP-compatible tool for integration with other AI agents.")
        @file(name: "output-manager.ts", type: "typescript", description: "**src/output-manager.ts:**  Manages the command-line interface output, providing consistent logging, progress updates, and a clean user experience in the terminal.")
        @file(name: "progress-manager.ts", type: "typescript", description: "**src/progress-manager.ts (Alternative Progress Display):**  An alternative implementation for managing and rendering progress updates in the CLI, utilizing progress bars and ANSI escape codes for visual feedback (can be swapped with 'output-manager.ts').")
        @file(name: "prompt.ts", type: "typescript", description: "**src/prompt.ts:**  Centralizes prompt management for Gemini LLM interactions. Defines the main 'systemPrompt' used to instruct Gemini and potentially other specialized prompts for different research tasks.")
        @file(name: "run.ts", type: "typescript", description: "**src/run.ts (Standalone CLI Script):**  Provides the command-line interface entry point for running the 'open-deep-research' agent directly from the terminal, without requiring the MCP server.  Used for standalone research execution.")
        @file(name: "text-splitter.ts", type: "typescript", description: "**src/text-splitter.ts (Text Chunking Logic):**  Implements text splitting functionalities, particularly the 'RecursiveCharacterTextSplitter', used to divide long text inputs and outputs into manageable chunks for LLM processing and context window limitations.")
        @file(name: "text-splitter.test.ts", type: "typescript", description: "**src/text-splitter.test.ts:**  Contains unit tests specifically designed to verify the functionality and correctness of the text splitting logic implemented in 'text-splitter.ts', ensuring code reliability.")
      }
      @directory(name: "dist", description: "**dist/: Compiled JavaScript Output Directory:**  Output directory for the compiled JavaScript code.  Generated automatically when you build the TypeScript project using `npm run build`. **Do not modify files in this directory directly.**")
      @directory(name: "config", description: "**config/: Configuration Files Directory:**  Holds project configuration files. Currently contains `.env.local` for storing local environment variables and API keys securely.")
      @directory(name: "docs", description: "**docs/: Documentation Directory (Optional):**  A placeholder directory intended for storing additional project documentation files (currently empty in the base project). You can add design documents, research notes, or extended documentation here.")
      @directory(name: "utils", description: "**utils/: Utility Functions & Helpers Directory:**  Houses general utility functions and helper modules that are used across different parts of the 'open-deep-research' project, promoting code reuse and modularity.")
      @file(name: "index.ts", type: "typescript", description: "**index.ts (src/index.ts):**  The main entry point for the 'src' directory.  Currently used for exporting modules and potentially for project-wide initialization in the future.")
      @file(name: "package.json", type: "json", description: "**package.json (Project Manifest):**  Defines project metadata, dependencies (libraries used), and scripts (npm commands) for building, running, and testing the 'open-deep-research' application.  Essential for Node.js project management.")
      @file(name: "tsconfig.json", type: "json", description: "**tsconfig.json (TypeScript Configuration):**  Specifies the configuration options for the TypeScript compiler, guiding how TypeScript code is compiled into JavaScript.  Customizes compiler behavior and output settings.")
      @file(name: ".env.local", type: "env", description: "`.env.local` (Environment Variables File):**  Stores local environment variables, primarily API keys (Gemini and Firecrawl), and other configuration settings that are specific to your development environment. **Crucially, this file is excluded from version control to protect sensitive API keys.**")
      @file(name: "README.md", type: "markdown", description: "**README.md (Project Overview & Instructions):**  The main README file for the 'open-deep-research' project. Provides a high-level overview of the project, its features, setup instructions, usage examples, and licensing information.  Serves as the primary entry point for users and developers to understand the project.")
      @file(name: "Dockerfile", type: "dockerfile", description: "**Dockerfile (Docker Containerization):**  Defines the Dockerfile used to containerize the 'open-deep-research' application.  Enables easy deployment, consistent runtime environments, and simplified distribution of the research agent.")
    }
  }

  // -------------------- Code Snippets --------------------

  @code-snippet-collection(description: "Reusable code snippets for open-deep-research project - Accelerate your coding workflow") {

    @snippet-category(name: "Gemini API Interactions (open-deep-research)") {
      @snippet(name: "Gemini Prompt Call (GeminiPro) - Configurable Model & Settings", description: "**Use this snippet to quickly call the GeminiPro model** with a prompt, and configure the model name, temperature, and safety settings. Ideal for testing different Gemini models and response variations.") {
        @code(language: "typescript") {
          ```typescript
          import { GeminiPro } from '@modelcontextprotocol/sdk'; // MCP SDK import

          async function callGeminiProConfigurable(prompt: string, modelName: string = "gemini-pro", temperature?: number, safetySettings?: any): Promise<string> {
            const model = new GeminiPro({
              apiKey: process.env.GEMINI_API_KEY,
              modelName: modelName, // Configurable model name
              temperature: temperature, // Optional temperature
              safetySettings: safetySettings // Optional safety settings
            });
            try {
              const response = await model.generateContent(prompt);
              return response.text();
            } catch (error) {
              console.error("GeminiPro API Error:", error);
              return "";
            }
          }

          // Example usage: (showing different model and temperature)
          // // 1. Basic call with default 'gemini-pro' model:
          // const geminiResponseDefault = await callGeminiProConfigurable("Your research prompt");
          // console.log("Default Gemini Response:", geminiResponseDefault);

          // // 2. Call with 'gemini-pro-vision' model and temperature 0.7:
          // const geminiResponseVision = await callGeminiProConfigurable("Analyze this image and describe it.", "gemini-pro-vision", 0.7);
          // console.log("Vision Model Response:", geminiResponseVision);

          // // 3. You can also adjust safetySettings if needed (advanced):
          // const geminiResponseSafe = await callGeminiProConfigurable("Potentially sensitive prompt.", "gemini-pro", undefined, { // ... safetySettings object ... });
          // console.log("Response with Safety Settings:", geminiResponseSafe);
          ```
        }
        @usage-notes {
          "Uses GeminiPro from '@modelcontextprotocol/sdk'.  Allows configuring 'modelName' (default: 'gemini-pro'), 'temperature', and 'safetySettings'.  Remember to set 'GEMINI_API_KEY'."
        }
        @tags {
          @tag: "Gemini"
          @tag: "GeminiPro"
          @tag: "API Call"
          @tag: "Prompt"
          @tag: "LLM"
          @tag: "Model Configuration"
          @tag: "Temperature"
          @tag: "Safety Settings"
        }
        @function-signature {
          @parameter-doc(name: "prompt", type: "string", description: "The prompt string to send to Gemini")
          @parameter-doc(name: "modelName", type: "string", optional: "true", default: "'gemini-pro'", description: "Optional Gemini model name (e.g., 'gemini-pro', 'gemini-pro-vision')")
          @parameter-doc(name: "temperature", type: "number", optional: "true", description: "Optional temperature setting (0-1)")
          @parameter-doc(name: "safetySettings", type: "any", optional: "true", description: "Optional safetySettings object for Gemini API")
          @return-type-doc: "Promise<string>"
          @return-description: "A Promise that resolves with the Gemini API response text (string)"
        }
        @see-section(name: "Software and API Requirements", description: "See the 'Software and API Requirements' section for information about setting up the Gemini API key environment variable.")
        @see-section(name: "Usage Instructions", description: "See the 'Usage Instructions' section for examples of how Gemini is used in the project.")
      }

      @snippet(name: "Gemini Prompt Call (Raw @google/generative-ai)", description: "**Use this snippet to directly call the raw @google/generative-ai library.** Provides a basic Gemini API call using the raw library for maximum control and flexibility.") {
        @code(language: "typescript") {
          ```typescript
          import { GoogleGenerativeAI } from "@google/generative-ai"; // Raw Gemini library

          async function callRawGeminiAPI(prompt: string): Promise<string> {
            const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
            const model = genAI.getGenerativeModel({ model: "gemini-pro" }); // Or specify model
            const result = await model.generateContent(prompt);
            const response = result.response;
            return response.text();
          }

          // Example usage:
          // const geminiResponse = await callRawGeminiAPI("Your research prompt here");
          // console.log("Gemini Response:", geminiResponse);
          ```
        }
        @usage-notes {
          "Uses raw '@google/generative-ai' library.  Requires 'GEMINI_API_KEY' and '@google/generative-ai' dependency. Choose appropriate Gemini model if needed. Useful when you need fine-grained control over the Gemini API."
        }
        @tags {
          @tag: "Gemini"
          @tag: "Raw API"
          @tag: "@google/generative-ai"
          @tag: "Prompt"
          @tag: "LLM"
        }
        @function-signature {
          @parameter-doc(name: "prompt", type: "string", description: "The prompt string to send to Gemini")
          @return-type-doc: "Promise<string>"
          @return-description: "A Promise that resolves with the Gemini API response text (string)"
        }
        @see-section(name: "Software and API Requirements", description: "See the 'Software and API Requirements' section for information about setting up the Gemini API key environment variable.")
        @see-section(name: "Usage Instructions", description: "See the 'Usage Instructions' section for examples of how Gemini is used in the project.")
      }
    }

    @snippet-category(name: "SerpAPI Queries (open-deep-research)") {
      @snippet(name: "SerpAPI Google Search - Configurable Region/Language", description: "**Use this snippet to perform a Google Search using SerpAPI,** with options to configure geo-location and host language for localized results.  Tailor search parameters for specific regional or linguistic research needs.") {
        @code(language: "typescript") {
          ```typescript
          import SerpApi from 'serpapi';

          async function performGoogleSearchLocalized(query: string, geoLocation?: string, hostLanguage?: string): Promise<any> {
            const serpApiClient = new SerpApi.GoogleSearch(process.env.SERPAPI_API_KEY);
            const params = {
              q: query,
              engine: "google",
              api_key: process.env.SERPAPI_API_KEY,
              gl: geoLocation,   // Optional geo-location (e.g., "US", "GB")
              hl: hostLanguage    // Optional host-language (e.g., "en", "es")
              // Add other parameters as needed (e.g., num results, date range, etc.)
            };

            try {
              const response = await serpApiClient.json(params);
              return response;
            } catch (error) {
              console.error("SerpAPI Google Search Error:", error);
              return null;
            }
          }

          // Example usage: (showing localized search and query placeholder)
          // // 1. Basic Google Search - remember to replace '{{searchQuery}}' placeholder:
          // const basicSearchResults = await performGoogleSearchLocalized("replace with your query"); // Replace '{{searchQuery}}'

          // // 2. Localized search in Great Britain, English language:
          // const localizedResultsGB = await performGoogleSearchLocalized("local news london", "GB", "en");
          // console.log("Localized Results (GB):", localizedResultsGB.organic_results);
          ```
        }
        @usage-notes {
          "Uses 'serpapi' library.  **Configure 'geoLocation' (gl) and 'hostLanguage' (hl) parameters for localized search results.** Shows example of accessing 'organic_results'.  Remember to set 'SERPAPI_API_KEY'. Customize search parameters in 'params' as needed to refine search scope and results."
        }
        @tags {
          @tag: "SerpAPI"
          @tag: "Google Search"
          @tag: "Search"
          @tag: "Query"
          @tag: "Localized Search"
          @tag: "Region"
          @tag: "Language"
        }
        @function-signature {
          @parameter-doc(name: "query", type: "string", description: "The search query string for SerpAPI (used as cache key)")
          @parameter-doc(name: "geoLocation", type: "string", optional: "true", description: "Optional geo-location parameter (e.g., 'US', 'GB')")
          @parameter-doc(name: "hostLanguage", type: "string", optional: "true", description: "Optional host-language parameter (e.g., 'en', 'es')")
          @return-type-doc: "Promise<any>"
          @return-description: "A Promise that resolves with the SerpAPI JSON response object (any), either from cache or from a new API call."
        }
        @see-section(name: "Software and API Requirements", description: "See the 'Software and API Requirements' section for information about setting up the SERPAPI_API_KEY environment variable.")
        @see-section(name: "Usage Instructions", description: "See the 'Usage Instructions' section for examples of how SerpAPI is used in the project.")
      }
    }

    @snippet-category(name: "Text Processing (open-deep-research)") {
      @snippet(name: "Chunk Text with Tiktoken (cl100k_base)", description: "**Use this function to chunk long text into smaller segments using the cl100k_base Tiktoken encoder.**  Helpful for preparing text for language models with context length limits.  Adjust chunk size precisely using the 'maxTokens' parameter.") {
        @code(language: "typescript") {
          ```typescript
          import { Tiktoken, TiktokenModel } from "js-tiktoken";

          function chunkTextWithTiktoken(text: string, maxTokens: number = 140): string[] {
            const encoder = new Tiktoken(TiktokenModel.CL100K_BASE);
            const tokens = encoder.encode(text);
            const chunks: string[] = [];
            let currentChunkTokens: number[] = [];

            for (const token of tokens) {
              if (currentChunkTokens.length + 1 > maxTokens) {
                chunks.push(encoder.decode(currentChunkTokens));
                currentChunkTokens = [token];
              } else {
                currentChunkTokens.push(token);
              }
            }
            if (currentChunkTokens.length > 0) {
              chunks.push(encoder.decode(currentChunkTokens));
            }
            encoder.free();
            return chunks;
          }

          // Example usage: (showing different chunk sizes and iteration)
          // // 1. Basic chunking into ~140 token segments (default - for Gemini Pro):
          // const longText = "Your long research paper content here...";
          // const textChunks = chunkTextWithTiktoken(longText);
          // console.log("Text Chunks (default size ~140 tokens):", textChunks);

          // // 2. Chunking into smaller ~50 token segments (e.g., for smaller models):
          // const smallerChunks = chunkTextWithTiktoken("Another long text", 50);
          // console.log("Smaller Chunks (50 tokens):", smallerChunks);

          // // 3. Process each chunk individually (common workflow):
          // textChunks.forEach((chunk, index) => {
          //   console.log(`Chunk ${index + 1} (Token Count: ${encoder.encode(chunk).length}):`, chunk.substring(0, 50) + "..."); // Show first 50 chars
          //   // ... Process each chunk (e.g., send to Gemini API for analysis) ...
          // });
          ```
        }
        @usage-notes {
          "Uses 'js-tiktoken' library with 'cl100k_base' encoder (optimal for models like OpenAI's and potentially suitable for Gemini context). **Precisely control chunk size by adjusting the 'maxTokens' parameter.** Ensure 'js-tiktoken' library is installed in your project."
        }
        @tags {
          @tag: "Text Processing"
          @tag: "Chunking"
          @tag: "Tiktoken"
          @tag: "cl100k_base"
          @tag: "Tokenization"
          @tag: "Context Limit"
        }
        @function-signature {
          @parameter-doc(name: "text", type: "string", description: "The long text string that needs to be split into smaller chunks")
          @parameter-doc(name: "maxTokens", type: "number", optional: "true", default: "140", description: "Maximum number of tokens allowed per chunk (default: 140, suitable for Gemini Pro)")
          @return-type-doc: "string[]"
          @return-description: "An array of text chunks (string[]), each chunk containing approximately 'maxTokens' tokens or fewer."
        }
        @see-section(name: "Project File Structure", description: "Relates to 'text-splitter.ts' module in the project structure, which provides more advanced text splitting capabilities if needed.")
      }
    }

    @snippet-category(name: "CursorRules & Data Handling (open-deep-research)") {
      @snippet(name: "Load CursorRules Data from JSON File", description: "**Use this function to load CursorRulesData from a JSON file efficiently.** Provides robust error handling and gracefully returns an empty structure if file loading fails, preventing application crashes.") {
        @code(language: "typescript") {
          ```typescript
          import * as fs from 'fs/promises';
          import { CursorRulesData } from './src/data/cursorRulesInterfaces'; // Adjust path to your interfaces

          async function loadCursorRulesData(filePath: string): Promise<CursorRulesData> {
            try {
              const data = await fs.readFile(filePath, 'utf-8');
              return JSON.parse(data) as CursorRulesData;
            } catch (error) {
              console.error("Error loading CursorRulesData from JSON:", error);
              return { branches: [] }; // Gracefully return empty data structure on error
            }
          }

          // Example usage: (loading from a file named 'research_data.json' in the project root)
          // const cursorRules = await loadCursorRulesData('./research_data.json');
          // console.log("CursorRules Data:", cursorRules);
          ```
        }
        @usage-notes {
          "Adjust the import path for 'CursorRulesData' to match your project structure. **Customize error handling within the 'catch' block** to suit your specific application needs (e.g., throw error, retry loading, use default data).  Ensure to provide the correct 'filePath' to your CursorRules JSON data file."
        }
        @tags {
          @tag: "CursorRules"
          @tag: "Data Handling"
          @tag: "JSON"
          @tag: "File Loading"
          @tag: "Data Persistence"
          @tag: "Error Handling"
        }
        @function-signature {
          @parameter-doc(name: "filePath", type: "string", description: "The file path to the CursorRules JSON data file that you want to load.")
          @return-type-doc: "Promise<CursorRulesData>"
          @return-description: "A Promise that resolves with the loaded CursorRulesData object if successful, or an empty CursorRulesData object if loading fails (error handling)."
        }
        @see-section(name: "Project File Structure", description: "Relates to 'cursorRulesService.ts' (if you implement a service) and 'cursorRulesInterfaces.ts' in the 'data' directory, which define how CursorRules data is structured and managed.")
      }

      @snippet(name: "Create a basic Thought object", description: "**Use this template to quickly instantiate a new Thought object** with essential properties pre-set.  Customize the 'type', 'content', and 'confidence' to represent different stages and qualities of research insights.") {
        @code(language: "typescript") {
          ```typescript
          import { v4 as uuidv4 } from 'uuid';
          import { Thought } from './src/data/cursorRulesInterfaces'; // Adjust path

          function createThought(type: string, content: string): Thought {
            return {
              thoughtId: uuidv4(),
              timestamp: new Date().toISOString(),
              type: type,
              content: content,
              confidence: 0.75, // Default confidence - adjust as needed for thought quality
              keyPoints: [],     // Initially empty, populate key points during analysis
              relatedInsights: [], // Links to related insights (initially empty)
              crossRefs: [],       // Cross-references to other branches (initially empty)
              metadata: {}        // Metadata object for extensible data (initially empty)
            };
          }

          // Example usage: (creating an 'analysis' thought with a descriptive content string)
          // const newThought = createThought('analysis', 'Initial analysis thought content: Synthesizing findings from SERP results and Gemini response.');
          // console.log("New Thought object created:", newThought);
          ```
        }
        @usage-notes {
          "Adjust import path for 'Thought'.  **Customize the default 'confidence' value** to reflect the assessed reliability or certainty of the thought.  Populate optional fields like 'keyPoints', 'metadata', 'relatedInsights', and 'crossRefs' to add more detail and connections to the Thought object as your research progresses. Uses 'uuidv4()' for generating unique 'thoughtId' values - ensure 'uuid' package is installed."
        }
        @tags {
          @tag: "CursorRules"
          @tag: "Data Handling"
          @tag: "Thought"
          @tag: "Object Creation"
          @tag: "Data Structure"
          @tag: "Research Data"
        }
        @function-signature {
          @parameter-doc(name: "type", type: "string", description: "The 'type' of the Thought.  Categorize thoughts (e.g., 'search_query', 'serp_result', 'analysis', 'insight') for structured research tracking.")
          @parameter-doc(name: "content", type: "string", description: "The 'content' of the Thought.  The main textual information or observation captured in this Thought object.")
          @return-type-doc: "Thought"
          @return-description: "Returns a newly created 'Thought' object, ready to be added to your CursorRules data structure."
        }
        @see-section(name: "Project File Structure", description: "Relates to 'cursorRulesInterfaces.ts' in the 'data' directory, where the 'Thought' interface and other CursorRules data structures are defined.")
      }
    }

    @snippet-category(name: "Error Handling & Logging (open-deep-research)") {
      @snippet(name: "Basic Try-Catch Block", description: "**Insert a basic try-catch block** for error handling.  Use this template to wrap code that might throw errors and implement error handling logic in the 'catch' block.") {
        @code(language: "typescript") {
          ```typescript
          try {
            // Code that might throw an error
            // ... your code that interacts with APIs, file system, etc. ...
          } catch (error) {
            console.error("An error occurred:", error);
            // Handle the error (e.g., log, display message, retry, etc.)
          }
          ```
        }
        @usage-notes {
          "Basic try-catch block template. **Replace '// ... your code ...' with the code you want to protect.**  Implement specific error handling strategies within the 'catch' block, such as logging errors, providing user feedback, or attempting recovery actions. Consider using 'output.log' for consistent error logging."
        }
        @tags {
          @tag: "Error Handling"
          @tag: "Try-Catch"
          @tag: "Exception Handling"
          @tag: "Code Structure"
          @tag: "Robustness"
        }
      }

      @snippet(name: "Logging with output.log", description: "**Use 'output.log' for consistent and structured logging in your application.** Ensures all log messages are handled by the OutputManager, preserving progress display and providing a unified logging interface.") {
        @code(language: "typescript") {
          ```typescript
          import { OutputManager } from './src/output-manager'; // Adjust path

          const output = new OutputManager(); // Assuming you have OutputManager initialized

          output.log("Starting a new research process...", { timestamp: new Date().toISOString(), query: "user's research query" });
          // ... your code performing research ...
          output.log("Process completed successfully.", { resultsCount: 15, timeTaken: "2 minutes" });
          output.log("Error encountered during data processing:", { errorDetails: error }); // Example error logging
          ```
        }
        @usage-notes {
          "Uses 'output.log' for consistent logging through your OutputManager. **Remember to import and initialize 'OutputManager'** in your module if you haven't already. Include relevant context and details in your log messages (e.g., timestamps, process names, variable values) to aid in debugging and monitoring."
        }
        @tags {
          @tag: "Logging"
          @tag: "OutputManager"
          @tag: "CLI Output"
          @tag: "Progress Tracking"
          @tag: "Debugging"
        }
      }
    }

    @snippet-category(name: "Data Processing & Manipulation (open-deep-research)") {
      @snippet(name: "Filter Array of Objects by Property", description: "**Efficiently filter arrays of objects based on specific property values** using JavaScript's built-in 'filter' array method. Adapt the interface, array variable, property name, and filtering condition to your data structures.") {
        @code(language: "typescript") {
          ```typescript
          interface MyObject { // Define or import your object interface
            name: string;
            value: number;
            status?: string;
            // ... other properties ...
          }

          const myArray: MyObject[] = [ // Replace 'myArray' with your actual array variable
            // ... your array of MyObject instances ...
          ];

          const filteredArray = myArray.filter(item => item.value > 10 && item.status !== 'pending'); // Filter: value > 10 AND status is not 'pending' - **customize condition**

          console.log("Filtered Array (value > 10 and not pending):", filteredArray);
          ```
        }
        @usage-notes {
          "Leverage JavaScript's native 'filter' method for array filtering. **Adapt the 'MyObject' interface to match your object structure.**  Replace 'myArray' with the actual array variable you want to filter. **Customize the filter condition** (e.g., 'item.value > 10 && item.status !== 'pending'') within the 'filter' method to implement your desired filtering logic. You can combine multiple conditions using '&&' (AND) and '||' (OR) operators."
        }
        @tags {
          @tag: "Data Processing"
          @tag: "Array"
          @tag: "Object"
          @tag: "Filtering"
          @tag: "JavaScript Array Methods"
          @tag: "Efficiency"
        }
      }

      @snippet(name: "Map Array of Objects to Property Array", description: "**Transform arrays of objects into simpler arrays containing only a specific property** using the efficient 'map' array method.  Useful for extracting a specific field from a collection of objects.") {
        @code(language: "typescript") {
          ```typescript
          interface MyObject { // Define or import your object interface
            name: string;
            value: number;
            description?: string;
            // ... other properties ...
          }

          const myArray: MyObject[] = [ // Replace 'myArray' with your data array
            // ... your array of MyObject instances ...
          ];

          const namesArray: string[] = myArray.map(item => item.name); // Extract 'name' property - **customize property name to extract**

          console.log("Extracted Names Array:", namesArray); // 'namesArray' now contains only the 'name' property from each object
          ```
        }
        @usage-notes {
          "Maps an array of objects ('myArray') to a new array containing only a specific property ('name' in this example). **Adapt the interface 'MyObject', array 'myArray', and customize the 'item.name'** within the 'map' method to extract the property you need. This creates a new array containing only the values of the chosen property."
        }
        @tags {
          @tag: "Data Processing"
          @tag: "Array"
          @tag: "Object"
          @tag: "Mapping"
          @tag: "Transformation"
          @tag: "JavaScript Array Methods"
          @tag: "Efficiency"
        }
      }
    }

  } // End of @code-snippet-collection

  @section(name: "Future Development Tools & Agent Enhancements", level: 1) { // (Same Future Tools Section as before) }

  @section(name: "LRU-Cache Implementation Ideas & Snippets", level: 1) { // (LRU Cache Section - No Changes) }

} // End of @project-documentation

---
> Source: [ssdeanx/deep-research-mcp-server](https://github.com/ssdeanx/deep-research-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
