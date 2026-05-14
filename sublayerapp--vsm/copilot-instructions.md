## vsm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VSM is a Ruby gem that provides an async, recursive agent framework based on the Viable System Model. It implements five named systems (Operations, Coordination, Intelligence, Governance, Identity) in a capsule-based architecture for building AI agents.

## Development Commands

### Testing
```bash
bundle exec rspec                    # Run all tests
bundle exec rspec spec/vsm_spec.rb   # Run specific test file
rake spec                            # Alternative test runner
```

### Code Quality
```bash
bundle exec rubocop                  # Run linter
rake rubocop                         # Alternative linter runner
rake                                 # Run both tests and linter (default task)
```

### Gem Development
```bash
bundle install                       # Install dependencies
bundle exec rake build              # Build the gem
bundle exec rake install            # Install gem locally
bundle exec rake release            # Release gem (maintainers only)
```

### Interactive Development
```bash
bundle exec irb -r vsm              # Start IRB with VSM loaded
```

## Architecture

### Core Components

**Capsule** (`lib/vsm/capsule.rb:4`) - Main building block containing five systems plus AsyncChannel bus. The core dispatch loop runs in `lib/vsm/capsule.rb:18` processing messages through coordination → governance → routing.

**Five Systems** (Viable System Model roles):
- **Operations** (`lib/vsm/roles/operations.rb:7`) - Handles `:tool_call` messages, routes to child capsules/tools
- **Coordination** (`lib/vsm/roles/coordination.rb:3`) - Message scheduling, turn management, session floor control
- **Intelligence** (`lib/vsm/roles/intelligence.rb`) - Planning/decision making (typically LLM integration)
- **Governance** (`lib/vsm/roles/governance.rb`) - Policy enforcement, safety, budgets
- **Identity** (`lib/vsm/roles/identity.rb`) - Purpose definition, invariants, escalation

**AsyncChannel** (`lib/vsm/async_channel.rb:4`) - Non-blocking message bus built on `async` gem with pub/sub support.

**DSL Builder** (`lib/vsm/dsl.rb:4`) - Declarative capsule construction with role injection and child management.

### Tool System

**ToolCapsule** (`lib/vsm/tool/capsule.rb:3`) - Base class for tools that are capsules with ActsAsTool mixin.

**ActsAsTool** (`lib/vsm/tool/acts_as_tool.rb:3`) - Mixin providing tool metadata (name, description, JSON schema) and descriptor generation.

**Tool Descriptors** (`lib/vsm/tool/descriptor.rb`) - Provider-agnostic tool definitions with `to_openai_tool`, `to_anthropic_tool`, `to_gemini_tool` methods.

### Execution Models

**Executors** (`lib/vsm/executors/`) - Pluggable execution strategies:
- `FiberExecutor` - Default async/IO-bound execution
- `ThreadExecutor` - CPU-bound or blocking library execution

Tools specify execution mode via `execution_mode` method returning `:fiber` or `:thread`.

### Message Flow

Messages follow this structure:
```ruby
VSM::Message.new(
  kind: :user | :assistant | :assistant_delta | :tool_call | :tool_result | :plan | :policy | :audit,
  payload: any_data,
  corr_id: "correlation_id_for_tool_calls",
  meta: { session_id: "uuid", ... }
)
```

Core flow: User input → Intelligence (planning) → Operations (tool execution) → Coordination (scheduling) → Assistant output.

### Provider Integration

LLM drivers are in `lib/vsm/drivers/` with async implementations for:
- OpenAI (`drivers/openai/async_driver.rb`)
- Anthropic (`drivers/anthropic/async_driver.rb`) 
- Gemini (`drivers/gemini/async_driver.rb`)

Drivers are provider-agnostic and integrate through Intelligence implementations.

## Key Development Patterns

### Creating Tools
Inherit from `VSM::ToolCapsule`, define tool metadata with class methods, implement `run(args)`:

```ruby
class MyTool < VSM::ToolCapsule
  tool_name "my_tool"
  tool_description "Description"
  tool_schema({ type: "object", properties: { ... }, required: [...] })
  
  def run(args)
    # Tool implementation
  end
  
  def execution_mode = :fiber # or :thread
end
```

### Building Capsules
Use the DSL in `lib/vsm/dsl.rb:45` to define capsules with five systems:

```ruby
capsule = VSM::DSL.define(:name) do
  identity     klass: VSM::Identity, args: { identity: "name", invariants: [] }
  governance   klass: VSM::Governance
  coordination klass: VSM::Coordination  
  intelligence klass: MyIntelligence
  operations do
    capsule :tool_name, klass: ToolClass
  end
end
```

### Testing Approach
- Unit test individual roles and tools directly
- Test capsule message flow with queues to capture emissions
- Use RSpec with async-rspec for fiber-based testing
- Mock external dependencies (LLM APIs) in Intelligence tests

The codebase follows Ruby conventions with frozen string literals, clear namespacing, and small focused classes following SOLID principles.

---
> Source: [sublayerapp/vsm](https://github.com/sublayerapp/vsm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
