## axn

> Design and implement **Axn**, a clean, step-based DSL library for defining actions with parameter validation, authorization, telemetry, and custom business logic. The DSL should prioritize simplicity, explicitness, and ease of following the execution flow.

# Axn - Action DSL Design Specifications

## Overview

Design and implement **Axn**, a clean, step-based DSL library for defining actions with parameter validation, authorization, telemetry, and custom business logic. The DSL should prioritize simplicity, explicitness, and ease of following the execution flow.

## Goals

- **Explicit over implicit**: Each action should clearly show its execution flow
- **Composable**: Steps should be reusable across actions and modules
- **Safe by default**: Telemetry and error handling should not leak sensitive data
- **Simple to implement**: Minimal macro magic, straightforward execution model
- **Easy to test**: Steps should be pure functions that are easy to unit test
- **Familiar patterns**: Should feel natural to Elixir developers

## Core Concepts

### 1. Actions
Actions are named units of work that execute a series of steps in order. Each action automatically gets telemetry wrapping and error handling.

### 2. Steps
Steps are individual functions that take a context and either continue the pipeline or halt it. Steps follow a simple contract: `(ctx, opts) -> {:cont, new_ctx} | {:halt, result}`.

### 3. Context
An `Axn.Context` struct that flows through the step pipeline, carrying request data, user information, and any step-added fields. Provides helper functions similar to `Plug.Conn` and `Phoenix.Component`.

## DSL Syntax Specification

### Module Declaration
```elixir
defmodule MyApp.UserActions do
  use Axn
  # OR with custom metadata:
  # use Axn, metadata: &__MODULE__.telemetry_metadata/1

  # Actions defined here...
end
```

### Action Declaration
```elixir
action :action_name do
  step :step_name
  step :step_name, option: value
  step {ExternalModule, :external_step}, option: value

  # Local step implementation
  def step_name(ctx) do
    # Implementation
  end
end

# Action with custom telemetry metadata
action :action_name, metadata: &metadata_function/1 do
  step :step_name
  # ...
end
```

### Step Declaration
```elixir
# Built-in step with schema only
step :cast_validate_params, schema: %{field!: :type, field2: [field: :type, default: value]}

# Built-in step with schema and custom validation
step :cast_validate_params, schema: %{phone!: :string}, validate: &validate_phone_number/1

# Built-in authorization step
step :authorize, &authorization_function/1

# Custom step in same module
step :my_custom_step

# External step
step {ExternalModule, :external_step}, option: value
```

## Context Specification

### Axn.Context Struct
```elixir
defstruct [
  :action,        # atom() - Current action name
  assigns: %{},   # map() - Phoenix-style assigns (includes current_user, etc.)
  params: %{},    # map() - Cast and validated parameters
  private: %{},   # map() - Internal DSL state (raw_params, changeset, etc.)
  result: nil     # any() - Action result
]

# Example context (before parameter validation)
%Axn.Context{
  action: :create_user,
  assigns: %{current_user: %User{id: 123}},
  params: %{"email" => "user@example.com", "name" => "John"},  # Raw params initially
  private: %{},
  result: nil
}

# Example context (after parameter validation with cast_validate_params)
%Axn.Context{
  action: :create_user,
  assigns: %{current_user: %User{id: 123}},
  params: %{email: "user@example.com", name: "John"},  # Cast and validated params
  private: %{raw_params: %{"email" => "user@example.com", "name" => "John"}, changeset: #Changeset<>},
  result: nil
}
```

### Context Helper Functions
```elixir
# Assign values (like Phoenix.Component.assign/3)
Context.assign(ctx, :current_user, user)
Context.assign(ctx, %{current_user: user, theme: "dark"})
Context.assign(ctx, current_user: user, theme: "dark")  # Phoenix LiveView style

# Get private values
Context.get_private(ctx, :correlation_id)           # Returns value or nil
Context.get_private(ctx, :correlation_id, "default") # Returns value or default

# Put private values (like Plug.Conn.put_private/3)
Context.put_private(ctx, :correlation_id, id)
Context.put_private(ctx, :changeset, changeset)

# Update params
Context.put_params(ctx, validated_params)

# Update result
Context.put_result(ctx, {:ok, user})
```

