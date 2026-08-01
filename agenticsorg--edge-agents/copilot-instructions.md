## edge-agents

> required: true                         # Ensure all code changes are backed by tests

# SPARC Focus: Specification - Define project objectives, requirements, and user scenarios
test_coverage:
  required: true                         # Ensure all code changes are backed by tests
  minimum_percentage: 80                 # Set a baseline of 80% test coverage
  exclude_paths:                         # Exclude paths that do not require testing
    - test/fixtures
    - scripts

# SPARC Focus: Specification - Set guidelines for code quality
lint_rules:
  disable_requires_approval: true        # Require explicit approval to disable lint rules
  enforce_eslint: true                   # Enforce ESLint for code consistency
  enforce_prettier: true                 # Enforce Prettier for consistent formatting

# SPARC Focus: Specification - Define styling and UI/UX guidelines
styling:
  use_tailwind: true                     # Utilize Tailwind CSS for standardized design
  prefer_functional_components: true     # Prefer functional components for UI development
  css_variables_location: webview-ui/src/index.css  # Central location for CSS variable definitions

# SPARC Focus: Specification - Environment-specific configurations
environments:
  python:
    virtual_env: true                    # Use a virtual environment for Python projects
    linter: flake8                       # Lint Python code with Flake8
    formatter: black                     # Format Python code with Black
  node:
    package_manager: npm                 # Use npm as the package manager
    test_runner: jest                    # Use Jest for running JavaScript tests
  deno:
    enabled: true                        # Enable Deno support
    version: ">=1.30.0"                  # Minimum Deno version required

# SPARC Focus: Pseudocode & Architecture - Model configuration for AI integration
provider:
  default: google/gemini-2.5-pro-experimental   # Default AI model provider
thinkingProvider:
  model: google/gemini-2.0-flash                # Model for quick, responsive analysis
docModel: google/gemini-2.0-pro                 # Model for processing documentation

# SPARC Focus: Pseudocode - Operational mode for code processing
mode: code                                      # Set the mode to 'code' for development tasks

# SPARC Focus: Architecture - Real-time updates configuration
real_time_updates:
  enabled: true                                 # Enable real-time context updates
  update_triggers:
    project_related:
      - documentation_gap                       # Trigger when documentation is incomplete
      - knowledge_update                        # Trigger upon new information or updates
    system_related:
      - error_pattern                           # Trigger when repeated error patterns are found
      - performance_insight                     # Trigger when performance insights become available

# SPARC Focus: Refinement - Use unified diffs for structured code edits
edit_format: unified_diff                       # Adopt unified diff for clear code modifications
high_level_edits: true                          # Encourage holistic, high-level changes
exclude_line_numbers: true                      # Exclude line numbers for cleaner diffs

# SPARC Focus: Architecture - Mode switching for multiple development contexts
mode_switching:
  enabled: true                                 # Allow automatic mode switching
  preserve_context: true                        # Retain context during mode transitions

# SPARC Focus: Pseudocode - Intent-based triggers for switching modes
intent_triggers:
  code:
    - implement                                 # Switch to code mode for implementing features
    - create                                    # Switch to code mode for creating components
    - build                                     # Switch to code mode for building modules
    - fix                                       # Switch to code mode for fixing issues
  architect:
    - design                                    # Switch to architect mode for designing systems
    - structure                                 # Switch to architect mode for structural planning
    - plan                                      # Switch to architect mode for project planning

# SPARC Focus: Architecture - File-based triggers for dynamic mode switching
file_triggers:
  - pattern: "\.tsx$"                           # Activate code mode for TSX files
    target_mode: code
    condition: file_edit
  - pattern: "\.md$"                            # Activate document mode for Markdown files
    target_mode: document
    condition: file_create

