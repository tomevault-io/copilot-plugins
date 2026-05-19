## devcontext

> - **ALWAYS Use Context7 Before Using External Libraries**

# These rules MUST be followed ALWAYS without exception.

## RULE: You are an agent - please keep going until the user’s query is completely resolved, before ending your turn and yielding back to the user. Only terminate your turn when you are sure that the problem is solved.

## RULE: If you are not sure about file content or codebase structure pertaining to the user’s request, use your tools to read files and gather the relevant information: do NOT guess or make up an answer.

## RULE: You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls. DO NOT do this entire process by making function calls only, as this can impair your ability to solve the problem and think insightfully.

## RULE: External Library Documentation Requirements

- **ALWAYS Use Context7 Before Using External Libraries**

  - The agent MUST retrieve and review documentation via Context7 before implementing any code that uses an external library
  - This applies to ALL libraries not part of the standard language libraries
  - No exceptions - even for commonly known libraries like React, Express, or Lodash

- **Two-Step Documentation Retrieval Process**

  ```javascript
  // ✅ DO: ALWAYS follow this exact two-step process
  // Step 1: Resolve the library name to a Context7-compatible ID
  const libraryIdResponse =
    (await mcp_context7_resolve) -
    library -
    id({
      libraryName: "express",
    });

  // Step 2: Get the documentation using the resolved ID
  const docsResponse =
    (await mcp_context7_get) -
    library -
    docs({
      context7CompatibleLibraryID: libraryIdResponse.libraryId,
      tokens: 10000, // Adjust based on documentation needs
      topic: "routing", // Optional: focus on specific area
    });

  // ❌ DON'T: Skip the resolution step
  // ❌ DON'T: Use hardcoded library IDs
  // ❌ DON'T: Proceed with implementation without review
  ```

- **Never Skip Documentation Retrieval**

  - Documentation MUST be retrieved even for seemingly simple APIs
  - Do not rely on previously cached knowledge for current implementations
  - Never make assumptions about library interfaces, verify with current documentation

- **Document First, Implement Second**

  ```javascript
  // ✅ DO: Review documentation BEFORE writing implementation code
  // 1. Identify library need
  // 2. Retrieve documentation
  // 3. Review relevant sections
  // 4. THEN implement solution

  // ❌ DON'T: Implementation without documentation
  const app = express(); // WRONG - Documentation not retrieved first
  app.get("/", (req, res) => res.send("Hello"));
  ```

- **MUST Use Web Search When Documentation Is Unavailable**

  - If Context7 cannot provide documentation or returns insufficient information, the agent MUST use the web search tool
  - Always search for the most recent documentation as of mid-2025
  - Verify the library version against the latest available release

  ```javascript
  // ✅ DO: Fallback to web search when Context7 fails
  try {
    // First attempt to use Context7
    const libraryIdResponse =
      (await mcp_context7_resolve) -
      library -
      id({
        libraryName: "some-library",
      });

    const docsResponse =
      (await mcp_context7_get) -
      library -
      docs({
        context7CompatibleLibraryID: libraryIdResponse.libraryId,
      });

    // Check if documentation is insufficient
    if (!docsResponse.content || docsResponse.content.length < 100) {
      throw new Error("Insufficient documentation");
    }
  } catch (error) {
    // If Context7 fails or returns insufficient docs, use web search
    const webResults = await web_search({
      search_term: "some-library latest documentation api reference mid 2025",
      explanation: "Context7 documentation was unavailable or insufficient",
    });

    // Analyze multiple search results to get comprehensive information
    const latestDocs = webResults.filter(
      (result) =>
        result.includes("documentation") ||
        result.includes("api reference") ||
        result.includes("guide")
    );

    // Use these web results to guide implementation
  }

  // ❌ DON'T: Skip web search when Context7 fails
  // ❌ DON'T: Proceed with implementation without documentation
  // ❌ DON'T: Use outdated web search results (verify they're current as of mid-2025)
  ```

---
> Source: [aiurda/devcontext](https://github.com/aiurda/devcontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