**Note**: `Context` is automatically aliased when using `use Axn`, so you don't need to write `Axn.Context`.

### Context Evolution
- Steps may modify context using helper functions or direct struct updates
- Steps should never remove core fields (action, assigns, params, private, result)
- Follow Phoenix conventions: user data goes in `assigns`, internal state in `private`
- **Parameter Flow**: Raw params start in `ctx.params`. When `cast_validate_params` runs, raw params are preserved in `ctx.private.raw_params` and cast params replace `ctx.params`
- Common private fields: `:raw_params` (after validation), `:changeset`, `:correlation_id`
- The `result` field holds the final action result
- Use helper functions for consistency: `assign/2`, `assign/3`, `get_private/2`, `get_private/3`, `put_private/3`, `put_params/2`, etc.

## Step Specification

### Function Signatures
```elixir
# Signature 1: Context only
step_function(%Axn.Context{} = ctx) -> step_result

# Signature 2: Context with options
step_function(%Axn.Context{} = ctx, opts) -> step_result
```

### Return Value Contract
```elixir
step_result =
  {:cont, %Axn.Context{}} |          # Continue pipeline with updated context
  {:halt, {:ok, result}} |           # Stop pipeline with success result
  {:halt, {:error, reason}}          # Stop pipeline with error

# {:cont, new_ctx} - Continue to next step with updated context
# {:halt, {:ok, result}} - Stop pipeline and set ctx.result = {:ok, result}
# {:halt, {:error, reason}} - Stop pipeline and set ctx.result = {:error, reason}
```

### Step Implementation Rules
1. **Always return the specified tuple format**
2. **Never mutate the input context** - create new context maps
3. **Handle errors gracefully** - return `{:halt, {:error, reason}}` instead of raising
4. **Be pure when possible** - avoid hidden side effects
5. **Document side effects** when they exist
6. **Validate options** at the beginning of the function

### Built-in Steps

#### :cast_validate_params
```elixir
step :cast_validate_params, schema: schema, validate: validation_function

# schema = %{
#   field_name!: :type,                    # Required field
#   field_name: :type,                     # Optional field
#   field_name: [field: :type, default: value], # With default
#   field_name: [field: :type, cast: &func/1]   # With custom cast function
# }
#
# validation_function = (changeset) -> changeset  # Optional custom validation
```

**Behavior**: Cast raw_params according to schema, then apply optional custom validation function. Updates `ctx.params` and `ctx.changeset`.

**Success**: `{:cont, ctx |> Axn.Context.put_params(cast_params) |> Axn.Context.put_private(:changeset, final_changeset)}`

**Failure**: `{:halt, {:error, %{reason: :invalid_params, changeset: changeset}}}`

#### Authorization Patterns (No Built-in Step)
Authorization is too application-specific for a built-in step. Instead, use simple custom steps following these patterns:

```elixir
# Pattern 1: Simple role check
step :require_admin

def require_admin(ctx) do
  if admin?(ctx.assigns.current_user) do
    {:cont, ctx}
  else
    {:halt, {:error, :unauthorized}}
  end
end

# Pattern 2: Resource-based authorization  
step :authorize_user_access

def authorize_user_access(ctx) do
  if can_access?(ctx.assigns.current_user, ctx.params.user_id) do
    {:cont, ctx}
  else
    {:halt, {:error, :unauthorized}}
  end
end

# Pattern 3: Action-based authorization
step :authorize_action

def authorize_action(ctx) do
  if allowed?(ctx.assigns.current_user, ctx.action) do
    {:cont, ctx}
  else
    {:halt, {:error, :unauthorized}}
  end
end
```

**Standard Error Response**: Always use `{:halt, {:error, :unauthorized}}` for authorization failures.

## Telemetry Integration

