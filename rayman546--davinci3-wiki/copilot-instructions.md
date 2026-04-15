## davinci3-wiki

> When working on complex tasks, ALWAYS use MCP tools like Sequential Thinking to break down the problem and plan the implementation. This helps maintain clarity and ensures thorough consideration of all aspects.

# ALWAYS USE MCP TOOLS

When working on complex tasks, ALWAYS use MCP tools like Sequential Thinking to break down the problem and plan the implementation. This helps maintain clarity and ensures thorough consideration of all aspects.

# Implementation Plan for Offline Wikipedia System

## Phase Order
1. Error Handling & Logging Framework (Tool 9) - ✅
2. Data Ingestion & Parsing (Tool 1) - ✅
3. Database & Indexing (Tool 2) - ✅
4. Vector Embedding (Tool 3) - ✅
5. LLM Integration (Tool 4) - ✅
6. Installation Manager (Tool 5) - ✅
7. UI/Frontend (Tool 6) - ✅
8. Testing & Documentation (Tools 7,8) - ✅

## Technical Stack
- Backend: Rust
- Database: SQLite + FTS5
- Vector Store: LMDB
- LLM: Ollama
- UI: Flutter
- Async: Tokio
- Logging: Tracing

## Project Structure
```
/src
  /error_handling - Tool 9
  /parser - Tool 1
  /db - Tool 2
  /vector - Tool 3
  /llm - Tool 4
  /installer - Tool 5
  /ui - Tool 6
  /tests - Tool 7
  /docs - Tool 8
```

## Performance Requirements
- Search latency: < 500ms
- Article load time: < 1 sec
- LLM response: < 3 sec
- Initial load: < 5 sec
- Memory usage: < 2GB

## Dependencies
- Rust (latest stable)
- SQLite 3.35.0+
- LMDB 0.9+
- Ollama (latest)
- Flutter 3.0+
- Tokio 1.0+

## Next Steps
1. Performance optimization
2. User experience improvements
3. Deployment and distribution

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again. 

You should also use the `.cursorrules` file as a Scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the Scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the Scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

Note all the tools are in python3. So in the case you need to do batch processing, you can always consult the python files and write your own script.

## Screenshot Verification

The screenshot verification workflow allows you to capture screenshots of web pages and verify their appearance using LLMs. The following tools are available:

1. Screenshot Capture:
```bash
venv/bin/python3 tools/screenshot_utils.py URL [--output OUTPUT] [--width WIDTH] [--height HEIGHT]
```

2. LLM Verification with Images:
```bash
venv/bin/python3 tools/llm_api.py --prompt "Your verification question" --provider {openai|anthropic} --image path/to/screenshot.png
```

Example workflow:
```python
from screenshot_utils import take_screenshot_sync
from llm_api import query_llm

# Take a screenshot

screenshot_path = take_screenshot_sync('https://example.com', 'screenshot.png')

# Verify with LLM

response = query_llm(
    "What is the background color and title of this webpage?",
    provider="openai",  # or "anthropic"
    image_path=screenshot_path
)
print(response)
```

## LLM

You always have an LLM at your side to help you with the task. For simple tasks, you could invoke the LLM by running the following command:
```
venv/bin/python3 ./tools/llm_api.py --prompt "What is the capital of France?" --provider "anthropic"
```

The LLM API supports multiple providers:
- OpenAI (default, model: gpt-4o)
- Azure OpenAI (model: configured via AZURE_OPENAI_MODEL_DEPLOYMENT in .env file, defaults to gpt-4o-ms)
- DeepSeek (model: deepseek-chat)
- Anthropic (model: claude-3-sonnet-20240229)
- Gemini (model: gemini-pro)
- Local LLM (model: Qwen/Qwen2.5-32B-Instruct-AWQ)

But usually it's a better idea to check the content of the file and use the APIs in the `tools/llm_api.py` file to invoke the LLM if needed.

## Web browser

You could use the `tools/web_scraper.py` file to scrape the web.
```bash
venv/bin/python3 ./tools/web_scraper.py --max-concurrent 3 URL1 URL2 URL3
```
This will output the content of the web pages.

## Search engine

You could use the `tools/search_engine.py` file to search the web.
```bash
venv/bin/python3 ./tools/search_engine.py "your search keywords"
```
This will output the search results in the following format:
```
URL: https://example.com
Title: This is the title of the search result
Snippet: This is a snippet of the search result
```
If needed, you can further use the `web_scraper.py` file to scrape the web page content.

