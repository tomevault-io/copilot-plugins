## rasti-ai

> Developer and agent reference. For usage examples see README.md.

# AGENTS.md — Rasti::AI internals

Developer and agent reference. For usage examples see README.md.

## Architecture

### Provider

`Rasti::AI::Provider` is the entry point that makes provider and model a runtime value instead of a class reference. An instance holds the transport configuration (`api_key`, `usage_tracker`, `logger`, HTTP timeouts) plus the default `model`, and knows how to build the provider's `Client` and `Assistant`.

The set of providers is **fixed** — there is no registry and third-party providers are not supported. Two frozen constants in the base class drive resolution:

```ruby
MODULES = {open_ai: 'OpenAI', gemini: 'Gemini', anthropic: 'Anthropic', open_router: 'OpenRouter', huawei_maas: 'HuaweiMaaS'}.freeze
ALIASES = {openai: :open_ai, openrouter: :open_router, huawei: :huawei_maas}.freeze
```

`Provider.build(name, **options)` normalizes the name (`downcase.to_sym`), applies aliases, and resolves the class with `AI.const_get(MODULES[key])::Provider` — at call time, so there is no load-order constraint. Passing an existing `Provider` instance returns it untouched. Unknown names raise `Errors::UnknownProvider`.

`client_class` / `assistant_class` are derived from `name` through `provider_module`, so `OpenRouter::Provider` (which inherits from `OpenAI::Provider`) gets `OpenRouter::Client` and `OpenRouter::Assistant` without overriding anything.

#### Provider — methods to implement (7 total)

Public — also consumed by the `Client`:

```ruby
def name                    # :open_ai — also used by provider_module
def default_model           # from Rasti::AI config
def default_api_key         # from Rasti::AI config
def base_url                # e.g. 'https://api.anthropic.com/v1'
def parse_usage(response)   # raw response => Usage or nil
```

Private — the request/response translation:

```ruby
def request(client:, messages:, system:, model:, json_schema:, thinking:)  # calls the client, returns raw response
def encode_message(message) # {role:, content:} with generic roles => provider message
def parse_content(response) # raw response => String
```