### Automatic Telemetry Events
Every action automatically emits standard telemetry events:

```elixir
# Fixed event names - no configuration needed
[:axn, :action, :start]     # When action starts
[:axn, :action, :stop]      # When action completes  
[:axn, :action, :exception] # When action fails with exception
```

### Event Metadata
Telemetry events always include metadata in this precedence order:

1. **Default metadata** (always included):
   ```elixir
   %{
     module: MyApp.UserActions,  # The action module
     action: :create_user,       # The action name
     duration: 45                # Only on :stop events (milliseconds)
   }
   ```

2. **Module-level metadata** (merged next, if provided)
3. **Action-level metadata** (merged last, overrides duplicates)

### Custom Metadata (Optional)
Add custom metadata at the module level and/or action level:

```elixir
defmodule MyApp.UserActions do
  use Axn, metadata: &__MODULE__.module_metadata/1

  # Module-level metadata (applies to all actions)
  def module_metadata(ctx) do
    %{
      user_id: ctx.assigns.current_user && ctx.assigns.current_user.id,
      tenant: ctx.assigns.tenant && ctx.assigns.tenant.slug
    }
  end

  # Action-specific metadata
  action :create_user, metadata: &action_metadata/1 do
    step :validate_params
    step :create_user
  end

  def action_metadata(ctx) do
    %{
      resource_type: :user,
      creation_method: ctx.params[:method] || "standard"
    }
  end
end

# Final merged metadata (precedence: default → module → action):
# %{
#   module: MyApp.UserActions,        # Default
#   action: :create_user,             # Default  
#   duration: 45,                     # Default (on :stop events only)
#   user_id: 123,                     # Module-level
#   tenant: "acme",                   # Module-level
#   resource_type: :user,             # Action-level
#   creation_method: "standard"       # Action-level
# }
```

**Metadata Precedence**: Action-level metadata is merged with module-level metadata, with action-level taking precedence for duplicate keys.

**Security**: Both metadata functions let you control exactly what data is included. Only include safe, non-sensitive information in telemetry events.

## Implementation Requirements

### 1. Macro System
- `use Axn` - Sets up the module
- `action/2` macro - Defines an action with options
- `step/1` and `step/2` macros - Adds steps to current action
- `@before_compile` hook - Generates the execution functions

### 2. Generated Functions
Each module using the DSL must generate:

```elixir
# Primary entry point - extracts result from context
def run(action_name, assigns, raw_params) -> {:ok, result} | {:error, reason}

# Internal pipeline runner
defp run_step_pipeline(steps, %Axn.Context{} = ctx) -> %Axn.Context{}  # Always returns context

# Step application
defp apply_step(step_name, %Axn.Context{} = ctx, opts) -> {:cont, %Axn.Context{}} | {:halt, result}

# Telemetry wrapper
defp run_action_with_telemetry(%Axn.Context{} = ctx, steps) -> %Axn.Context{}
```

### 3. Error Handling
- Pipeline stops on first `{:halt, result}`
- Final context always has result set (either from successful completion or halt)
- Telemetry spans are properly ended even on errors
- Errors should preserve original error information
- No sensitive data in error messages that might be logged

### 4. Module Attribute Tracking
During compilation, track:
- `@actions` - List of `{action_name, steps, action_opts}`
- `@telemetry_metadata_fn` - Optional module-level metadata function
- `@current_action` and `@steps` - For building actions during macro expansion

## Built-in Steps Implementation

### Parameters Module Integration
The `:cast_validate_params` step should integrate with any parameter validation system:

```elixir
def cast_validate_params(%Axn.Context{} = ctx, opts) do
  schema = Keyword.fetch!(opts, :schema)
  validate_fn = Keyword.get(opts, :validate)
  raw_params = ctx.private.raw_params

  # Works with any params library that implements this interface
  case ParamsModule.cast_and_validate(raw_params, schema) do
    {:ok, params, changeset} ->
      # Apply optional custom validation
      final_changeset = if validate_fn do
        validate_fn.(changeset)
      else
        changeset
      end

      if final_changeset.valid? do
        new_ctx = ctx
          |> Axn.Context.put_params(Ecto.Changeset.apply_changes(final_changeset))
          |> Axn.Context.put_private(:changeset, final_changeset)
        {:cont, new_ctx}
      else
        {:halt, {:error, %{reason: :invalid_params, changeset: final_changeset}}}
      end
    {:error, changeset} ->
      {:halt, {:error, %{reason: :invalid_params, changeset: changeset}}}
  end
end
```

## Testing Patterns

### Testing Individual Steps
```elixir
defmodule MyStepsTest do
  use ExUnit.Case

  describe "cast_validate_params/2" do
    test "successfully casts valid params" do
      ctx = %Axn.Context{private: %{raw_params: %{"name" => "John", "age" => "25"}}}
      opts = [schema: %{name!: :string, age: :integer}]

      assert {:cont, updated_ctx} = Steps.cast_validate_params(ctx, opts)
      assert updated_ctx.params == %{name: "John", age: 25}
      assert updated_ctx.private.changeset.valid?
    end

    test "applies custom validation function" do
      ctx = %Axn.Context{private: %{raw_params: %{"phone" => "+1234567890", "region" => "US"}}}
      validate_fn = fn changeset ->
        cast_validate_phone_number(changeset, :phone, region: get_field(changeset, :region))
      end
      opts = [schema: %{phone!: :string, region: :string}, validate: validate_fn]

      assert {:cont, updated_ctx} = Steps.cast_validate_params(ctx, opts)
      assert updated_ctx.private.changeset.valid?
    end

    test "halts on invalid params" do
      ctx = %Axn.Context{private: %{raw_params: %{"age" => "not_a_number"}}}
      opts = [schema: %{name!: :string, age!: :integer}]

      assert {:halt, {:error, %{reason: :invalid_params, changeset: changeset}}} = Steps.cast_validate_params(ctx, opts)
      refute changeset.valid?
    end
  end
end
```

### Testing Actions
```elixir
defmodule MyActionsTest do
  use ExUnit.Case

  test "request_otp action succeeds with valid input" do
    assigns = %{current_user: %User{id: 123}}
    params = %{"phone" => "+1234567890", "challenge_token" => "abc123"}

    # Test using run/3 for simple result checking
    assert {:ok, %{message: "OTP sent"}} = MyActions.run(:request_otp, params, assigns)
  end

  test "request_otp action returns full context" do
    assigns = %{current_user: %User{id: 123}}
    params = %{"phone" => "+1234567890", "challenge_token" => "abc123"}

    # With simplified API, just test the result
    assert {:ok, %{message: "OTP sent"}} = MyActions.run(:request_otp, params, assigns)
  end

  test "request_otp action fails authorization" do
    assigns = %{current_user: nil}  # No user
    params = %{"phone" => "+1234567890", "challenge_token" => "abc123"}

    # Test failure using run/3
    assert {:error, :unauthorized} = MyActions.run(:request_otp, params, assigns)
  end
end
```

## Example Implementation Structure

```
# Axn Library Structure
lib/
├── axn/
│   ├── axn.ex                     # Main macro module
│   ├── context.ex                 # Context struct and helper functions
│   ├── steps.ex                   # Built-in steps
│   └── telemetry.ex               # Telemetry helpers (optional)

# Using Axn in Your Application
lib/
├── my_app/
│   ├── accounts/
│   │   ├── user_actions.ex        # Example: UserActions using Axn
│   │   └── auth_actions.ex        # Example: AuthActions using Axn
│   ├── billing/
│   │   └── payment_actions.ex     # Example: PaymentActions using Axn
│   └── custom_steps/
│       ├── validation_steps.ex    # Your custom validation steps
│       └── business_steps.ex      # Your custom business logic steps

test/
├── support/
│   └── telemetry_helper.ex       # Simple telemetry test helper
├── axn/
│   ├── axn_test.exs              # Test core DSL functionality
│   └── steps_test.exs            # Test built-in steps
│   └── telemetry_test.exs        # Test telemetry integration (simplified)
└── my_app/
    ├── accounts/
    │   └── user_actions_test.exs  # Test your action modules
    └── custom_steps/
        └── validation_steps_test.exs  # Test your custom steps
```