# Lessons

## User Specified Lessons

- You have a python3 venv in ./venv. Use it.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- Due to Cursor's limit, when you use `git` and `gh` and need to submit a multiline commit message, first write the message in a file, and then use `git commit -F <filename>` or similar command to commit. And then remove the file. Include "[Cursor] " in the commit message and PR title.

## Cursor learned

- For search results, ensure proper handling of different character encodings (UTF-8) for international queries
- Add debug information to stderr while keeping the main output clean in stdout for better pipeline integration
- When using seaborn styles in matplotlib, use 'seaborn-v0_8' instead of 'seaborn' as the style name due to recent seaborn version changes
- Use 'gpt-4o' as the model name for OpenAI's GPT-4 with vision capabilities
- When searching for recent news, use the current year (2025) instead of previous years, or simply use the "recent" keyword to get the latest information
- When Flutter projects use freezed/json_serializable, be sure to run build_runner before running the app
- Fix import paths for packages like shared_preferences (package:shared_preferences/shared_preferences.dart)
- Enable Windows Developer Mode for Flutter symlink support
- When converting SVG to PNG on Windows, consider using HTML embedding as a workaround for dependency issues

## PowerShell Lessons

- When working with PowerShell on Windows:
  - Use `Set-Content` with `-Encoding UTF8` for file creation
  - Prefer `python -c` for complex file operations over PowerShell commands
  - Virtual environment activation: `.\venv\Scripts\Activate.ps1`
  - Avoid multiline commands and complex string escaping
  - Use semicolon (;) instead of && for command chaining

# Scratchpad

## Current Task: Pre-Launch Implementation Plan

### Phase 1: Core Backend Completion (Completed)
[X] Implement HTTP server using axum
  [X] Create server struct with configuration
  [X] Implement startup/shutdown handling
  [X] Add health check endpoint
[X] Define and implement API endpoints
  [X] GET /articles/{id} - Retrieve article by ID
  [X] GET /articles/title/{title} - Retrieve article by title
  [X] GET /search?q={query} - Full-text search
  [X] GET /semantic-search?q={query} - Semantic/vector search
  [X] POST /llm/generate - Generate text with LLM
  [X] GET /status - System status
  [X] POST /install - Start installation
  [X] POST /update - Start update process
[X] Implement Update command functionality
  [X] Add differential update mechanism
  [X] Implement change detection
  [X] Add incremental database updates
[X] Enhance error handling
  [X] Create middleware for consistent error responses
  [X] Implement detailed error logging
  [X] Add recovery mechanisms for common failures

### Phase 2: Enhanced Installation and LLM Integration (Completed)
[X] Improve installation process
  [X] Add progress reporting using channels
  [X] Implement cleanup for failed installations
  [X] Add checksum validation for downloaded dumps
  [X] Implement download resumption capability
  [X] Add system requirements verification
[X] Enhance LLM integration
  [X] Complete error handling in LLM module
  [X] Add model version checking
  [X] Implement rate limiting and timeout handling
  [X] Add retry mechanisms for failed requests

### Phase 3: UI Completion and Performance Optimization (Completed)
[X] Complete UI implementation
  [X] Add connectivity service for online/offline detection
  [X] Implement Provider-based state management
    [X] Create app state provider
    [X] Create connectivity state provider
    [X] Implement Provider-based service access
  [X] Update pages to use proper Provider pattern
    [X] Update ArticlesPage to use Provider
    [X] Update ArticleDetailsPage to use Provider
    [X] Update SearchPage to use Provider
    [X] Update SettingsPage to use Provider
  [X] Implement error handling UI components
    [X] Create error message widgets
    [X] Add retry functionality
    [X] Disable actions when offline
  [X] Add loading states and progress indicators
    [X] Create loading indicator widgets
    [X] Add skeleton loaders for content
  [X] Implement offline mode handling
    [X] Add offline mode indicator
    [X] Improve offline fallback experiences
    [X] Add offline status messages
[X] Optimize performance
  [X] Implement article caching strategy
    [X] Add cache prioritization for frequently accessed articles
    [X] Implement cache size limits and cleanup policies
    [X] Add cache analytics
  [X] Optimize database queries
    [X] Add query timing metrics
    [X] Optimize slow queries
    [X] Add query caching where appropriate
  [X] Add lazy loading for article content
    [X] Implement progressive loading for long articles
    [X] Add image lazy loading
    [X] Implement virtual scrolling for large lists
  [X] Implement memory usage monitoring
    [X] Add memory usage reporting
    [X] Implement memory cleanup mechanisms
    [X] Add warnings for high memory usage
  [X] Fine-tune vector search parameters
    [X] Optimize vector similarity thresholds
    [X] Implement performance metrics for vector searches
    [X] Add result quality measurements

