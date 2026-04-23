## picotorokko

> PicoRuby application development CLI for ESP32.

# picotorokko

PicoRuby application development CLI for ESP32.

**Tech Stack**: Ruby 3.4+, Rake, Thor CLI, RuboCop, test-unit, SimpleCov

@import AGENTS.md

## Scenario Test Debugging Workflow

### Overview

使用デバッグジェムでシナリオテストをステップ実行検証ぴょん。

### Quick Start

```bash
# 1. Install debug gem locally
gem install debug

# 2. Run test with step execution using ruby -r debug
ruby -r debug -Itest test/scenario/new_scenario_test.rb

# 3. At the debugger prompt, use:
#    step    - Step to next line
#    next    - Step over method calls
#    pp var  - Print variable
#    help    - Show all commands
```

### Detailed Documentation

Complete step execution guide:
```bash
see: .claude/docs/step-execution-guide.md
```

Key sections:
- **Prerequisites**: Debug gem installation
- **Basic Workflow**: Identify test → Set breakpoint → Inspect state
- **Practical Examples**: new_scenario_test, multi_env_test, phase5_e2e_test
- **Advanced Techniques**: Multiple breakpoints, conditional breaks, post-mortem debugging
- **Troubleshooting**: Common issues and solutions

### Integration with TDD Cycle

```
1. Red Phase
   └─ bundle exec ruby -Itest test/scenario/your_test.rb
      (See test failure)

2. Green Phase
   └─ ruby -r debug -Itest test/scenario/your_test.rb
      (Step through execution, understand expected behavior)
      └─ Modify implementation code based on findings

3. RuboCop Phase
   └─ bundle exec rubocop test/scenario/your_test.rb --autocorrect

4. Refactor Phase
   └─ ruby -r debug -Itest test/scenario/your_test.rb
      (Verify refactoring didn't break behavior)

5. Commit Phase
   └─ git add . && git commit -m "..."
```

### Example: Debug new_scenario_test

```bash
# Run test with step execution
ruby -r debug -Itest test/scenario/new_scenario_test.rb

# At (rdbg) prompt:
(rdbg) step                     # Move to next line
(rdbg) step                     # Enter test method
(rdbg) step                     # Create tmpdir
(rdbg) step                     # Generate project_id
(rdbg) step                     # Execute ptrk command
(rdbg) pp output               # See command output
(rdbg) pp status.success?       # Check exit status
(rdbg) continue                # Continue to next assertion
(rdbg) quit                    # Exit debugger
```

### Test Helper Reference

```ruby
# Execute ptrk CLI in specified directory
output, status = run_ptrk_command("new my-project", cwd: tmpdir)
# Returns: [String output, Process::Status status]

# Generate unique project ID for test isolation
project_id = generate_project_id
# Returns: "20251203_015030_abc123f" (timestamp + hash)

# Capture stdout/stderr from block
captured = capture_stdout { ... }
# Returns: String of captured output
```

### Useful Debug Commands

| Command | Purpose |
|---------|---------|
| `step` | Execute next line (step into methods) |
| `next` | Execute next line (step over methods) |
| `continue` | Continue execution to next breakpoint |
| `info locals` | Show all local variables |
| `pp var` | Pretty-print variable value |
| `pp var.class` | Show variable class |
| `pp var.inspect` | Inspect variable details |
| `list` | Show current code context |
| `help` | Show all debugger commands |
| `quit` | Exit debugger |

### Common Debugging Patterns

**Pattern 1: Check command success**
```ruby
output, status = run_ptrk_command("new #{project_id}", cwd: tmpdir)
# Debug: Is command succeeding?
(rdbg) pp status.success?
(rdbg) pp status.exitstatus
(rdbg) pp output  # See error message
```

**Pattern 2: Verify file system state**
```ruby
assert Dir.exist?(File.join(project_dir, "storage", "home"))
# Debug: What files actually exist?
(rdbg) system("find #{tmpdir} -type f")
(rdbg) system("ls -la #{project_dir}")
```

**Pattern 3: Check assertion failures**
```ruby
assert_match(/expected_pattern/, output)
# Debug: What's in output?
(rdbg) pp output.lines       # See as array of lines
(rdbg) pp output.include?("expected_pattern")  # Check if present
```

**Pattern 4: Multi-step workflows**
```ruby
# Multiple ptrk commands in sequence
run_ptrk_command("new #{project_id}", cwd: tmpdir)
run_ptrk_command("env set --latest", cwd: project_dir)
run_ptrk_command("device prepare", cwd: project_dir)

# Debug: Check state after each command
(rdbg) step  # Execute first command
(rdbg) system("ls -la #{tmpdir}")  # Check what was created
(rdbg) continue  # Jump to next command
(rdbg) pp output  # Check second command output
```

### Notes

1. **Debug gem location**: Installed locally (not in Gemfile) to avoid CI conflicts
2. **Step execution approach**: Use `ruby -r debug` instead of `rdbg` command (PATH compatibility)
3. **Test isolation**: Each test uses `Dir.mktmpdir` with `generate_project_id()` for safety
4. **Exit codes**: Always check `status.success?` or `status.exitstatus` after commands
5. **File permissions**: Remember tests run in isolated tmpdir, not actual project

See also:
- `SPEC.md` — Feature specification
- `TODO.md` — Project status and remaining work
- `test_helper.rb` — Test utility functions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bash0C7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