## Installation

```elixir
# mix.exs
def deps do
  [
    {:axn, "~> 1.0"}
  ]
end
```

## Basic Usage

```elixir
# lib/my_app/user_actions.ex
defmodule MyApp.UserActions do
  use Axn, metadata: &__MODULE__.telemetry_metadata/1

  action :create_user, metadata: &create_user_metadata/1 do
    step :cast_validate_params, schema: %{email!: :string, name!: :string}
    step :require_admin
    step :handle_create

    def require_admin(ctx) do
      if admin?(ctx.assigns.current_user) do
        {:cont, ctx}
      else
        {:halt, {:error, :unauthorized}}
      end
    end

    def handle_create(ctx) do
      case Users.create(ctx.params) do
        {:ok, user} -> {:halt, {:ok, user}}
        {:error, changeset} -> {:halt, {:error, %{reason: :creation_failed, changeset: changeset}}}
      end
    end

    def telemetry_metadata(ctx) do
      %{
        user_id: ctx.assigns.current_user && ctx.assigns.current_user.id,
        tenant: ctx.assigns.tenant && ctx.assigns.tenant.slug
      }
    end

    def create_user_metadata(ctx) do
      %{
        admin_creation: admin?(ctx.assigns.current_user),
        email_domain: ctx.params[:email] |> String.split("@") |> List.last()
      }
    end

    defp admin?(user), do: user && user.role == "admin"
  end
end

# In your Phoenix controller
defmodule MyAppWeb.UserController do
  use MyAppWeb, :controller

  def create(conn, params) do
    case MyApp.UserActions.run(:create_user, params, conn.assigns) do
      {:ok, user} ->
        json(conn, %{success: true, user: user})
      {:error, %{reason: :invalid_params, changeset: changeset}} ->
        json(conn, %{errors: format_changeset_errors(changeset)})
      {:error, reason} ->
        json(conn, %{error: reason})
    end
  end
end
```

## Implementation Checklist (TDD Approach)

### Phase 1: Foundation & Basic Tests ✅ **COMPLETED**
- [x] Create basic project structure with `mix new axn --lib`
- [x] Write failing tests for `Axn.Context` struct and helper functions
- [x] Implement `Axn.Context` module to make tests pass
- [x] Write failing tests for basic DSL compilation (empty actions)
- [x] Implement minimal `Axn` macro module with `__using__/1` to make tests pass

### Phase 2: Core DSL with TDD ✅ **COMPLETED**
- [x] Write failing tests for `action/2` macro functionality
- [x] Implement `action/2` macro that collects steps
- [x] Write failing tests for `step/1` and `step/2` macros
- [x] Implement `step/1` and `step/2` macros
- [x] Write failing tests for basic step pipeline execution
- [x] Implement `@before_compile` hook and basic step pipeline with `Enum.reduce_while/3`

### Code Optimization & Simplification ✅ **COMPLETED**
- [x] Consolidate duplicated function resolution logic in `apply_step/2`
- [x] Simplify `Context.assign/2` by reusing map handling for keyword lists
- [x] Streamline result processing in `run/3` function
- [x] Maintain all 30 passing tests through optimizations

### Phase 3: Built-in Steps with TDD ✅ **COMPLETED**
- [x] Write comprehensive failing tests for `:cast_validate_params` step
- [x] Implement `:cast_validate_params` step with schema validation and optional custom validation function
- [x] Write failing tests for step resolution (local vs external modules)
- [x] Implement step resolution functionality
- [x] Document authorization patterns instead of built-in `:authorize` step (simplified approach)