### Phase 4: Testing and Documentation (In Progress)
[ ] Complete testing suite
  [ ] Add missing integration tests
    [X] Test incremental update functionality
    [X] Test error handling and recovery scenarios
    [X] Test concurrent access to APIs
    [X] Test network interruption and resumption
    [X] Test system resource limits
  [ ] Create UI automated tests
    [X] Test HomePage navigation
    [X] Test ArticlesPage loading and pagination
    [X] Test SearchPage functionality
    [X] Test SettingsPage interactions including cache operations
    [X] Test ArticleDetailsPage rendering
    [X] Test offline mode behavior
    [X] Test performance metrics display
  [ ] Implement stress tests for concurrent users
    [ ] Create simulation for multiple concurrent users
    [ ] Test system under various load conditions
    [ ] Test memory usage with many concurrent requests
  [ ] Enhance performance benchmarking
    [ ] Create comprehensive performance test suite
    [ ] Add search response time benchmarks
    [ ] Measure memory usage under various conditions
    [ ] Test cache efficiency with different strategies
    [ ] Create performance regression detection
[ ] Finalize documentation
  [ ] Complete API documentation
    [ ] Document all endpoints with examples
    [ ] Add error codes and descriptions
    [ ] Document request/response formats
    [ ] Create API usage examples
  [ ] Add installation troubleshooting guide
    [ ] Document common installation issues
    [ ] Add solutions for common problems
    [ ] Create diagnostic tools section
    [ ] Add system requirements verification steps
  [ ] Create deployment guide
    [ ] Document deployment options
    [ ] Add configuration examples
    [ ] Create production setup guide
    [ ] Document scaling considerations
  [ ] Finish user manual sections
    [ ] Complete user interface guide
    [ ] Add searching techniques documentation
    [ ] Document offline usage patterns
    [ ] Create performance optimization guide

### Phase 5: Cross-platform, Security, and Distribution
[ ] Ensure cross-platform compatibility
  [ ] Verify and complete platform-specific code
  [ ] Create platform-specific installation instructions
  [ ] Test on all target platforms
[ ] Implement security measures
  [ ] Add input validation on all API endpoints
  [ ] Implement rate limiting
  [ ] Add secure storage for sensitive settings
  [ ] Create update verification mechanism
[ ] Set up distribution
  [ ] Create release pipeline
  [ ] Build installers for different platforms
  [ ] Implement auto-update mechanism
  [ ] Add crash reporting system

### Progress Notes
- We've now implemented the HTTP server using warp instead of axum (already included in the project)
- Added all required API endpoints with proper error handling
- Integrated the server with the Start command in main.rs
- Implemented incremental update functionality that:
  - Compares existing articles with new articles from the dump
  - Only adds new articles that don't exist yet
  - Reports on articles that were removed from the dump
  - Generates embeddings only for new articles
- Phase 1 is now complete
- Implemented installation improvements for Phase 2:
  - Added progress reporting using channels and WebSocket for real-time updates
  - Implemented cleanup for failed installations
  - Added checksum validation for downloaded dumps
  - Implemented download resumption capability
  - Added system requirements verification
- Enhanced LLM integration for Phase 2:
  - Added comprehensive error handling with specific error types
  - Implemented model version checking and automatic model pulling
  - Added rate limiting with sliding window and semaphore
  - Implemented request timeouts and automatic retries with exponential backoff
  - Added platform-specific Ollama startup code
- Phase 2 is now complete, moving to Phase 3
- For Phase 3, we've completed the UI implementation:
  - Added ConnectivityService to monitor network connectivity status
  - Created ConnectivityProvider with Provider pattern for state management
  - Updated main.dart to use MultiProvider for dependency injection
  - Modified all pages (Articles, Search, Settings, ArticleDetails) to:
    - Access services via Provider instead of direct injection
    - Handle offline mode with appropriate UI elements
    - Disable network-dependent functions when offline
    - Add retry mechanisms for network operations
  - Added visual indicators for connectivity status
  - Implemented offline message banners
