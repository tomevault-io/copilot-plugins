## tiny-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `bundle install` - Install dependencies
- `bin/setup` - Setup script that installs dependencies
- `bin/console` - Interactive Ruby console with the gem loaded

### Testing
- `rake test` or `bundle exec rake test` - Run all tests
- `rake` - Default task (runs tests)
- `ruby -Ilib:test test/test_tiny_mcp.rb` - Run specific test file

### Building and Release
- `bundle exec rake install` - Build and install gem locally
- `bundle exec rake release` - Create git tag and push gem to RubyGems

## Architecture Overview

TinyMCP is a Ruby implementation of an MCP (Model Context Protocol) server that enables creating tools callable through JSON-RPC.

### Core Components

1. **TinyMCP::Server** - JSON-RPC server that:
   - Reads requests from STDIN
   - Processes tool calls via JSON-RPC protocol
   - Writes responses to STDOUT
   - Implements MCP server protocol

2. **TinyMCP::Tool** - Base class for tools:
   - Inherit from this class to create new tools
   - Use DSL methods: `name`, `desc`, `arg`, `opt` for metadata
   - Implement `call` method with tool logic
   - Parameters are automatically validated

3. **Data Classes** (using Ruby's Data.define):
   - `Definition` - Tool metadata (name, description, parameters)
   - `Prop` - Parameter properties (name, description, type, required)

### Tool Definition Pattern

Tools use a declarative DSL:
```ruby
class MyTool < TinyMCP::Tool
  name 'my_tool'
  desc 'Tool description'
  arg :required_param, 'string', 'Parameter description'
  opt :optional_param, 'string', 'Optional parameter'
  
  def call(required_param:, optional_param: nil)
    # Implementation
  end
end
```

### JSON-RPC Protocol

The server implements standard JSON-RPC 2.0 for MCP:
- Methods: `initialize`, `tools/list`, `tools/call`
- Input/output via STDIN/STDOUT
- Request/response format follows MCP specification

## Important Notes

- Ruby version requirement: >= 3.1.0
- Test framework: Minitest
- No linting configuration currently set up
- Gem is in early development (has TODOs in gemspec)
- CI runs on Ruby 3.4.1 via GitHub Actions

---
> Source: [maxim/tiny_mcp](https://github.com/maxim/tiny_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