### Phase 4: Error Handling with TDD ✅ **COMPLETED**
- [x] Write failing tests for proper error propagation through pipeline
- [x] Implement error propagation mechanisms
- [x] Write failing tests for clear error messages without sensitive data
- [x] Implement error message sanitization (simplified approach using external libraries)
- [x] Write failing tests for graceful handling of step exceptions
- [x] Implement exception handling in step pipeline

### Phase 5: Telemetry Integration with TDD ✅ **COMPLETED**
- [x] Write failing tests for automatic telemetry span wrapping (simplified to use standard `:telemetry.span/3`)
- [x] Implement telemetry span functionality using standard patterns
- [x] Write failing tests for configurable event naming (simplified to use telemetry_prefix directly)
- [x] Implement event naming configuration
- [x] Write failing tests for safe metadata extraction (simplified to 3-field allowlist)
- [x] Implement safe metadata extraction with allowlist approach  
- [x] Write failing tests for telemetry reliability (simplified, removed complex concurrency tests)
- [x] Implement telemetry handling with standard patterns

**Simplification Notes**: 
- Removed complex test setup boilerplate in favor of `TelemetryHelper` module
- Switched from custom `:start/:stop/:exception` events to standard `:telemetry.span/3` pattern
- Replaced "security theater" (testing absence of 15+ fields) with simple 3-field allowlist
- Eliminated duration duplication and unnecessary concurrency testing
- Reduced test count from 20 complex tests to 9 focused tests

### Phase 6: Integration Tests & Polish
- [ ] Write end-to-end integration tests using example action modules
- [ ] Create comprehensive test suite covering all usage patterns from specification
- [ ] Write performance benchmarks and optimization tests
- [ ] Create test helpers for easier step testing in downstream applications
- [ ] Write property-based tests for edge cases and error conditions

## Security Considerations

1. **Telemetry Metadata**: Never include raw params, changeset errors, or assigns by default
2. **Error Messages**: Sanitize error details that might be logged or returned to clients
3. **Rate Limiting**: Ensure rate limit keys don't leak user data
4. **Authorization**: Always fail closed - unauthorized by default

## Performance Considerations

1. **Step Pipeline**: Use `Enum.reduce_while/3` for early termination
2. **Telemetry**: Minimal overhead when telemetry is disabled
3. **Context Size**: Keep context maps reasonably sized
4. **Memory**: Don't accumulate large data structures in context

## Extension Points

### Custom Steps
Users should be able to define steps in:
1. The same module as the action
2. External modules
3. Third-party libraries

### Custom Telemetry
Telemetry is automatic and safe by default. Advanced users can:
1. Subscribe to telemetry events and process them further
2. Add custom telemetry as steps within actions

### Custom Parameter Systems
The Axn library should work with any parameter validation system that implements:
```elixir
cast_and_validate(raw_params, schema) ->
  {:ok, params, changeset} | {:error, changeset}
```

The `:cast_validate_params` step extends this with optional custom validation:
```elixir
step :cast_validate_params, schema: schema, validate: validation_fn

# validation_fn receives the changeset after initial cast/validation
# and can apply additional custom validation logic
```

## Example Usage Patterns

### Simple Action
```elixir
action :ping do
  step :handle_ping

  def handle_ping(ctx) do
    {:cont, Context.put_result(ctx, "pong")}
  end
end
```

### Complex Action
```elixir
action :create_user do
  step :cast_validate_params, schema: %{email!: :string, name!: :string, phone: :string}
  step :require_admin
  step :validate_business_rules
  step :handle_create_user

  def require_admin(ctx) do
    if admin?(ctx.assigns.current_user) do
      {:cont, ctx}
    else
      {:halt, {:error, :unauthorized}}
    end
  end

  def validate_business_rules(ctx) do
    # Custom validation logic
    {:cont, ctx}
  end

  def handle_create_user(ctx) do
    # Business logic
    {:halt, {:ok, %{message: "User created"}}}
  end
end
```

