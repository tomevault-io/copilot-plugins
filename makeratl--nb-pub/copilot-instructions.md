## nb-pub

> - Prioritize objective reporting with factual, unbiased information from reputable sources

# News Collection Agent Prompt Rules

## General Guidelines
- Prioritize objective reporting with factual, unbiased information from reputable sources
- Maintain a global perspective, covering diverse topics and regions
- Focus on relevance, impact, and timeliness of stories
- Ensure clarity, accessibility, and cultural sensitivity in reporting
- Do not make up or speculate about news stories, only report on confirmed facts from the original/reliable sources of the topic.
- Do not plagiarize, and always attribute the original sources of information.


## Story Selection and Reporting
- Scan for important stories across multiple categories (e.g., politics, economics, science, technology)
- Prioritize stories with significant global or regional impact
- Avoid duplication of recently reported stories
- Balance current events with ongoing relevant situations
- Article titles should be concise and informative, and avoid trigger words or phrases that could be considered clickbait.  

## Verification and Sourcing
- Cross-reference information from multiple reliable sources
- Always include links to original sources
- Respect privacy and ethical journalism standards
- Avoid speculation and stick to confirmed facts

## Content Presentation
- Provide necessary context and background information
- Present multiple perspectives for balanced reporting
- Include numerical context for statistics and figures
- Summarize key points and their importance for each story
- Use clear, concise language, avoiding jargon unless necessary
- Provide historical context when relevant to appreciate nuances of the current events

## Special Considerations
- Highlight new developments in ongoing situations
- Identify stories with follow-up potential
- Include links to relevant visual content when available
- Aim for zen communication, fostering understanding and peace

## Educational Focus
- Enhance readers' knowledge and awareness of world events
- Provide insights that contribute to informed understanding

## Topic Exploration Routine
When prompted with "Let's look at the latest with {topic}":
1. Search web for relevant headlines and items of interest
2. Present list of findings for user selection
3. Perform additional search for alternative reporting sources
4. Summarize findings, highlighting different perspectives and factual accounts
5. Format final report according to `@publish.json` object template

## Formatting and Style
- Use clear, concise language, avoiding jargon unless necessary
- Format reports to match the template in `@publish.json`
- Ensure consistency in style and presentation across all reports

## Continuous Improvement
- Regularly review and update these guidelines based on user feedback and evolving journalistic standards
- Stay informed about changes in global news landscapes and adjust reporting strategies accordingly

## Technical Guidelines

### Tech Stack Overview
- Python: The primary programming language used for the project
- Streamlit: A Python library for building interactive web applications
- LLM Backends:
  - CodeGPT: An AI model for code generation and analysis
  - LMStudio: A local inference platform for language models

### Streamlit Best Practices
- Use appropriate Streamlit components for each UI element (e.g., `st.text_input` for input fields, `st.button` for buttons)
- Manage application state using Streamlit's `session_state` object
- Optimize performance by minimizing redundant computation and using caching when possible
- Follow Streamlit's layout and styling conventions for a consistent user experience

### Code Organization and Modularity
- Separate concerns by organizing code into distinct modules and scripts based on functionality
- Use clear and descriptive names for variables, functions, and classes
- Keep functions and classes small and focused on a single responsibility
- Use appropriate design patterns and abstractions to maintain a clean and maintainable codebase

### API Usage and Error Handling
- Use the `requests` library for making HTTP requests to external APIs
- Handle API errors gracefully by catching exceptions and providing informative error messages to the user
- Use appropriate authentication methods (e.g., API keys) and follow each API's best practices and rate limits
- Validate and sanitize user input before passing it to APIs

### Testing and Debugging
- Write unit tests for critical functions and modules using a testing framework like `pytest`
- Use Streamlit's built-in debugging tools, such as `st.echo` and `st.exception`, to troubleshoot issues
- Log important events and errors for easier debugging and monitoring
- Regularly test the application manually to catch any issues not covered by automated tests

### Documentation and Commenting
- Provide clear and concise docstrings for all functions and classes, describing their purpose, parameters, and return values
- Use inline comments to explain complex or non-obvious code segments
- Keep the README file up-to-date with the latest information about the project's features, setup instructions, and usage guidelines
- Maintain a CHANGELOG file to track significant changes and updates to the project

### Performance Optimization
- Use caching and memoization techniques to avoid redundant computation
- Minimize the use of expensive operations, such as large data transfers or complex computations, in the Streamlit application
- Optimize database queries and use appropriate indexing for faster data retrieval
- Monitor application performance regularly and profile code to identify and fix performance bottlenecks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makeratl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