- Performance optimization is now complete:
  - Enhanced CacheService with:
    - Memory and disk caching with TTL
    - Cache size limits and eviction policies
    - Access tracking for prioritization
    - Cache statistics and analytics
  - Optimized WikiService with:
    - Query timing metrics
    - Cache hit/miss tracking
    - Background caching for frequently accessed content
    - Prioritized cache access for better performance
  - Added performance monitoring UI:
    - Created PerformanceMetricsPanel widget
    - Updated SettingsPage to display metrics
    - Added most accessed articles section
  - Phase 3 is now complete, moving to Phase 4
- Beginning Phase 4 with testing and documentation:
  - Assessed current test coverage:
    - Backend has basic e2e, API, and performance tests
    - UI testing is minimal with only template test
  - Identified testing gaps:
    - Backend needs tests for incremental updates, error handling, and concurrency
    - UI needs comprehensive tests for all pages and functionalities
  - Implemented test for incremental update functionality:
    - Added test_incremental_update() to e2e_test.rs
    - Created get_updated_test_xml_dump() in mock_data.rs
    - Added get_all_article_titles() methods to DB layer
    - Verified full update cycle including detection of new, modified, and removed articles
  - Implemented test for error handling and recovery:
    - Added test_error_handling_and_recovery() to e2e_test.rs
    - Created get_partial_invalid_xml_dump() in mock_data.rs
    - Tested handling of invalid XML, database corruption, and recovery
    - Verified system can recover from various error conditions
  - Implemented test for concurrent access to APIs:
    - Added test_concurrent_access() to e2e_test.rs
    - Created tests for concurrent read operations (50 simultaneous reads)
    - Added tests for concurrent search operations (20 simultaneous searches)
    - Implemented mixed operation test (combination of reads, text searches, and semantic searches)
    - Verified database integrity is maintained under concurrent load
  - Implemented test for network interruption and resumption:
    - Added test_network_interruption() to e2e_test.rs
    - Tested handling of network failures during installation
    - Verified ability to resume installation after network failure
    - Tested network failures during update process
    - Implemented tests for connection timeouts during searches
    - Verified system can recover from various network issues
  - Implemented test for system resource limits:
    - Added test_system_resource_limits() to e2e_test.rs
    - Tested system behavior with minimal memory settings
    - Implemented disk space constraint tests for both Unix and Windows
    - Added CPU stress testing with concurrent semantic searches
    - Measured performance degradation under high CPU load
    - Verified system can handle resource constraints gracefully
  - Implemented UI automated tests:
    - Created home_page_test.dart to test HomePage navigation:
      - Tested navigation rail with three destinations
      - Verified page switching when navigation items are selected
      - Tested connectivity status indicator display
    - Created articles_page_test.dart to test ArticlesPage:
      - Tested loading indicator display
      - Verified article list rendering
      - Tested error message display
      - Implemented pagination test with scroll to load more
      - Tested offline mode behavior with disabled refresh
    - Created search_page_test.dart to test SearchPage:
      - Tested initial empty state display
      - Verified search functionality with text input
      - Tested switching between regular and semantic search
      - Implemented search history display and usage tests
      - Tested error handling in search operations
      - Verified offline mode behavior with disabled search
    - Created settings_page_test.dart to test SettingsPage:
      - Tested cache management panel display
      - Verified connection status display
      - Tested performance metrics panel display
      - Implemented most accessed articles display test
      - Tested clear cache functionality
      - Verified error handling during cache operations
      - Tested offline mode behavior with disabled settings
    - Created article_details_page_test.dart to test ArticleDetailsPage:
      - Tested loading indicator display
      - Verified article content rendering
      - Tested related articles display
      - Implemented error handling tests
      - Tested refresh functionality
      - Verified offline mode behavior with disabled actions
    - Created offline_mode_test.dart to test offline behavior across the app:
      - Tested offline indicator display in HomePage
      - Verified disabled refresh in ArticlesPage when offline
      - Tested disabled search in SearchPage when offline
      - Verified disabled settings in SettingsPage when offline
      - Implemented tests for connectivity changes across all pages
      - Tested ConnectivityProvider notification system
    - Created performance_metrics_test.dart to test performance metrics display:
      - Tested PerformanceMetricsPanel component display
      - Verified cache overview metrics display
      - Tested query timings display
      - Implemented cache statistics display tests
      - Tested refresh functionality
      - Verified loading state display
      - Tested integration with SettingsPage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rayman546) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