### Action with Custom Validation
```elixir
action :request_otp do
  step :cast_validate_params,
    schema: %{
      phone!: :string,
      region: [field: :string, default: "US"],
      challenge_token!: :string
    },
    validate: &validate_phone_and_token/1
  step :require_authenticated_user
  step :handle_request

  defp validate_phone_and_token(changeset) do
    params = Ecto.Changeset.apply_changes(changeset)

    changeset
    |> cast_validate_phone_number(:phone, region: params.region)
    |> validate_challenge_token_not_expired(:challenge_token)
  end

  def handle_request(ctx) do
    case Accounts.generate_otp(ctx.params.phone) do
      {:ok, otp} ->
        SmsService.send_otp(ctx.params.phone, otp.code)
        {:halt, {:ok, %{message: "OTP sent"}}}
      {:error, reason} ->
        {:halt, {:error, reason}}
    end
  end
end
```

### Action with External Steps
```elixir
action :complex_operation do
  step :cast_validate_params, schema: %{data!: :string}
  step :require_admin
  step {MySteps, :enrich_context}, fields: [:preferences, :billing]
  step {MySteps, :validate_external_service}
  step :handle_operation
end
```

## API Design

### Entry Point
```elixir
# Primary function - returns result tuple
MyActionModule.run(action, params, source) ->
  {:ok, result} | {:error, reason}

# The source parameter provides context and can be:
# - A plain map (treated as assigns directly)
# - Phoenix.LiveView.Socket (assigns extracted from socket.assigns)
# - Plug.Conn (assigns extracted from conn.assigns)
# - Any struct/map with assigns field (assigns are automatically extracted)

# This unified interface allows the same action logic to work across
# Phoenix Controllers and LiveViews where Plugs cannot be used

# The run/3 function processes the result from the internal action pipeline:
def run(action, params, source) do
  ctx = run_action_pipeline(action, params, source)

  case ctx.result do
    {:ok, value} -> {:ok, value}
    {:error, reason} -> {:error, reason}
    other -> {:ok, other}  # Plain values become {:ok, value}
  end
end
```

### Integration Points
```elixir
# Phoenix Controller - pass full conn for richer context
def create(conn, params) do
  case MyApp.UserActions.run(:create_user, params, conn) do
    {:ok, user} -> json(conn, %{success: true, user: user})
    {:error, %{reason: :invalid_params, changeset: changeset}} ->
      json(conn, %{errors: format_changeset_errors(changeset)})
    {:error, reason} ->
      json(conn, %{error: reason})
  end
end

# LiveView - pass full socket for richer context
def handle_event("submit", params, socket) do
  case MyApp.FormActions.run(:submit_form, params, socket) do
    {:ok, result} -> {:noreply, put_flash(socket, :info, "Success!")}
    {:error, reason} -> {:noreply, put_flash(socket, :error, "Error: #{reason}")}
  end
end

# Access original source in action steps
def my_custom_step(ctx) do
  source = get_private(ctx, :source)
  
  # Different behavior based on source type
  case source do
    %Plug.Conn{} -> 
      # Handle API request
      {:cont, assign(ctx, :request_type, :api)}
    %Phoenix.LiveView.Socket{} ->
      # Handle LiveView event
      {:cont, assign(ctx, :request_type, :liveview)}
    _assigns_map ->
      # Handle direct calls
      {:cont, assign(ctx, :request_type, :direct)}
  end
end
```

## Success Criteria

A successful implementation should:

1. **Compile successfully** with the example code provided
2. **Execute steps in order** with proper error handling
3. **Emit telemetry events** with configurable metadata
4. **Handle rate limiting** with flexible key generation
5. **Integrate with existing parameter validation** systems
6. **Provide clear error messages** for debugging
7. **Support testing** of individual steps and full actions
8. **Scale to dozens of actions** per module without performance issues
9. **Feel natural** to experienced Elixir developers
10. **Maintain backwards compatibility** as new features are added

This specification provides a complete blueprint for implementing a production-ready Action DSL that balances power with simplicity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guess)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/guess)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
