## cypher

> Handles **command parsing**, **execution logic**, and **command registry**.

1. **Project Structure**:
   - `src/agents`:  
     Contains **agent YAML configuration files** (e.g., `myAgent.yaml`, `terminalAgent.yaml`) defining agent behavior, personality, and model configuration.
   - `src/models`:  
     Houses **model clients** (OpenAI, Anthropic, Fireworks) and **adapters** to normalize inputs/outputs.
   - `src/features`:  
     Contains **feature modules** that add new terminal commands (tools) to the environment for the agent.
   - `src/terminal`:  
     Handles **command parsing**, **execution logic**, and **command registry**.
   - `terminalCore.ts`:  
     Manages the main loop running an agent continuously and applying features, enabling **autonomous operation**.
   - `src/gui`:  
     Contains GUI components like `loggerGUI.html` for **real-time monitoring** and logging of agent activity.
   - `config/personality.yaml`:  
     Defines **global personality traits** and dynamic variables shared across agents.
   - `index.ts`:  
     Entry point for initializing and running the system, including the Logger Server and `TerminalCore`.

2. **YAML-Based Agent Configuration**:
   Agents are defined entirely in **YAML** within `src/agents/` (e.g., `myAgent.yaml`):
   - **Agent name & description**
   - **Model/client selection** (OpenAI, Anthropic, Fireworks)
   - **System prompt** with dynamic variables & references to `personality.yaml`
   - **Main goals & dynamic variables**
   - **Tools (function calling)** and/or **structured output schemas**

   Reference global variables using ```{{from_personality:core_personality}}```, etc.

3. **Global Personality and Dynamic Variables**:
   - `config/personality.yaml` holds shared traits/variables.
   - Agents can reuse these variables to maintain consistent personality.

4. **BaseAgent**:
   - Manages **chat history** and communicates with **model clients**.
   - Uses YAML config for personality, tools, and schemas.
   - Supports **Function Calling (Tools)** and **Structured Outputs**.
   - No need to hard-code agent configurations—just edit YAML.

5. **Model Adapters & Clients**:
   - **Adapters** standardize interactions with different models.
   - **Clients** connect to OpenAI, Anthropic, Fireworks, etc.
   - Easily switch model backends without rewriting logic.

6. **Function Calling (Tools)**:
   When you define tools in the agent's YAML:
   ```yaml
   tools:
  - type: "function"
    function:
      name: "get_weather"
      description: "Get current weather for a location"
      parameters:
        type: object
        properties:
          location:
            type: string
            description: "City and state, e.g. San Francisco, CA"
        required: ["location"]
   ```
   
   - Agents may call these tools when they deem it appropriate.
   - The model returns a **function call** (tool call) with the required arguments.
   - **Your application** is responsible for executing this function call externally (e.g., calling a real weather API) and then feeding the results back to the agent by adding a message with ```role: "user"``` that contains the tool results. This enables the agent to incorporate real-world data.

   **Example Workflow**:
   1. Agent decides to call `get_weather` with `{"location":"San Francisco, CA"}`.
   2. Your application detects the function call in the agent's output, executes the external API call, and obtains the weather data.
   3. Your application then adds a `tool` role message with the results:
      ```typescript
      agent.addUserMessage({
        role: 'tool',
        content: JSON.stringify({ weather: "Sunny, 20°C" }),
        tool_call_id: "the_tool_call_id_returned_by_agent"
      });```

   4. You run the agent again. Now the agent sees the tool response and continues the conversation, giving the final answer to the user.

   **When to Use Tools**:  
   Use tools when you need the model to actively call external APIs or perform actions. Tools are best when the agent must fetch data or take action.

7. **Structured Outputs**:
   **Structured outputs** ensure the model responds in a strict JSON schema format without calling external tools. This is useful when:
   - You want to ensure a specific JSON shape to feed into a UI or another system.
   - You don’t need external API calls, just a well-structured response.

   **Example YAML**:
   ```yaml
   output_schema:
  type: object
  properties:
    answer:
      type: string
    confidence:
      type: number
  required: ["answer", "confidence"]
   ```

   With structured outputs, the agent returns strictly JSON that matches your schema. You can parse it directly:
   ```typescript
   const result = await agent.run("What's the capital of France?");
   if (result.success) {
   console.log("Parsed structured output:", result.output); // { answer: "Paris", confidence: 0.99 }
   }
   ```

   **When to Use Structured Outputs**:  
   Use structured outputs if you want the model’s reply in a defined format, like ```{"answer":"some_value", "confidence": number}```, without needing external function calls.

8. **Comparing Tools vs. Structured Outputs**:
   - **Tools (Function Calling)**:  
     Choose this if the model should dynamically call external functions or APIs. The agent might say: "I'm calling get_weather for 'San Francisco, CA'." You then execute that call in code and return the result.
   
   - **Structured Outputs**:  
     Choose this if you want a fixed schema output directly from the model. No external calls, just guaranteed format.

9. **TerminalCore & Features**:
   - `TerminalCore` runs the agent autonomously.
   - Add features by creating a `TerminalFeature` returning `Command[]`.
   - Register features in `TerminalCore` to provide additional commands.

10. **Integrating Tools in Your Application**:
    After the agent run:
    - Inspect the result for `functionCalls`.
    - For each function call, parse the arguments, call your external code (e.g., weather API).
    - Return the tool response as a `role: "user"` message.
    - Re-run the agent so it can integrate the tool results and respond to the user.

    Example pseudocode:
    ```typescript
    const result = await agent.run("What is the weather in Berlin?");
    if (result.success && result.outputIncludesToolCalls) {
      const toolCalls = result.functionCalls; 
      for (const call of toolCalls) {
    const toolName = call.functionName;
    const args = call.functionArgs; // e.g. {location: "Berlin"}
    
    // Execute the tool (your custom code):
    const toolResult = await callWeatherAPI(args.location);
    
    // Feed result back to agent:
    agent.addUserMessage({
      role: 'tool',
      content: JSON.stringify({ weather: toolResult }),
      tool_call_id: call.id
    });
  }
  
  // Now re-run the agent to get final user-facing output:
    const finalResult = await agent.run();
    console.log(finalResult.output);
    }```

11. **Integrating Structured Outputs in Your Application**:
    - If `output_schema` is defined, `agent.run()` returns structured JSON matching the schema.
    - Just parse and use it directly. No external calls needed.

12. **GUI & Logging**:
    - Use the GUI and logger to visualize agent decisions, tool calls, and outputs.
    - Adjust prompts, tools, and schemas based on behavior.

13. **Workflow & Iteration**:
    - Start with a base agent and personality.
    - Add tools if external data is needed, or an `output_schema` for structured responses.
    - Refine YAML configuration and observe results in GUI/logs.

14. **Extensibility & Modularity**:
    - YAML-based configs for agents & tools let you add/change capabilities without code changes.
    - Add features or tools by creating a new YAML or feature file.

15. **Running the System**:
    - Launch from `index.ts` to run tests.
    - Monitor with GUI.
    - Integrate with your application by handling function calls or parsing structured outputs.

---
> Source: [kingbootoshi/cypher](https://github.com/kingbootoshi/cypher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
