## new-tool-detection

> >-


# New Component Detection — Auto-Generate Tests

When ANY new plugin component is added to the project, IMMEDIATELY write evaluation
tests for it. Do NOT just suggest — write the YAML directly.

## What to Do

### When a new MCP tool is created (`registerTool()` / `server.tool()`):

1. **Read the tool** to understand its purpose, parameters, and required env vars
2. **Check if tests exist** in `plugin-eval.yaml` for this tool
3. **If no tests exist, WRITE THEM** — all 4 test types:

**Unit test** — add to the registration test's `expected_tools` list:
```yaml
- name: new-tool-registers
  check: registration
  expected_tools: [new_tool_name]
```

**Integration test** — happy path with assertions:
```yaml
- name: new-tool-happy-path
  tool: new_tool_name
  difficulty: simple
  args: {realistic args based on schema}
  assert:
    - field: content.0.text
      op: exists
```

**LLM test** — natural language prompt:
```yaml
- name: new-tool-selection
  difficulty: simple
  prompt: "Natural language that should trigger this tool"
  expected:
    tools: [new_tool_name]
  evaluators:
    - tool-selection
    - correctness
    - mcp-protocol
    - security
```

**Negative test** — error handling:
```yaml
- name: new-tool-invalid-args
  tool: new_tool_name
  difficulty: moderate
  args: {intentionally invalid args}
  expect_error: true
```

### When a new Skill is created (`SKILL.md`):

1. **Read the skill** — understand name, description, triggers, and what tools/commands it references
2. **Write ALL of these tests:**

**Static frontmatter test:**
```yaml
- name: skill-{name}-frontmatter
  check: skill_frontmatter
  components: [{skill-directory-name}]
```

**Skill activation test (positive — LLM layer):**
```yaml
- name: skill-{name}-activates
  difficulty: simple
  prompt: "{phrase that matches one of the skill's declared triggers}"
  evaluators: [skill-trigger, content-quality, correctness]
```

**Skill activation test (negative — LLM layer):**
```yaml
- name: skill-{name}-no-false-activate
  difficulty: moderate
  prompt: "{related but off-topic phrase that should NOT activate this skill}"
  evaluators: [skill-trigger, correctness]
```

**Cross-reference test:**
```yaml
- name: skill-{name}-cross-refs
  check: cross_component_coherence
  components: [{skill-directory-name}]
```

### When a new Rule is created (`.mdc`):

1. **Read the rule** — understand description, alwaysApply/globs, and what it enforces
2. **Write these tests:**

**Static frontmatter test:**
```yaml
- name: rule-{name}-frontmatter
  check: rule_frontmatter
  components: [{rule-filename}]
```

**Content quality test:**
```yaml
- name: rule-{name}-content
  check: rule_content_quality
  components: [{rule-filename}]
```

### When a new Agent is created (`.md` in agents/):

1. **Read the agent** — understand name, description, model, and what tools it should use
2. **Write these tests:**

**Static frontmatter test:**
```yaml
- name: agent-{name}-frontmatter
  check: agent_frontmatter
  components: [{agent-filename}]
```

**Behavior test (LLM layer):**
```yaml
- name: agent-{name}-behavior
  difficulty: moderate
  prompt: "{task that the agent is designed for}"
  evaluators: [tool-selection, correctness, content-quality]
```

### When a new Command is created (`.md` in commands/):

1. **Read the command** — understand name, description, argument-hint, and allowed-tools
2. **Write these tests:**

**Static frontmatter test:**
```yaml
- name: command-{name}-frontmatter
  check: command_frontmatter
  components: [{command-filename}]
```

**Execution test (LLM layer):**
```yaml
- name: command-{name}-executes
  difficulty: simple
  prompt: "{invocation of the command with expected arguments}"
  evaluators: [task-completion, correctness]
```

## When to Stay Silent

- If the component already has eval tests covering all categories above
- If the change is a minor edit to an existing component (description update, typo fix)
- If only the body content changed but triggers/schema/frontmatter stayed the same

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
