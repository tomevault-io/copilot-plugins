## ruby-llm-mcp

> Define reusable MCP toolsets and wire them into RubyLLM agents with lazy connection startup.


# Agents
{: .no_toc }

{: .label .label-yellow }
New in v1.1

This guide covers two integration points:

- `RubyLLM::MCP.toolset` for declaring named MCP tool groups.
- `RubyLLM::MCP::Agents` for attaching toolsets and MCP clients to `RubyLLM::Agent`.

## MCP DSL on Toolset

Use a named toolset to group MCP clients and apply include/exclude rules.

```ruby
# config/initializers/ruby_llm_mcp_toolsets.rb
RubyLLM::MCP.toolset(:support) do |toolset|
  toolset.from_clients :filesystem, :projects
  toolset.exclude_tools "delete_file"
end
```

That is the module-style config. You can also use hash-style DSL:

```ruby
RubyLLM::MCP.toolset(
  :support,
  clients: [:filesystem, :projects],
  exclude_tools: ["delete_file"],
  include_tools: ["read_file", "search_files"]
)
```

Use the block-scoped API when you want explicit access to the tools:

```ruby
RubyLLM::MCP.toolset(:support).with_tools do |support_tools|
  RubyLLM.chat.with_tools(*support_tools).ask("Find unresolved support items.")
end
```

`with_tools` keeps the selected MCP connections alive while the block runs and closes them afterward.

## MCP::Agents module example

Use `RubyLLM::MCP::Agents` in your agent class to wire toolsets into `ask` without loading MCP before execution.

```ruby
class SupportAgent < RubyLLM::Agent
  include RubyLLM::MCP::Agents

  with_toolsets :support
end

SupportAgent.new.ask("Find unresolved items for files modified in the last day.")
```

This gives the agent lazy startup and automatic cleanup around each call.

You can also attach MCP clients directly:

```ruby
class SupportAgent < RubyLLM::Agent
  include RubyLLM::MCP::Agents

  with_mcps :filesystem, :projects
end
```

You can combine both:

```ruby
class SupportAgent < RubyLLM::Agent
  include RubyLLM::MCP::Agents

  with_toolsets :support
  with_mcps :filesystem
end
```

Tool filtering should be configured on the toolset itself (via `include_tools` and `exclude_tools` in `RubyLLM::MCP.toolset`), not on the agent class.

`with_mcp_tools` remains available as an alias for `with_toolsets`.

Tools are attached only when `ask` runs, so MCP clients are started lazily and cleaned up right after the call.

---
> Source: [patvice/ruby_llm-mcp](https://github.com/patvice/ruby_llm-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