Plus a `THINKING_LEVELS` constant (see [Thinking levels](#thinking-levels)). OpenAI-compatible providers only override `name`, `default_model` and `default_api_key`.

#### Public API

- **`Provider#generate_text(prompt:/messages:, system:, model:, json_schema:, thinking:, client:)`** — one request, no tools, no loop. Normalizes generic-role messages, extracts `system` messages into the provider's system field, and returns a `Result`.
- **`Provider#create_assistant(state:, system:, model:, tools:, mcp_servers:, json_schema:, thinking:, client:)`** — builds the provider-specific `Assistant`. `system:` is sugar for `state: AssistantState.new(context: ...)`; passing both raises `ArgumentError`.
- **`Provider#build_client`** — the provider's `Client` with the instance's transport options.
- **`Rasti::AI.provider`, `Rasti::AI.generate_text`, `Rasti::AI.create_assistant`** — thin module-level wrappers. They split transport options (used to build the provider) from call options (forwarded to the provider method).

`model:` accepts `'provider:model'` (split on the first `:`), but only when `provider:` is not given explicitly. The provider falls back to `Rasti::AI.default_provider` (`ENV['AI_DEFAULT_PROVIDER']`); nil raises `ArgumentError`. `model` on a `Provider` is only its default — every call accepts a `model:` override.

The private helpers `build_provider` and `split_provider_and_model` live in a `class << self` block with `private` inside it.

### Result

`Rasti::AI::Result` (a `Rasti::Model`) is the normalized output of `generate_text`:

| Attribute | Content |
|---|---|
| `content` | The text the model returned, or the JSON string when `json_schema` is used |
| `usage` | A `Usage` instance, from `Provider#parse_usage` |
| `raw` | The untouched provider payload |

`Provider#parse_usage` is public because the `Client` also calls it, from `track_usage`, on every response of the assistant's tool loop.

`Assistant#call` still returns a plain `String` — unchanged for compatibility.

### Generic roles

`Rasti::AI::Roles` (`USER`, `ASSISTANT`, `SYSTEM`) is the provider-agnostic vocabulary used by `generate_text(messages:)`. Each `Provider#encode_message` translates it (Gemini maps `assistant` to its `model` role; `system` messages are pulled out of the array by `split_system` and passed through the provider's system field).

The per-provider `Roles` modules still exist and are what the `Assistant` classes use. Inside `Rasti::AI::<Provider>::*`, a bare `Roles::USER` resolves to the provider's module by lexical scope — use the fully qualified `Rasti::AI::Roles::USER` for the generic one.

> ⚠️ Do not reference a provider's `Roles` constants in a constant definition inside `<provider>/provider.rb`: `multi_require` loads `provider.rb` **before** `roles.rb` (alphabetical order). Resolve them inside method bodies instead (see `Gemini::Provider#encode_role`).

#### Known duplication

`Anthropic::Provider` reimplements the structured-output tool and its parsing, which also live in `Anthropic::Assistant` (Anthropic has no native `json_schema` support). Same for Gemini's `generation_config`. This is temporary: once the `Assistant` is rebuilt on top of `Provider#request`, both collapse into one. Keep them in sync until then.

### Template method pattern

`Rasti::AI::Client` and `Rasti::AI::Assistant` are abstract base classes. Provider-specific subclasses implement a fixed set of methods; all shared logic (HTTP retries, tool caching, the request/response loop) lives in the base.

#### Client — methods to implement

Nothing is mandatory. A client only defines the provider's endpoint method (`chat_completions`, `messages`, `generate_content`) and, if the API needs credentials in the request, one of:

```ruby
def build_request(uri)       # super + add auth headers
def build_url(relative_url)  # super + query params (e.g. Gemini key)
```

Everything else — API key, default model, base URL, usage parsing — lives in the `Provider`, and the client delegates (`provider` and `provider_module` come from the `ProviderAware` module, shared with `Assistant`):

```ruby
def default_api_key ; provider.default_api_key ; end
def default_model   ; provider.default_model   ; end
def base_url        ; provider.base_url        ; end
def track_usage(response) ; ... provider.parse_usage(response) ... ; end
```

As a result `OpenRouter::Client` and `HuaweiMaaS::Client` have **empty bodies**. Keep them: they are the namespace marker that `Provider#client_class` and `ProviderAware#provider_module` resolve against, and they keep `Rasti::AI::<Provider>::Client.new` working as a documented entry point.

`Provider#build_client` injects itself (`provider: self`). When a client is built standalone (`OpenAI::Client.new`) the private `provider` method lazily builds the provider of its own namespace, derived from the class name — so nothing breaks and no provider-specific config is duplicated.

> ⚠️ In `Client#initialize`, the `provider:` keyword argument shadows the private `provider` method. That's why the fallback goes through `default_api_key` instead of calling `provider.default_api_key` inline.

> ⚠️ The class-name derivation means an anonymous client subclass (`Class.new(OpenAI::Client)`) has no provider. Subclass explicitly if you need one in a test.

The base `post` method handles JSON serialization, logging, retries on network errors and 5xx, and calls `track_usage` after each successful response.

#### Assistant — methods to implement (11 total)

```ruby
def build_user_message(prompt)     # {role: ..., content: prompt}
def build_assistant_message(content)
def build_assistant_tool_calls_message(response)
def build_tool_result_message(tool_call, name, result)
def request_completion             # calls client's API method
def parse_tool_calls(response)     # returns array; empty = no tool call
def parse_content(response)        # extracts text string
def finished?(response)            # true when model is done
def extract_tool_call_info(tool_call)  # returns [name, args_hash]
def wrap_tool_serialization(raw)   # adapts ToolSerializer output to provider format
def extract_tool_name(wrapped)     # string name from wrapped tool hash
```

The base `call` loop: add user message → request completion → if tool calls: execute each, add results, loop → else: return content.

The client is **not** a template method: the base `Assistant` gets it from its provider (`provider.build_client`), lazily, so nothing is built until the first request. As with `Client`, `Provider#create_assistant` injects `provider: self`, and direct instantiation (`Rasti::AI::OpenRouter::Assistant.new`) falls back to the namespace derivation.

That leaves `OpenRouter::Assistant` and `HuaweiMaaS::Assistant` with **empty bodies** — like their clients, they survive as the documented public entry point per provider and as the namespace marker `Provider#assistant_class` resolves.

#### Tool — optional base class

Simple tool classes only need a `.form` class method and a `#call(params={})` instance method.

Tools inheriting from `Rasti::AI::Tool` define a nested `Form` class and an `execute(form)` method. The base `call` wraps `execute` and JSON-serializes the result — so **`call_tool` in the assistant always receives a String**.

Tool names are derived automatically via `Inflecto.underscore(Inflecto.demodulize(class_name))` — e.g. `MyApp::GetWeatherTool` → `get_weather_tool`.


## Project structure

```
lib/
  rasti-ai.rb                  # entry point, just requires rasti/ai
  rasti/
    ai.rb                      # module definition, config, loads all files
    ai/
      assistant.rb             # abstract base class (template method pattern)
      assistant_state.rb       # conversation history + tool result cache
      client.rb                # abstract HTTP client base
      tool.rb                  # optional base class for tools
      tool_serializer.rb       # converts tool classes to JSON Schema hashes
      usage.rb                 # value object for token consumption data
      errors.rb                # RequestFail, ToolSerializationError, UndefinedTool
      open_ai/
        roles.rb
        client.rb
        assistant.rb
        provider.rb
      gemini/
        roles.rb
        client.rb
        assistant.rb
        provider.rb
      anthropic/
        roles.rb
        client.rb
        assistant.rb
        provider.rb
      open_router/
        roles.rb
        client.rb
        assistant.rb
        provider.rb
      huawei_maas/
        roles.rb
        client.rb
        assistant.rb
        provider.rb
      provider.rb              # entry point: resolves provider/model, builds client + assistant, generate_text
      provider_aware.rb        # shared lazy provider lookup by namespace (Client + Assistant)
      result.rb                # normalized generate_text output (content, usage, raw)
      roles.rb                 # generic roles (user, assistant, system)
      mcp/
        server.rb              # Rack middleware exposing tools via JSON-RPC 2.0
        tools_registry.rb      # per-request tool registry used by the middleware
        client.rb              # HTTP client for MCP servers
        errors.rb

spec/
  minitest_helper.rb           # test config, shared tool class (GoalsByPlayer)
  support/helpers/
    erb.rb                     # ERB rendering helper for JSON resource templates
    resources.rb               # read_resource / read_json_resource helpers
  resources/
    open_ai/                   # ERB-templated JSON fixtures
    gemini/
    anthropic/
  open_ai/
    client_spec.rb
    assistant_spec.rb
  gemini/
    client_spec.rb
    assistant_spec.rb
  anthropic/
    client_spec.rb
    assistant_spec.rb
  open_router/
    client_spec.rb
    assistant_spec.rb
  huawei_maas/
    client_spec.rb
    assistant_spec.rb
  mcp/
    client_spec.rb
    server_spec.rb
    tools_registry_spec.rb
  tool_serializer_spec.rb
```


## Key runtime dependencies

| Gem | Role |
|---|---|
| `multi_require` | Auto-requires all files matching a glob pattern (alphabetically sorted) |
| `rasti-form` | Typed structs used as tool parameter schemas |
| `rasti-model` | Typed value objects (e.g. `Usage`) |
| `class_config` | DSL for `attr_config` on modules — provides the `Rasti::AI.configure` block |
| `inflecto` | String inflection: `underscore` and `demodulize` to derive tool names from class names |
| `net/http` | HTTP client (stdlib, no extra gem) |


## Provider API differences

| | OpenAI | Gemini | Anthropic |
|---|---|---|---|
| Auth | `Authorization: Bearer {key}` | `?key=` query param | `x-api-key: {key}` + `anthropic-version: 2023-06-01` header |
| Usage payload | `usage.prompt_tokens` / `completion_tokens` | `usageMetadata.promptTokenCount` / `candidatesTokenCount` | `usage.input_tokens` / `output_tokens` |
| Endpoint | `POST /chat/completions` | `POST /models/{model}:generateContent` | `POST /messages` |
| System prompt | message with `role: system` | top-level `system_instruction` | top-level `system` string |
| `max_tokens` | optional | optional | **required** (default 4096 in client) |
| Tool schema key | `parameters` | `parameters` | `input_schema` |
| Tool choice | `"tool_choice": "auto"` | _(not sent)_ | `"tool_choice": {"type": "auto"}` |
| Tool result role | `tool` | `function` | `user` (with content block `type: tool_result`) |
| Tool args in response | JSON string in `function.arguments` | object in `functionCall.args` | object in `input` |
| Stop signal | `choices[0].finish_reason` | `candidates[0].finishReason` | `stop_reason` |
| `json_schema` impl | native `response_format` | native `generation_config.response_schema` | forced tool use: adds `structured_output` tool + `tool_choice: {type: tool}` |

`ToolSerializer` always outputs `inputSchema`. Each provider renames it in `wrap_tool_serialization`:
- OpenAI: keeps as-is (passes `inputSchema` directly, API accepts it)
- Gemini: renames to `parameters`
- Anthropic: renames to `input_schema`

### OpenAI-compatible providers

OpenRouter and Huawei MaaS speak the OpenAI chat completions protocol verbatim — same endpoint shape (`POST /chat/completions`), same request body, same response structure, same tool calling format, same Bearer token auth. They inherit from the OpenAI provider; their whole implementation is `name` / `default_model` / `default_api_key` / `base_url` on the provider. Their clients and assistants are empty subclasses.

| | OpenRouter | Huawei MaaS |
|---|---|---|
| Base URL | `https://openrouter.ai/api/v1` | `https://api-ap-southeast-1.modelarts-maas.com/v2` |
| Auth | `Authorization: Bearer {key}` | `Authorization: Bearer {key}` |
| Config | `openrouter_api_key` / `openrouter_default_model` | `huawei_maas_api_key` / `huawei_maas_default_model` |
| `provider` in Usage | `'open_router'` | `'huawei_maas'` |

Thinking is inherited too: `reasoning_effort` is passed through as-is (same as OpenAI).


## Thinking levels

The base `Assistant` accepts `thinking: 'low' | 'medium' | 'high'` (validated on construction; nil = disabled). Each provider translates it in a private `thinking_config` method and passes the result to the client. The client includes it in the request body only if present.

| Level | OpenAI `reasoning_effort` | Anthropic `budget_tokens` | Gemini `thinking_budget` |
|---|---|---|---|
| `'low'` | `'low'` | `1_024` | `1_024` |
| `'medium'` | `'medium'` | `8_000` | `8_192` |
| `'high'` | `'high'` | `16_000` | `24_576` |

The `THINKING_LEVELS` table lives in each **`Provider`** class (it's provider metadata, not assistant behavior). `Assistant#thinking_config` reads it as `Provider::THINKING_LEVELS[thinking]` — resolved by lexical scope to the provider's own class. `Provider#thinking_config` also validates the level against the table's keys.

For Gemini, `thinking_config` goes inside `generation_config` — the client doesn't need a new param. For OpenAI and Anthropic, it's a separate top-level param in the client method (`reasoning_effort:` and `thinking:` respectively).

OpenRouter and Huawei MaaS inherit OpenAI's behavior: `thinking` is passed through as `reasoning_effort` with the same string values (`'low'`/`'medium'`/`'high'`). No `thinking_config` override is needed. Note that Huawei's models typically accept only `'high'` and `'max'` — `'high'` is the safe choice, and `'max'` is not expressible through the gem's universal levels.

The loop does not change. Anthropic thinking blocks (`type: 'thinking'`) in responses are ignored by `parse_content` (looks for `type == 'text'`) and preserved automatically by `build_assistant_tool_calls_message` (passes full `response['content']` array).


## Adding a new provider

Create four files under `lib/rasti/ai/<provider>/`:

1. **`roles.rb`** — string constants for role names
2. **`client.rb`** — inherits `Rasti::AI::Client`, implements the main API method + auth
3. **`assistant.rb`** — inherits `Rasti::AI::Assistant`, implements all 11 template methods
4. **`provider.rb`** — inherits `Rasti::AI::Provider`, implements the 7 methods + `THINKING_LEVELS`

Add to `lib/rasti/ai.rb`:
```ruby
attr_config :<provider>_api_key,       ENV['<PROVIDER>_API_KEY']
attr_config :<provider>_default_model, ENV['<PROVIDER>_DEFAULT_MODEL']
```

Add the provider to `MODULES` (and `ALIASES` if the name has a common alternative spelling) in `lib/rasti/ai/provider.rb`, and create a fourth file `provider.rb` implementing the 6 methods listed in [Provider](#provider).

The client must expose a private `default_model` returning the configured default (all providers do) and a **public** `parse_usage`.

If the new provider supports thinking, define a `THINKING_LEVELS` constant and a private `thinking_config` method (see existing providers). The base constructor already validates and exposes `thinking`.

Add an entry to the `PROVIDERS` table in `tasks/assistant.rake` so the interactive task is also available for the new provider:

```ruby
PROVIDERS = {
  # existing providers ...
  '<provider>' => {key: '<PROVIDER>_API_KEY', klass: -> { Rasti::AI::<Provider>::Assistant }}
}.freeze
```

The task name, description, env-key check, logger path and banner are all derived automatically from this entry.

Add to `spec/minitest_helper.rb`:
```ruby
config.<provider>_api_key       = 'test_<provider>_api_key'
config.<provider>_default_model = '<provider>-test'
```

### OpenAI-compatible providers

If the new provider speaks the OpenAI chat completions protocol (same endpoint, request body, response shape, tool calling, Bearer auth), inherit from `Rasti::AI::OpenAI::Client`, `Rasti::AI::OpenAI::Assistant` and `Rasti::AI::OpenAI::Provider` instead of the abstract bases:

- **client** — empty body, just the subclass declaration
- **assistant** — empty body, just the subclass declaration
- **provider** — overrides `name`, `default_model`, `default_api_key` and `base_url`; inherits `request`, `encode_message`, `parse_content`, `parse_usage` and `THINKING_LEVELS`

`Usage#provider` comes from `Provider#name`, so nothing else needs to know the provider's identity.

See `open_router/` and `huawei_maas/` for reference implementations.

### ⚠️ multi_require load order

`require_relative_pattern 'ai/**/*'` loads files alphabetically. If the new provider name sorts before `assistant` or `client` (e.g. `anthropic` < `assistant`), the subclass is loaded before the base class and raises `NameError`.

**Fix already in place**: `lib/rasti/ai.rb` explicitly requires the base classes before the pattern:

```ruby
require_relative 'ai/errors'
require_relative 'ai/usage'
require_relative 'ai/assistant_state'
require_relative 'ai/tool'
require_relative 'ai/tool_serializer'
require_relative 'ai/client'
require_relative 'ai/assistant'
require_relative 'ai/open_ai/roles'
require_relative 'ai/open_ai/client'
require_relative 'ai/open_ai/assistant'
require_relative_pattern 'ai/**/*'   # duplicates are skipped by Ruby's require
```

The OpenAI provider is also explicitly required because OpenAI-compatible providers (OpenRouter, Huawei MaaS) inherit from it, and some of their directory names sort before `open_ai` alphabetically (e.g. `huawei_maas` < `open_ai`).

If you add a provider whose name sorts before `client` alphabetically, the same mechanism protects it.


## Code conventions

### Constants

Constants are always defined at the **top of the class body, before `private`**. Never inside the `private` section or between method definitions.

```ruby
class Client < Rasti::AI::Client

  ANTHROPIC_VERSION  = '2023-06-01'.freeze
  DEFAULT_MAX_TOKENS = 4096

  private

  def base_url ...
end
```

This also applies to per-provider constants in `Assistant` subclasses (`THINKING_LEVELS`, `ALLOWED_SCHEMA_FIELDS`, etc.).

### Frozen strings

All string constants use `.freeze`. Integer and array/hash literals that are already frozen by `%w[]`/`.freeze` on the outer value don't need it again on the inner elements. Integers never need `.freeze`.

```ruby
USER = 'user'.freeze
ASSISTANT = 'assistant'.freeze

VALID_THINKING_LEVELS = %w[low medium high].freeze

THINKING_LEVELS = {
  'low' => {thinking_budget: 1_024}.freeze,
  'medium' => {thinking_budget: 8_192}.freeze,
}.freeze
```

### Building request bodies

Start with the required keys, then conditionally add optional ones. Never include optional fields as `nil`.

```ruby
body = {
  model: model || Rasti::AI.anthropic_default_model,
  max_tokens: max_tokens || DEFAULT_MAX_TOKENS,
  messages: messages
}

body[:thinking] = thinking if thinking
body[:system] = system if system
body[:tools] = tools unless tools.empty?
body[:tool_choice] = tool_choice if tool_choice
```

### Hash alignment

Use `key: value` without padding spaces. Do not align values across keys:

```ruby
{
  model: model,
  max_tokens: DEFAULT_MAX_TOKENS,
  messages: messages
}
```

Nested hashes always go on their own lines, indented one level. Never inline a multi-key hash next to its parent key or inside an array bracket:

```ruby
# Preferred
{
  role: Roles::USER,
  content: [
    {
      type: 'tool_result',
      tool_use_id: tool_call['id'],
      content: result
    }
  ]
}

# Avoid
{
  role:    Roles::USER,
  content: [{
    type:        'tool_result',
    tool_use_id: tool_call['id'],
    content:     result
  }]
}
```

Exception: a single-key nested hash that fits naturally on one line can stay inline (e.g. `THINKING_LEVELS` entries). Apply judgment — the goal is always readability.

### Parentheses

Omit parentheses in method calls when they add no clarity — particularly in single-argument calls, `if`/`unless` conditions, and DSL-style invocations:

```ruby
# Preferred
raise NotImplementedError
attr_reader :client
puts response

# Avoid
raise(NotImplementedError)
attr_reader(:client)
puts(response)
```

Include parentheses when the call uses splat, double-splat, or block arguments (`*`, `**`, `&`), or when omitting them causes ambiguity in a complex expression:

```ruby
# Required — splat/block args
object.forward(*args, &block)

# Required — disambiguate argument boundary in compound conditions
if tool && tool.class.respond_to?(:form)  # correct
if tool && tool.class.respond_to? :form   # wrong: :form is parsed as arg to &&

# Required — call is the value of a hash key or inside an array literal with a constant arg
# (Ruby 2.3 parser raises SyntaxError: unexpected tCONSTANT)
{ inputSchema: ToolSerializer.serialize_form(SumTool::Form) }   # correct
{ inputSchema: ToolSerializer.serialize_form SumTool::Form }    # wrong: SyntaxError in Ruby 2.3
[ToolSerializer.serialize(HelloWorldTool)]                      # correct
[ToolSerializer.serialize HelloWorldTool]                       # wrong: SyntaxError in Ruby 2.3

# Required — call is an intermediate argument in a multi-arg call
# (without parens the outer parser greedily passes subsequent args to the inner call)
post path, JSON.dump(body), 'CONTENT_TYPE' => 'application/json'   # correct
post path, JSON.dump body, 'CONTENT_TYPE' => 'application/json'    # wrong: 'CONTENT_TYPE' => ... is passed to JSON.dump

# Fine without parentheses — multiple regular args
http.post '/path', body: '{}'
calc.sum 1, 2
```

### `private` and `attr_reader`

`private` is placed immediately after the class-level constants (if any) and before all method definitions. `attr_reader` always lives inside the `private` section — never in the public interface unless the attribute is intentionally public (e.g. `state`, `model`, `thinking` on `Assistant`).

```ruby
class Assistant < Rasti::AI::Assistant

  THINKING_LEVELS = { ... }.freeze

  private

  attr_reader :client, :json_schema, :tools, :serialized_tools, :logger

  def build_default_client ...
end
```

### Instance variables (`@`)

Avoid bare `@variable` references in method bodies. Always declare an `attr_reader` inside the `private` section, then access the attribute by its reader name throughout the class. Direct `@` usage is only acceptable inside `initialize` assignments and inside the writer itself.

```ruby
# Bad — @session_id scattered across methods
def request_with_session(method, params={})
  raise unless e.message =~ /session/i && @session_id.nil?
end

# Good — declared once, accessed via reader everywhere
private

attr_reader :session_id

def request_with_session(method, params={})
  raise unless e.message =~ /session/i && session_id.nil?
end

def initialize_session
  @session_id = response['mcp-session-id']  # @ only for assignment
end
```

### Keyword arguments

All method signatures use keyword arguments. Required params have no default; optional params default to `nil`, `[]`, `{}`, or `false` as appropriate.

```ruby
def messages(messages:, model:nil, system:nil, tools:[], tool_choice:nil, thinking:nil)
```

### Template methods

Abstract methods in base classes always raise `NotImplementedError` with no message. Do not use `raise NotImplementedError, "override me"` — the bare form is the convention.

```ruby
def build_default_client
  raise NotImplementedError
end
```

### No unused constants

Don't leave constants defined unless they are referenced in the same file. If a constant was added in anticipation of future use, remove it until it's actually needed.

### Ruby compatibility

The gem must run on Ruby **2.3 and later**. Do not use language features or stdlib methods introduced after 2.3. Common pitfalls:

| Avoid | Use instead |
|---|---|
| `hash.transform_keys { \|k\| ... }` | `Hash[hash.map { \|k, v\| [transform(k), v] }]` |
| `hash.filter { ... }` | `hash.select { ... }` |
| Numbered block params (`_1`, `_2`) | Named block params (`\|k, v\|`) |
| Pattern matching (`case/in`) | `case/when` or conditionals |
| `Array#sum` with initial value | `inject(:+)` or `reduce` |
| `Hash#slice` | `select` + key check |

When in doubt, check the Ruby 2.3 docs or test against the lowest version in the CI matrix.


## Test conventions

### Framework and libraries

- **Minitest** with `describe`/`it` blocks (spec style)
- **WebMock** for HTTP stubbing — real connections are disabled in all tests
- **Minitest::Mock** for mock objects

### JSON fixtures with ERB

Request/response bodies live in `spec/resources/<provider>/` as ERB-templated JSON files. Use `read_resource` to render them:

```ruby
read_resource('anthropic/basic_request.json', model: model, prompt: question)
# => '{"model":"claude-test","max_tokens":4096,...}'

read_json_resource('anthropic/basic_response.json', content: answer)
# => parsed Ruby hash
```

Template example (`basic_request.json`):
```
{"model":"<%= model %>","max_tokens":4096,"messages":[{"role":"user","content":"<%= prompt %>"}]}
```

Variables are set via `binding.local_variable_set` so any Ruby expression works inside `<%= %>`.

### Stubbing HTTP requests

```ruby
stub_request(:post, 'https://api.anthropic.com/v1/messages')
  .with(
    headers: {'x-api-key' => Rasti::AI.anthropic_api_key},
    body: read_resource('anthropic/basic_request.json', model: model, prompt: question)
  )
  .to_return(body: read_resource('anthropic/basic_response.json', content: answer))
```

Body matching is a string comparison, so JSON key order in the fixture must match exactly what the client sends (`JSON.dump` of a Ruby hash with symbol keys produces alphabetical-ish order based on insertion).

### Testing multi-turn tool flows

For tests involving tool calls (where the model calls a tool then continues), use `Minitest::Mock` on the client instead of HTTP stubs — it's simpler to set up multiple sequential responses:

```ruby
let(:client) { Minitest::Mock.new }

client.expect :messages, tool_response do |params|
  params[:messages].last[:role] == 'user' &&
    params[:messages].last[:content] == question
end

client.expect :messages, basic_response(answer) do |params|
  params[:messages].last[:content] == [{type: 'tool_result', ...}]
end

assistant = Rasti::AI::Anthropic::Assistant.new client: client, tools: [tool]
assistant.call question
client.verify
```

The block form of `expect` is used (instead of the positional args form) because keyword argument matching is more reliable with it across Ruby versions.

### Shared test tool

`GoalsByPlayer` is defined in `minitest_helper.rb` and used across all provider assistant specs:

```ruby
class GoalsByPlayer
  def self.form
    Rasti::Form[player: Rasti::Types::String, team: Rasti::Types::String]
  end

  def call(params={})
    '672'
  end
end
```

It's a simple class (not a `Tool` subclass) that returns a plain string — covering the most common tool interface.


## Development setup

- The minimum supported Ruby version is **2.3** (`required_ruby_version >= 2.3` in the gemspec). All code must run on 2.3 and every version up through the CI matrix ceiling. See [Ruby compatibility](#ruby-compatibility) in Code conventions.
- **Run tests**: `bundle exec rake spec`
- **Run a single file**: `bundle exec rake spec TEST=spec/anthropic/client_spec.rb`
- **Run a single test by line**: `bundle exec rake spec TEST=spec/anthropic/client_spec.rb:42`
- **Run by name**: `bundle exec rake spec NAME=tool`
- **Console**: `bundle exec rake console` (loads the gem + Pry)
- **Interactive chat** (requires provider API key in env):
  ```
  rake assistant:openai      # OPENAI_API_KEY
  rake assistant:gemini      # GEMINI_API_KEY
  rake assistant:anthropic   # ANTHROPIC_API_KEY
  rake assistant:openrouter  # OPENROUTER_API_KEY
  rake assistant:huawei_maas # HUAWEI_MAAS_API_KEY
  ```
  Each task validates the key, writes logs to `log/<provider>.log`, connects to the [Pipeworx](https://pipeworx.io) public weather MCP server, and starts a `You:` / `Assistant:` prompt loop (`exit` or `Ctrl+C` to quit). The model can be overridden with the matching env variable (e.g. `OPENAI_DEFAULT_MODEL=gpt-4o`).


## ToolSerializer

`ToolSerializer.serialize_form(form_class)` delegates to `form_class.to_schema` (`Rasti::Model::Schema.serialize`) and converts the result to JSON Schema via two private methods:

- **`json_schema_from_model_schema(schema)`** — takes the full model schema hash (`{model:, attributes: [...]}`) and builds a JSON Schema `{type: 'object', properties: {...}, required: [...]}`. Required attributes are those with `options[:required]` truthy.
- **`json_schema_for_type(type_hash)`** — maps a single model-schema type hash to its JSON Schema equivalent. Recursive for `:array` (via `items`) and `:model` (delegates back to `json_schema_from_model_schema`).

Type mapping:

| Model schema type | JSON Schema |
|---|---|
| `:string`, `:symbol` | `{type: 'string'}` |
| `:integer` | `{type: 'integer'}` |
| `:float` | `{type: 'number'}` |
| `:boolean` | `{type: 'boolean'}` |
| `:time` | `{type: 'string', format: 'date'}` |
| `:enum` | `{type: 'string', enum: [...]}` |
| `:array` | `{type: 'array', items: ...}` (recursive) |
| `:model` | `{type: 'object', properties: ...}` (recursive) |
| `:hash` | `{type: 'object'}` |
| `:any`, `:unknown`, anything else | `{}` (no constraints, no crash) |

### Extension points

Because serialization goes through `Rasti::Model::Schema`, both extension mechanisms it provides work automatically:

- **`Rasti::Model::Schema.register_type_serializer(type, serialized_type=nil, &block)`** — registers a mapping for a custom type class. The returned type hash goes through `json_schema_for_type` like any other.
- **`to_schema` duck-typing** — if the type itself responds to `to_schema`, `Rasti::Model::Schema` calls it and uses the result. No changes needed in `ToolSerializer`.

Custom type registrations should live in the application (e.g. an initializer), not in `rasti-ai`.

## MCP Server

`Rasti::AI::MCP::Server` is a Rack middleware with a class-level DSL backed by `ClassConfig`. Two independent config blocks drive its behavior:

| Config | DSL method | Role |
|---|---|---|
| `authenticator` | `authenticate(&block)` | Optional auth check; runs in `handle_mcp_request` before the body is read |
| `tools_loader` | `load_tools(&block)` | Per-request tool registration; runs inside `handle_mcp_request` after the body is parsed |

### Request flow

`call`:
1. Path + method check — non-MCP requests pass through to `app` unchanged
2. `handle_mcp_request`

`handle_mcp_request`:
1. Auth check — if `authenticator` is set and returns falsy → return `unauthorized_response` (HTTP 401), stop
2. Read and parse body
3. `build_tools_registry`, dispatch by `data['method']`

### Auth response format

Auth is checked before the body is parsed, so `id` is `nil`. The MCP spec states that HTTP-level errors (e.g. 403 for invalid Origin) MAY include a JSON-RPC error body with no `id`; the same pattern applies to 401:

```
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{"jsonrpc":"2.0","id":null,"error":{"code":-32002,"message":"Unauthorized"}}
```

Error code `-32002` maps to `JSON_RPC_SERVER_UNAUTHORIZED` defined in `mcp/constants.rb`.

### Why auth goes inside `handle_mcp_request`

`handle_mcp_request` owns the complete request lifecycle: auth, parse, dispatch. Keeping the check there makes the method the single place to read when reasoning about what happens to an incoming MCP request. `call` stays minimal — just routing.

The check is inlined, consistent with how other error responses are built in the class:

```ruby
if !authorized? request
  response = error_response nil, JSON_RPC_SERVER_UNAUTHORIZED, 'Unauthorized'
  return [401, {'Content-Type' => 'application/json'}, [JSON.dump(response)]]
end
```

Nota: `authorized?` usa `authenticator.call(request)` con paréntesis. En Ruby 2.3, `proc.call arg` sin paréntesis con un solo argumento genera `SyntaxError` — el parser espera `end` después de `call` y ve el identificador suelto. Con múltiples argumentos (`tools_loader.call tools_registry, request`) funciona porque la coma actúa de separador.

## CI

GitHub Actions — `.github/workflows/ci.yml`.

- **Matrix**: Ruby `2.3` through `3.3` + `jruby-9.4` (JRuby 9.4 = Ruby 3.1 compat)
- **`ruby/setup-ruby@v1`** with `bundler-cache: true` — handles `bundle install` and caching automatically, no separate install step needed
- **No native extensions** — do not add `libcurl4-openssl-dev` or `force_ruby_platform` steps; this gem uses only `net/http` (stdlib)
- **`required_ruby_version`** is set to `>= 2.3` in the gemspec, consistent with the matrix

> If adding Ruby versions beyond 3.3, verify that the dev dependencies (`rake ~> 12.0`, `minitest ~> 5.0, < 5.11`) still install. If they don't, the gemspec constraints will need updating.

---
> Source: [gabynaiman/rasti-ai](https://github.com/gabynaiman/rasti-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