# SPARC Focus: Refinement - Terminal command management
terminal:
  allowed_commands:
    - npm test                                  # Run JavaScript tests
    - npm install                               # Install Node dependencies
    - tsc                                       # Compile TypeScript code
    - git log                                   # Show commit history
    - git show                                  # Display Git object content
    - cd                                        # Change directory
    - pip                                       # Install Python packages
    - docker                                    # Execute Docker commands
    - cd ../                                    # Move one directory up
    - python                                    # Run Python interpreter
    - aider                                     # Custom or third-party script
    - streamlit                                 # Launch Streamlit for Python web apps
    - export                                    # Set environment variables
    - ls                                        # List directory contents
    - coverage                                  # Check code coverage
    - node                                      # Run Node.js commands
    - npm run                                   # Execute npm scripts
    - cargo                                     # Use Cargo for Rust projects
    - pytest                                    # Run Python tests with pytest
    - source                                    # Source environment variables
    - cd agents/                                # Navigate to agents directory
    - cd parser                                 # Navigate to parser directory
    - chmod                                     # Change file permissions
    - mkdir                                     # Create a new directory
    - bash                                      # Run Bash commands
    - curl                                      # Transfer data from or to a server
    - npx                                       # Execute npm package binaries
    - sparc2run                                 # Custom or third-party script
    - uvicorn                                   # ASGI server for Python
    - pkill                                     # Kill processes by name
    - pwd                                       # Print working directory
    - grep                                      # Search for text patterns
    - deno task                                 # Run Deno tasks
    - deno compile                              # Compile Deno applications
    - deno test                                 # Run Deno tests
    - deno fmt                                  # Format Deno code
    - deno lint                                 # Lint Deno code
    - deno check                                # Type-check Deno code
  blocked_commands:
    - rm -rf                                    # Prevent dangerous recursive deletion
    - git push                                  # Block pushing changes without review

# SPARC Focus: Architecture - Model Control Panel (MCP) integration
mcp_server:
  enabled: true                                 # Enable MCP server features
  url: http://localhost:3001                    # Updated MCP server URL to port 3001
  auth_token: local_dev_token                   # Authentication token for local environment
  features:
    - code_search                               # Enable code searching functionality
    - project_indexing                          # Index projects for faster lookups
  servers:
    sparc2-mcp:                                 # Configuration for sparc2-mcp server
      url: http://localhost:3001                # Server URL
      tools:
        - analyze_code                          # Code analysis tool
        - modify_code                           # Code modification tool
        - search_code                           # Code search tool
      file_paths:
        - src/                                  # Base path for source files
        - scripts/                              # Base path for scripts

# SPARC Focus: Refinement - Automatic checkpoints for safe code recovery
checkpoints:
  enabled: true                                 # Turn on automatic checkpointing
  auto_save: true                               # Save checkpoints automatically
  interval_minutes: 15                          # Checkpoint interval in minutes
  max_checkpoints: 5                            # Maximum number of stored checkpoints

# SPARC Focus: Refinement - Configure repository search
repository_search:
  enabled: true                                 # Activate repository search
  exclude_paths:
    - node_modules                              # Exclude node_modules folder for efficiency
    - .git                                      # Exclude Git repository metadata
    - dist                                      # Exclude distribution output
  include_file_types:
    - .ts                                       # Include TypeScript files
    - .tsx                                      # Include TSX files
    - .js                                       # Include JavaScript files
    - .jsx                                      # Include JSX files

# SPARC Focus: Completion - Custom command shortcuts
command_shortcuts:
  start_dev: npm run dev                        # Shortcut to start the development server
  run_tests: deno test --allow-read --allow-env src/  # Updated to use Deno for tests
  build_project: cd scripts/sparc2 && deno task build  # Updated to use Deno for building SPARC2
  build_npm: cd scripts/sparc2 && node scripts/build-npm.js  # Added for building NPM package
  deploy_staging: npm run deploy:staging        # Shortcut to deploy to a staging environment
  analyze_code: sparc2 analyze                  # Fixed typo and added command to analyze code quality
  format_code: deno fmt src/                    # Added shortcut for code formatting
  check_types: deno check src/**/*.ts           # Added shortcut for type checking
  
# SPARC Focus: Completion - Manage memory for dynamic context updates
memory_bank:
  update_requests:
    high_priority:
      - activeContext.md                        # Update active context details
      - progress.md                             # Log ongoing progress
    medium_priority:
      - decisionLog.md                          # Track significant decisions
      - productContext.md                       # Capture product-specific context

---
> Source: [agenticsorg/edge-agents](https://github.com/agenticsorg/edge-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
